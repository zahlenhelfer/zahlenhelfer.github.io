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

# BSI Grundschutz APP.4.4.A1 - Planung der Separierung der Anwendungen - Teil 2

>DISCLAIMER: Ich bin weder Auditor noch Anwalt - daher keine Gewähr!

## APP.4.4 Kubernetes - die Basis-Anforderungen
Jetzt geht es los. Als Erstes werden im APP.4.4 die Basis-Anforderungen beschrieben. Wie der Name es schon sagt, das ist der Grundstein. Das __MÜSSEN__ wir mit unserem Kubernetes mindestens erbringen. Starten wir daher mit dem ersten Baustein.
### APP.4.4.A1 - Planung der Separierung der Anwendungen (B)

Nehmen wir jetzt den Text Absatz für Absatz auseinander:
>Vor der Inbetriebnahme MUSS geplant werden, wie die in den Pods betriebenen Anwendungen und deren unterschiedlichen Test- und Produktions-Betriebsumgebungen separiert werden. Auf Basis des Schutzbedarfs der Anwendungen MUSS festgelegt werden, welche Architektur der Namespaces, Meta-Tags, Cluster und Netze angemessen auf die Risiken eingeht und ob auch virtualisierte Server und Netze zum Einsatz kommen sollen.

Unsere Anwendung wird auf jeden Fall einen eigenen `Namespace`bekommen. Das kannst Du entweder als eigene `Ressource`in deinen Manifest-Dateien vorsehen oder Falls Du `Helm`-Charts nutzt, dort auch explizit festlegen.

Aber eine solche Anforderung sollte natürlich auch sicher gestellt werden. Dafür könnten wir die Kubernetes Manifeste vor dem Deployment scannen, oder auch das Policy Tool [Kyverno](https://kyverno.io/) einsetzen. Hier wird dann mit der Policy [disallow-default-namespace](https://kyverno.io/policies/best-practices/disallow-default-namespace/disallow-default-namespace/) gearbeitet.  Es verhindert dass Anlegen von Kubernetes-Ressourcen im `default`-Namespace . Wie Du Kyverno nutzt bzw. in Kubernetes installiert, erfährst Du in einem kommenden Blogpost über Kyverno.

Nicht zu vergessen ist auch der Teil der Meta-Tags. Gibt es ein Label-Konzept? Hier hilft sicherlich die [Kubernetes Dokumentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/) für den ersten Schritt. Aber neben diese Vorschlägen kann auch  z.B. der Schutzbedarf damit sichtbar gemacht werden. Beispl.: `Schutzbedarf=[normal,hoch,sehrhoch]` Und nicht nur Pods sollte über ein Konzept verfügen, auch die Verwendung bei Nodes und Namespaces sollte überdacht werden. Hier kann bei Nodes z.B. über Labels wie `Infranodes` nachgedacht werden.

>Die Planung MUSS Regelungen zu Netz-, CPU- und Festspeicherseparierung enthalten. Die Separierung SOLLTE auch das Netzzonenkonzept und den Schutzbedarf beachten und auf diese abgestimmt sein.

In diesem Absatz sind gleich zwei Dinge enthalten. TODO

>Anwendungen SOLLTEN jeweils in einem eigenen Kubernetes-Namespace laufen, der alle Programme der Anwendung umfasst. Nur Anwendungen mit ähnlichem Schutzbedarf und ähnlichen möglichen Angriffsvektoren SOLLTEN einen Kubernetes-Cluster teilen.

Hier kann es nun hart werden. Für Anwendungen mit ähnlichem Schutzbedarf darf __ein__ Cluster genutzt werden. Kommt nun eine weitere Anwendung mit einem anderen bsplw. hohen Schutzbedarf sollte hier möglichst ein __eigener Cluster__ bereit gestellt werden. Damit kann sehr schnell die Anzahl der Cluster wachsen und damit eventuell auch das Budget sprengen. Neben dem aufsetzten von mehreren physischen Clustern gibt es auch andere Alternativen. Das Produkt `vCluster` z.B. hat einen spannenden Ansatz auf einem physischen Kubernetescluster mehrere unabhängige Kubernetes-Cluster laufen zu lassen. Dabei ist jedes Cluster nur ein `Namespace` für den "Vater"-Cluster. Mehr Technik und eine Demo wie immer, in einem eigenen Blog-Post.