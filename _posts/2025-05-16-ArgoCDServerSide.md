---
layout: post
title: ArgoCD-Fehler "Too long must have at most 262144 bytes" - ein Klassiker
category: kubernetes
tags:
  - blog
  - k8s
  - tip
  - de
  - argocd
  - fehlermeldung
permalink: /:year/:month/:day/:title:output_ext
published: true
---

## Das Problem: Größenbeschränkung (256kb) bei Kubernetes-Objekten (für Annotationen).
Die Fehlermeldung `Too long must have at most 262144 bytes` in **ArgoCD** bedeutet, dass ein Objekt (z. B. eine **ConfigMap**, ein **Secret**, ein **Application Manifest**, etc.) **zu groß** ist. Die maximale erlaubte Größe von **262.144 Bytes** (also **256 KiB**) wurde überschritten. Das wird bei den meisten Ressourcen nie erreicht. Es gibt aber Objekte die sehr schnell die 256kb-Grenze überschreiten. Sehr prominent ist hier der PrometheusCRD. Die meisten Leser werden daher die o.g. Fehlermeldung wohl aus dem `kube-prometheus-stack` kennen. (ca. 500kb).
## Die Lösung: Verwendung von Server Side Apply
Beim Synchronisieren der Prometheus-CRD in ArgoCD wird `kubectl apply` ausgeführt und versucht, die 500kb große JSON-Darstellung als Annotation hinzuzufügen. Dies führt zu der Fehlermeldung `Too long: must have at most 262144 bytes`, da die Größe der Kubernetes-Annotationen auf 256kb (oder 262144 Bytes) begrenzt ist. Ein einfaches `kubectl apply --server-side -f xyz.yaml` würde hier helfen. 

Die Unterstützung für Server Side Apply wurde mit ArgoCD v2.5 hinzugefügt und kann aktiviert werden, indem man sie in den Sync-Optionen der Sync-Policy hintrlegt `ServerSideApply=true` (Application/ApplicationSet). ArgoCD Versionen kleiner als v2.5 haben **keine** Unterstützung für Server Side Apply.

## Bespiel: Deployment - mit / ohne ServerSide
Sehr einfach kann dieses stille Anwachsen auch mit einem einfachen Deployment gezeigt werden.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx
```
Diese Ressource wird nun mit 
- `kubectl apply -f deployment.yaml`
an Kubernetes übergeben. Ein 
- `kubectl get deployment my-app -o yaml`
zeigt nun die erstellte Annotation `last-applied-configuration` an und verdoppelt damit mal eben die Grösse der Datei.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"my-app","namespace":"default"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"my-app"}},"template":{"metadata":{"labels":{"app":"my-app"}},"spec":{"containers":[{"image":"nginx","name":"my-app"}]}}}}
...
```

Zum Vergleich mit dem Argument `--server-side`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2025-05-25T14:19:19Z"
```

Zum selber Testen:
- Erst die Ressource wieder löschen
	`kubectl delete -f deployment.yaml`
- Dann mit dem `--server-side` Argument erstellen
	`kubectl apply --server-side -f deployment.yaml` 
- Prüfen des Deployment 
  `kubectl get deployment my-app -o yaml`

Jetzt wird die Annotation vermieden. In ArgoCD kann dies einfach in den `syncOptions` beeinflusst werden. Hier einmal stark verkürzt darsgestellt.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: test-deployment
spec:
...
  syncPolicy:
    syncOptions:
    - ServerSideApply=true
```

## Fazit:
- Kubernetes Ressourcen haben Limits (256kb).
- Die Annotation `last-applied-configuration` beschleunigt die Vergrösserung von Ressourcen
- bei `kubectl apply` das Argument `--server-side` nutzen
- in ArgoCD `syncOptions` nutzen

