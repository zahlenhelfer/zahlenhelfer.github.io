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
published: false
render_with_liquid: "false"
---

## Das Problem: --kubelet-insecure-tls und der metrics-server
Bei vielen meiner Trainings für das Thema Kubernetes kommt der Punkt wo wir den [metrics-server](https://github.com/kubernetes-sigs/metrics-server) installieren. Sei es um den Horizontal-Pod-Autoscaler zu zeigen oder einfach damit `kubectl top pod` funktioniert. Dabei gibt es jedesmal die Meldung:
```
E0108 13:29:15.336920 1 scraper.go:149] "Failed to scrape node" err="Get \"https://167.71.63.166:10250/metrics/resource\": tls: failed to verify certificate: x509: cannot validate certificate for 167.71.63.166 because it doesn't contain any IP SANs" node="k8s-node-0"
```
Was ist passiert? Kurz zusammengefasst - der Subject Alternative Name oder auch kurz SAN ist nicht korrekt gesetzt. Der `metrics-server` möchte gerne die IP-Adresse und nicht den DNS Namen des Zertifikates validieren. Du kannst es relativ einfach prüfen:
`openssl x509 -text -noout -in /var/lib/kubelet/pki/kubelet.crt`
Dort fehlt gerade bei selbst erstellten Kubernetes-Nodes häufig die IP Adresse im SAN-Teil. 
```
X509v3 Subject Alternative Name:
    DNS:k8s-node-0
```
Auch ein Umstellen der Reihefolge der Argumente beim Adress-Type
`--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname`
bringt nicht immer eine Lösung. Hier kann zwar der `Hostname` an erste Stelle gezigen werden, da aber gerade in Homelabs der Hostname vom DNS gerne mal nicht auflösen möchte bringt das auch keinen Erfolgt. Also lesen wir mal bei GitHub nach, was das Projekt dazu meint.
## Keine Lösung: `--kubelet-insecure-tls`
Genau dafür gibt die offizielle README.md des `metric-server` auf [GitHub](https://github.com/kubernetes-sigs/metrics-server?tab=readme-ov-file#requirements) einen Ratschlag:
>- Kubelet certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing `--kubelet-insecure-tls` to Metrics Server)

Also schnell mal `kubectl edit deployment metrics-server -n kube-system` gestartet und `--kubelet-insecure-tls` als Argument hinzugefügt.
Aber das kann doch nicht die "richtige" Lösung sein. Es ist doch nur ein Feld in den Zertifikaten. Es ist eben Quick & Dirty und wird gerne genutzt.

## Die schnelle & saubere Lösung: `tlsBootstrap`
Da es sich bei dem Thema um das die Zertifikate für das `Kubelet`handelt sollten wir "einfach" ein neues Zertifikat bei der Kubernetes-CA ausstellen und signieren lassen. Das ganze kann natürlich mit `openssl`-Magie passieren. Aber es geht auch noch einfacher. Es reichen diese drei einfache Schritte: 
- Die kubelete-Config mit [`serverTLSBootstrap`](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/) erweitern
- Den Dienst `kubelet` neu starten und damit einen `csr` in Kubernetes auslösen (siehe Kubernetes-Dokumentation)
- Den `certificate signing request` in Kubernetes genehmigen

## Beispiel:  signierte Zertifikate von der Kubernetes-CA
Die config-Datei des kubelet erweitern `/var/lib/kubelet/config.yaml`

```shell
serverTLSBootstrap: true
```

Jetzt noch den Dienst neu starten
```shell
systemctl restart kubelet
```

Gib die Eingaben an (falls wie im Beispiel definiert) und Bestätige mit **Run**. Das war es auch schon. Wenn der Workflow dann durchgelaufen ist, kann Du wie gewohnt die Logs inspizieren.
```shell
kubectl get csr
NAME        AGE   SIGNERNAME                      REQUESTOR              REQUESTEDDURATION   CONDITION
csr-dnrzp   2s    kubernetes.io/kubelet-serving   system:node:k8s-node-0   <none>              Pending
```

```shell
kubectl certificate approve csr-dnrzp
```
Das muss natürlich für alle Nodes durchgeführt werden. Wenn Ihr dann bei den metrics-server-pods schaut sollte der READY-Status von 0/1 auf 1/1 gewechselt haben.
## Fazit
Manchmal ist es einfach kein Aufwand die Dinge richtig zu machen. Ja, Quick & Dirty ist schön aber der Aufwand ist doch überschaubar, also lasst es uns richtig machen. PS: schaut mal in der kubeconfig-Ressource
