---
layout: post
title: Empfehlungen für Ressource-Limits/Ressource Requests per Vertical Pod Autoscaler
category: kubernetes
tags:
  - blog
  - de
  - kubernetes
  - vpa
  - tip
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---
## Das Problem: Gute Werte für `Resource Requests`
Wer kennt es nicht: Man setzt CPU- und Memory-Requests in Kubernetes – und trifft dabei fast nie den richtigen Wert. In meinen Trainings ist es eine der Top 10 Fragen. Wie mache ich es richtig bzw. wo kommen diese Werte her? Zu niedrig konfiguriert, werden Pods gedrosselt oder mit einem OOMKill beendet. Zu großzügig konfiguriert, verschwendest Du teure Ressourcen. Nicht jeder kann schon am Anfang mit Erfahrungswerten oder einer guten Instrumentierung wie OpenTelemetry arbeiten.

Genau hier kommt der **Vertical Pod Autoscaler (VPA)** ins Spiel. Er analysiert den tatsächlichen Ressourcenverbrauch deiner Pods und passt die Requests automatisch an.
## Die Lösung: `enableServiceLinks: false`

Der VPA besteht dabei aus drei Komponenten:
- **Recommender**: Analysiert den Ressourcenverbrauch und erstellt Empfehlungen
- **Updater**: Beendet Pods, die neue Ressourcen benötigen (durch Eviction)
- **Admission Controller**: Setzt beim Start neuer Pods direkt die empfohlenen Werte

Natürlich braucht es dazu einige Komponenten in deinem Cluster:
- Metrics Server
- VPA Recommender
- 
## Beispiel:  Werte für einen Webserver



## Fazit:
