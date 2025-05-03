---
layout: post
title: Behebung des ArgoCD-Fehler "Too long must have at most 262144 bytes"
category: kubernetes
tags:
  - blog
  - k8s
  - tip
  - de
  - argocd
permalink: /:year/:month/:day/:title:output_ext
published: false
---

## Das Problem: Kubernetes-Objekte haben eine feste Größenbeschränkung von 256kb für Annotationen.
Wenn kubectl apply verwendet wird, um Ressourcen zu aktualisieren (was Argo CD tut), wird versucht, eine last-applied-configuration-Annotation zu setzen, die die JSON-Darstellung der zuletzt angewendeten Objektkonfiguration enthält.

Wenn wir zum Beispiel ein Deployment-Objekt haben, das wir mit kubectl apply aktualisieren, wird die last-applied-configuration-Annotation das JSON-serialisierte Format des gesamten Deployment-Objekts enthalten:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment", "metadata":{}, ...}
```

Dies ist für die meisten Ressourcen kein Problem, aber es gibt Objekte, die die 256kb-Grenze überschreiten, wie z.B. die Prometheus CRD aus dem kube-prometheus-stack Helm-Diagramm, die 500kb groß ist.

## Die Lösung: automatische Schema Generierung aus der `values.yaml`
Hier sei das GitHub-Projekt [helm-values-schema-json](https://github.com/losisin/helm-values-schema-json/tree/main) genannt. 


Beim Synchronisieren der Prometheus-CRD in Argo CD wird kubectl apply ausgeführt und versucht, die 500kb große JSON-Darstellung als Annotation hinzuzufügen. Dies führt zu der Fehlermeldung "Too long: must have at most 262144 bytes", da die Größe der Kubernetes-Annotationen auf 256kb (oder 262144 Bytes) begrenzt ist.

Lösung
Die Lösung besteht darin, die Verwendung von Client Side Apply (die aktuelle Standardeinstellung beim Ausführen von kubectl apply) zu beenden und stattdessen Server Side Apply zu verwenden, das die Annotation „last-applied-configuration“ nicht zu Objekten hinzufügt.

Es ist geplant, dass Server Side Apply in zukünftigen Kubernetes- und Argo-CD-Versionen die Standard-Anwendungsmethode sein wird, aber im Moment müssen wir es explizit aktivieren.

Die Unterstützung für Server Side Apply wurde in Argo CD v2.5 hinzugefügt und kann aktiviert werden, indem man sie in den Sync-Optionen für eine Anwendungsressource einstellt:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
spec:
  syncPolicy:
    syncOptions:
    - ServerSideApply=true
```

Argo CD Versionen kleiner als v2.5 haben keine Unterstützung für Server Side Apply. In diesen Fällen können wir auf die Verwendung von kubectl replace zurückgreifen, um das Objekt zu aktualisieren. Wir können dies tun, indem wir die Sync-Option Replace=true setzen. Beachten Sie jedoch, dass replace zu unerwarteten Ergebnissen führen kann, wenn mehrere Clients ein Objekt ändern.