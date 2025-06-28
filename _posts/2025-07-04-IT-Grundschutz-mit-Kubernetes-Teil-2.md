---
layout: post
title: BSI Grundschutz Bausteine in APP4.4 für Kubernetes - Teil 2
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

## BSI Grundschutz und Kubernetes - passt das zusammen? - Erfahrungen aus der Praxis

>DISCLAIMER: Ich bin weder Auditor noch Anwalt - daher keine Gewähr!

### APP.4.4 Kubernetes - die Basis-Anforderungen
Wie der Name es schon sagt, das ist der Grundstein, das __MÜSSEN__ wir mit unserem Kubernetes m

Die folgenden Anforderungen MÜSSEN für diesen Baustein vorrangig erfüllt werden.

APP.4.4.A1 Planung der Separierung der Anwendungen (B)

Vor der Inbetriebnahme MUSS geplant werden, wie die in den Pods betriebenen Anwendungen und

deren unterschiedlichen Test- und Produktions-Betriebsumgebungen separiert werden. Auf Basis des

Schutzbedarfs der Anwendungen MUSS festgelegt werden, welche Architektur der Namespaces, Meta-

Tags, Cluster und Netze angemessen auf die Risiken eingeht und ob auch virtualisierte Server und

Netze zum Einsatz kommen sollen.

Die Planung MUSS Regelungen zu Netz-, CPU- und Festspeicherseparierung enthalten. Die

Separierung SOLLTE auch das Netzzonenkonzept und den Schutzbedarf beachten und auf diese

abgestimmt sein.

Anwendungen SOLLTEN jeweils in einem eigenen Kubernetes-Namespace laufen, der alle Programme

der Anwendung umfasst. Nur Anwendungen mit ähnlichem Schutzbedarf und ähnlichen möglichen

Angriffsvektoren SOLLTEN einen Kubernetes-Cluster teilen.