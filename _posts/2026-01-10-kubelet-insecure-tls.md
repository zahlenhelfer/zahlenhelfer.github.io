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

## Das Problem: Zertifikate und der metrics-server
Bei vielen meiner Trainings für das Thema Kubernetes kommt der Punkt, wo wir den [metrics-server](https://github.com/kubernetes-sigs/metrics-server) installieren. Sei es um den Horizontal-Pod-Autoscaler zu zeigen oder einfach damit `kubectl top pod` funktioniert. Dabei gibt es jedesmal die Meldung im Log des metric-server-pods:
```
E0108 13:29:15.336920 1 scraper.go:149] "Failed to scrape node" err="Get \"https://167.71.63.166:10250/metrics/resource\": tls: failed to verify certificate: x509: cannot validate certificate for 167.71.63.166 because it doesn't contain any IP SANs" node="k8s-node-0"
```
Was ist passiert? Kurz zusammengefasst - der Subject Alternative Name oder auch kurz SAN ist nicht korrekt gesetzt. Der `metrics-server` möchte gerne die IP-Adresse und nicht den DNS Namen des Zertifikates validieren. Du kannst es relativ einfach per Terminal prüfen:
`openssl x509 -text -noout -in /var/lib/kubelet/pki/kubelet.crt`
Dort fehlt gerade bei selbst erstellten Kubernetes-Nodes häufig die IP Adresse im SAN-Teil. 
```
X509v3 Subject Alternative Name:
    DNS:k8s-node-0
```
## Keine Lösung: `--kubelet-preferred-address-types`
Die erste Lösung könnte das Umstellen der Reihefolge des Arguments beim Adress-Type sein. Hier ist der Standard:
`--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname`
Hier kann zwar der `Hostname` an erste Stelle gezogen werden, da aber gerade in Homelabs der Hostname vom DNS gerne mal nicht auflösen möchte - bringt das auch keinen Erfolg. Also lesen wir mal bei GitHub nach, was das Projekt dazu meint.
## Keine Lösung: `--kubelet-insecure-tls`
Genau dafür gibt die offizielle `README.md` des `metric-server` auf [GitHub](https://github.com/kubernetes-sigs/metrics-server?tab=readme-ov-file#requirements) einen Ratschlag:
>- Kubelet certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing `--kubelet-insecure-tls` to Metrics Server)

Also schnell mal `kubectl edit deployment metrics-server -n kube-system` gestartet und `--kubelet-insecure-tls` als Argument hinzugefügt.
Aber das kann doch nicht die _richtige_ Lösung sein. Es ist doch nur ein Feld in den Zertifikaten bzw. ein Approve von der CA notwendig. Es ist aber eben Quick & Dirty und wird gerne genutzt.

## Die schnelle & saubere Lösung: `tlsBootstrap`
Da es sich bei dem Thema um die Zertifikate für das `Kubelet`handelt, sollten wir _einfach_ ein neues Zertifikat bei der Kubernetes-CA ausstellen und signieren lassen. Das ganze kann natürlich mit `openssl`-Magie passieren. Aber es geht auch noch einfacher. Es reichen diese drei Schritte: 
- Die kubelete-Config mit [`serverTLSBootstrap`](https://kubernetes.io/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/) erweitern
- Den Dienst `kubelet` neu starten und damit einen `csr` in Kubernetes auslösen (siehe Kubernetes-Dokumentation)
- Den `certificate signing request` in Kubernetes genehmigen

## Beispiel:  signierte Zertifikate von der Kubernetes-CA
Die config-Datei des kubelet um die folgende Zeile erweitern 

```yaml
# File: `/var/lib/kubelet/config.yaml`
serverTLSBootstrap: true
```

Jetzt noch den Dienst neu starten
```shell
systemctl restart kubelet
```

Wenn wir jetzt per `kubectl get csr` den Cluster fragen ob certificate signing request vorhanden sind, könnte die Antwort so aussehen:
```shell
NAME        AGE   SIGNERNAME                      REQUESTOR              REQUESTEDDURATION   CONDITION
csr-dnrzp   2s    kubernetes.io/kubelet-serving   system:node:k8s-node-0   <none>              Pending
```

```shell
kubectl certificate approve csr-dnrzp
```
Bitte bedenke das natürlich auf allen Nodes durchzuführen. Wenn Ihr dann bei den metrics-server-pods schaut sollte der `READY`-Status von 0/1 auf 1/1 gewechselt haben und die Fehlermeldung im Log auch verschwunden sein.
## Fazit
Manchmal ist es einfach kein Aufwand die Dinge richtig zu machen. Ja, Quick & Dirty ist schön aber lasst es uns richtig machen. 

PS: Wenn Du das schon beim Bootstrappen der Nodes beachten möchtest, schau mal in der [Kubelet Configuration (v1beta1)](https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/#kubelet-config-k8s-io-v1beta1-KubeletConfiguration)-Ressource von Kubernetes nach.
