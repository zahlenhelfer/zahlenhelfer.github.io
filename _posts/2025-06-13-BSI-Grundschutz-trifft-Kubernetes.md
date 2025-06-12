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

Das IT-Grundschutzkompendium befasst sich auch mit Kubernetes. Die Damen und Herren des BSI haben dem Produkt sogar einen eigenen Baustein gewidmet. Der Baustein [APP.4.4 Kubernetes](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Grundschutz/IT-GS-Kompendium_Einzel_PDFs_2022/06_APP_Anwendungen/APP_4_4_Kubernetes_Edition_2022.pdf?__blob=publicationFile&v=3). Ich möchte in dieser Serie die Bausteine beleuchten und mögliche Implementierungen aufzeigen.

TODO: Callout machen
DISCLAIMER: Ich bin weder Auditor noch Anwalt

Bevor ich jetzt die x´te Einführung in den Grundschutz mache gehen wir die Bausteine doch einfach durch. Nur ein minimales Verständnis vorab:
- Wenn deine Firma es noch nicht weiss, stelle fest ob Du zur kritischen Infrasturktur gehörst (KRITIS)
- Nicht alle Verfahren oder Prozesse müssen zwangsläufig KRITIS sein

Zielsetzung:
Ziel dieses Bausteins ist der Schutz von Informationen, die in Kubernetes-Clustern verarbeitet, angeboten oder darüber übertragen werden.
