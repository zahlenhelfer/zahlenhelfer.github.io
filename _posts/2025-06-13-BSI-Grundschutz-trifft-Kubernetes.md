---
layout: post
title: BSI Grundschutz trifft auf Kubernetes-Cluster
category: kubernetes
tags:
  - blog
  - de
  - bsi
  - grundschutz
  - kubernetes
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---

## BSI Grundschutz und Kubernetes - passt das zusammen?

Das IT-Grundschutzkompendium befasst sich auch mit Kubernetes. Die Damen und Herren des BSI haben dem Produkt sogar einen eigenen Baustein gewidmet - [APP.4.4 Kubernetes](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Grundschutz/IT-GS-Kompendium_Einzel_PDFs_2022/06_APP_Anwendungen/APP_4_4_Kubernetes_Edition_2022.pdf?__blob=publicationFile&v=3). Ich möchte in dieser Serie die Bausteine beleuchten und mögliche Implementierungen aufzeigen.

>DISCLAIMER: Ich bin weder Auditor noch Anwalt - Keine Gewähr!

Bevor ich jetzt die 3.242te-Einführung in den IT-Grundschutz mache gehen wir die Bausteine doch einfach durch. Aber ein minimales Verständnis vorab für die Dinge die jetzt passieren werden:
- Das Grundschutz-Kompendium

## Zielsetzung der Serie:
Ziel dieses Bausteins ist der Schutz von Informationen, die in Kubernetes-Clustern verarbeitet, angeboten oder darüber übertragen werden.
