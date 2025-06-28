---
layout: post
title: BSI Grundschutz APP.4.4.A1 - Planung der Separierung der Anwendungen - Teil 2
category: kubernetes
tags:
  - blog
  - de
  - bsi
  - grundschutz
  - kubernetes
  - kyverno
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---

## BSI Grundschutz und Kubernetes - passt das zusammen? - Erfahrungen aus der Praxis

>DISCLAIMER: Ich bin weder Auditor noch Anwalt - daher keine Gewähr!

## APP.4.4 Kubernetes - die Basis-Anforderungen
Jetzt geht es los. Als Erstes werden im APP.4.4 die Basis-Andorderungen beschrieben. Wie der Name es schon sagt, das ist der Grundstein. Das __MÜSSEN__ wir mit unserem Kubernetes mindestens erbringen. Starten wir daher mit dem ersten Baustein - APP.4.4.A1 - Planung der Separierung der Anwendungen (B).
### APP.4.4.A1 - Planung der Separierung der Anwendungen (B)

Nehmen wir den Text Absatz für Absatz auseinander
>Vor der Inbetriebnahme MUSS geplant werden, wie die in den Pods betriebenen Anwendungen und deren unterschiedlichen Test- und Produktions-Betriebsumgebungen separiert werden. Auf Basis des Schutzbedarfs der Anwendungen MUSS festgelegt werden, welche Architektur der Namespaces, Meta-Tags, Cluster und Netze angemessen auf die Risiken eingeht und ob auch virtualisierte Server und Netze zum Einsatz kommen sollen.

Unsere Anwendung wird auf jeden Fall einen eigenen `Namespace`bekommen. Das kannst Du entweder als eigene `Ressource`in deinen Manifestdateien vorsehen oder Falls Du `Helm`-Charts nutzt, dort auch explizit festlegen. 

Eine solche Anforderung sollte natürlich auch sicher gestellt werden. Dafür könnten wir die Kubernetes Manifeste scannen, oder das Policy Tool [Kyverno](https://kyverno.io/) einsetzen. Hier könnten wir mit der Policy [disallow-default-namespace](https://kyverno.io/policies/best-practices/disallow-default-namespace/disallow-default-namespace/) arbeiten um das anlegen von Kubernetes-Ressourcen im `default`-Namespace zu verhindern.

>Die Planung MUSS Regelungen zu Netz-, CPU- und Festspeicherseparierung enthalten. Die Separierung SOLLTE auch das Netzzonenkonzept und den Schutzbedarf beachten und auf diese abgestimmt sein.

In diesem Absatz sind gleich zwei Dinge enthalten. TODO

>Anwendungen SOLLTEN jeweils in einem eigenen Kubernetes-Namespace laufen, der alle Programme der Anwendung umfasst. Nur Anwendungen mit ähnlichem Schutzbedarf und ähnlichen möglichen Angriffsvektoren SOLLTEN einen Kubernetes-Cluster teilen.

Hier wird es hart bzw. kann bitter werden. Entweder werden für unterschiedliche Cluster 