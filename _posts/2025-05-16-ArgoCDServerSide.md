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
published: false
---

## Das Problem: Größenbeschränkung (256kb) bei Kubernetes-Objekten (für Annotationen).
"Ist doch klar" - mögen die Erfahrenen sagen. Aber auch Klassiker haben einen Platz wiederholt zu werden.

Für die Fehlermeldung  `Too long must have at most 262144 bytes` im ArgoCD, ist das Problem **nicht** ArgoCD sondern Kubernetes.

Bei einem verwenden von z.B. `kubectl apply` um ein Deployment zu aktualisieren,  wird  eine Annotation `last-applied-configuration-Annotation` erstellt bzw. aktualisiert. Es enthält die JSON-Darstellung der zuletzt angewendeten Deploymentkonfiguration.

```yaml

```

Das ist bei den meisten meisten Ressourcen kein Problem. Es gibt aber Objekte die sehr schnell die 256kb-Grenze überschreiten. Sehr prominent ist hier der Prometheus CRD. Die meisten werden daher die o.g. Fehlermeldung wohl aus dem kube-prometheus-stack kennen. (500kb)

## Die Lösung: Verwendung von Server Side Apply
Beim Synchronisieren der Prometheus-CRD in Argo CD wird kubectl apply ausgeführt und versucht, die 500kb große JSON-Darstellung als Annotation hinzuzufügen. Dies führt zu der Fehlermeldung "Too long: must have at most 262144 bytes", da die Größe der Kubernetes-Annotationen auf 256kb (oder 262144 Bytes) begrenzt ist.

Lösung
Die Lösung besteht darin, die Verwendung von Client Side Apply (die aktuelle Standardeinstellung beim Ausführen von kubectl apply) zu beenden und stattdessen Server Side Apply zu verwenden, das die Annotation „last-applied-configuration“ nicht zu Objekten hinzufügt.

Es ist geplant, dass Server Side Apply in zukünftigen Kubernetes- und Argo-CD-Versionen die Standard-Anwendungsmethode sein wird, aber im Moment müssen wir es explizit aktivieren.

Die Unterstützung für Server Side Apply wurde mit Argo CD v2.5 hinzugefügt und kann aktiviert werden, indem man sie in den Sync-Optionen für eine Anwendungsressource einstellt:

```yaml
spec:
  syncPolicy:
    syncOptions:
    - ServerSideApply=true
```

Argo CD Versionen kleiner als v2.5 haben **keine** Unterstützung für Server Side Apply. In diesen Fällen können wir auf die Verwendung von kubectl replace zurückgreifen, um das Objekt zu aktualisieren. Wir können dies tun, indem wir die Sync-Option Replace=true setzen. Beachten Sie jedoch, dass replace zu unerwarteten Ergebnissen führen kann, wenn mehrere Clients ein Objekt ändern.

## Fazit:
