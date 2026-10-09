---
layout: post
title: "Tip: --kubelet-insecure-tls NICHT mehr nutzen"
category: kubernetes
tags:
  - blog
  - tip
  - de
  - kubernetes
  - tls
permalink: /:year/:month/:day/:title:output_ext
published: true
render_with_liquid: "false"
---

> Update Oktober 2026: Gegen Kubernetes v1.37 und metrics-server 0.9.x geprüft - das Vorgehen funktioniert unverändert. Ergänzt habe ich die `kubelet-config`-ConfigMap, die Rotation nach einem Jahr und die Prüfung des neuen Zertifikats.

## Das Problem: Zertifikate und der metrics-server

Bei vielen meiner Trainings für das Thema Kubernetes kommt der Punkt, wo wir den [metrics-server](https://github.com/kubernetes-sigs/metrics-server) installieren. Sei es, um den Horizontal-Pod-Autoscaler zu zeigen, oder einfach, damit `kubectl top pod` funktioniert. Dabei gibt es jedes Mal diese Meldung im Log der metrics-server-Pods:

```text
E0108 13:29:15.336920 1 scraper.go:149] "Failed to scrape node" err="Get \"https://167.71.63.166:10250/metrics/resource\": tls: failed to verify certificate: x509: cannot validate certificate for 167.71.63.166 because it doesn't contain any IP SANs" node="k8s-node-0"
```

Was ist passiert? Der `metrics-server` spricht den Node über seine IP-Adresse an, im Subject Alternative Name (kurz SAN) des kubelet-Zertifikats steht aber nur der Hostname. Du kannst es relativ einfach im Terminal prüfen:

```bash
openssl x509 -text -noout -in /var/lib/kubelet/pki/kubelet.crt
```

```text
X509v3 Subject Alternative Name:
    DNS:k8s-node-0
```

Das ist aber nur die halbe Wahrheit. Bei mit `kubeadm` gebauten Clustern stellt sich das kubelet dieses Zertifikat beim ersten Start **selbst** aus - nur mit dem Hostnamen und ohne Unterschrift der Cluster-CA. Die fehlende IP ist also das Symptom, das selbstsignierte Zertifikat die Ursache.

## Keine Lösung: `--kubelet-preferred-address-types`

Die erste Idee könnte sein, die Reihenfolge der Adresstypen umzustellen. Das offizielle Manifest und das Helm-Chart setzen:

```text
--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
```

Hier kann zwar der `Hostname` an die erste Stelle gezogen werden. Gerade in Homelabs löst der Hostname im DNS aber gerne mal nicht auf. Und selbst wenn: Dann meckert der `metrics-server` als Nächstes über `x509: certificate signed by unknown authority`, denn das Zertifikat bleibt selbstsigniert. Also lesen wir mal bei GitHub nach, was das Projekt dazu meint.

## Keine Lösung: `--kubelet-insecure-tls`

Genau dafür gibt die offizielle `README.md` des `metrics-server` auf [GitHub](https://github.com/kubernetes-sigs/metrics-server?tab=readme-ov-file#requirements) einen Ratschlag:

> Kubelet certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing `--kubelet-insecure-tls` to Metrics Server)

Also schnell mal `kubectl edit deployment metrics-server -n kube-system` gestartet und `--kubelet-insecure-tls` als Argument hinzugefügt.

Aber das kann doch nicht die _richtige_ Lösung sein. Dieselbe README schreibt zu dem Flag: "For testing purposes only." Es ist eben Quick & Dirty und wird trotzdem gerne genutzt.

## Die schnelle & saubere Lösung: `serverTLSBootstrap`

Da es um das Serving-Zertifikat des `kubelet` geht, sollten wir _einfach_ ein neues Zertifikat von der Kubernetes-CA signieren lassen. Das Ganze kann natürlich mit `openssl`-Magie passieren. Aber es geht auch einfacher. Es reichen diese drei Schritte:

- Die kubelet-Config um [`serverTLSBootstrap`](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/#kubelet-serving-certs) erweitern
- Den Dienst `kubelet` neu starten und damit einen `csr` in Kubernetes auslösen
- Den Certificate Signing Request in Kubernetes genehmigen

## Ein Beispiel: signierte Zertifikate von der Kubernetes-CA

Zuerst die clusterweite Vorlage anpassen. `kubeadm` hält die kubelet-Config in einer ConfigMap vor und überschreibt damit bei jedem `kubeadm upgrade` die lokale Datei. Wer nur die Datei auf dem Node ändert, ist die Einstellung nach dem nächsten Upgrade wieder los.

```bash
kubectl edit cm -n kube-system kubelet-config
```

Dort im Key `kubelet` dieselbe Zeile ergänzen wie danach auf jedem Node in der Datei `/var/lib/kubelet/config.yaml`:

```yaml
serverTLSBootstrap: true
```

Jetzt noch den Dienst neu starten:

```bash
systemctl restart kubelet
```

Wenn wir jetzt per `kubectl get csr` den Cluster fragen, ob Certificate Signing Requests vorhanden sind, könnte die Antwort so aussehen:

```text
NAME        AGE   SIGNERNAME                      REQUESTOR                REQUESTEDDURATION   CONDITION
csr-dnrzp   2s    kubernetes.io/kubelet-serving   system:node:k8s-node-0   <none>              Pending
```

Bevor Du genehmigst, lohnt ein Blick in den Request - stehen dort wirklich nur Hostname und IP dieses Nodes?

```bash
kubectl get csr csr-dnrzp -o jsonpath='{.spec.request}' | base64 -d | openssl req -noout -text | grep -A1 "Subject Alternative Name"
```

```bash
kubectl certificate approve csr-dnrzp
```

Lass Dir damit nicht zu viel Zeit: Bis zur Genehmigung hat das kubelet kein Serving-Zertifikat, `kubectl logs` und `kubectl exec` laufen für diesen Node so lange ins Leere.

Bitte denke daran, das auf allen Nodes durchzuführen - auch auf den Control-Plane-Nodes. Das neue Zertifikat landet übrigens nicht in `kubelet.crt`, die alte Datei bleibt einfach liegen. Prüfen kannst Du es so:

```bash
openssl x509 -noout -issuer -ext subjectAltName -in /var/lib/kubelet/pki/kubelet-server-current.pem
```

Als Issuer steht dort jetzt die Cluster-CA, im SAN zusätzlich die IP-Adresse. Bei den metrics-server-Pods sollte der `READY`-Status von 0/1 auf 1/1 gewechselt haben und die Fehlermeldung im Log verschwunden sein.

## Hinweis: nach einem Jahr geht es wieder los

Das Serving-Zertifikat läuft standardmäßig nach einem Jahr ab. Das kubelet stellt rechtzeitig einen neuen CSR - genehmigen muss ihn aber wieder jemand. Der `kube-controller-manager` tut das für den Signer `kubernetes.io/kubelet-serving` aus Sicherheitsgründen bewusst nicht. Bleibt der CSR liegen, steht ein Jahr später derselbe Fehler im Log, nur mit `certificate has expired`.

Im Training ist das egal, im Dauerbetrieb nicht. Dort übernimmt das ein Controller wie der [kubelet-csr-approver](https://github.com/postfinance/kubelet-csr-approver), den auch die Kubernetes-Doku nennt. Wichtig ist, dass er Hostnamen und IP-Bereiche wirklich prüft (`providerRegex`, `providerIpPrefixes`) und nicht einfach alles durchwinkt.

## Fazit

Manchmal ist es einfach kein Aufwand, die Dinge richtig zu machen. Ja, Quick & Dirty ist schön, aber lasst es uns richtig machen.

PS: Wenn Du das schon beim Bootstrappen des Clusters beachten möchtest, gib `kubeadm init` per `--config` diese Datei mit:

```yaml
apiVersion: kubeadm.k8s.io/v1beta4
kind: ClusterConfiguration
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
serverTLSBootstrap: true
```

Details stehen in der Kubernetes-Doku unter [Enabling signed kubelet serving certificates](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/#kubelet-serving-certs).
