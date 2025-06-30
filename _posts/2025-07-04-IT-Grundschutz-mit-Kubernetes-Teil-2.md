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
Jetzt geht es los. Als Erstes werden im APP.4.4 die Basis-Andorderungen beschrieben. Wie der Name es schon sagt, das ist der Grundstein. Das __MÜSSEN__ wir mit unserem Kubernetes mindestens erbringen. Starten wir daher mit dem ersten Baustein - APP.4.4.A1 - Planung der Separierung der Anwendungen (B).
### APP.4.4.A1 - Planung der Separierung der Anwendungen (B)

Nehmen wir den Text Absatz für Absatz auseinander
>Vor der Inbetriebnahme MUSS geplant werden, wie die in den Pods betriebenen Anwendungen und deren unterschiedlichen Test- und Produktions-Betriebsumgebungen separiert werden. Auf Basis des Schutzbedarfs der Anwendungen MUSS festgelegt werden, welche Architektur der Namespaces, Meta-Tags, Cluster und Netze angemessen auf die Risiken eingeht und ob auch virtualisierte Server und Netze zum Einsatz kommen sollen.

Unsere Anwendung wird auf jeden Fall einen eigenen `Namespace`bekommen. Das kannst Du entweder als eigene `Ressource`in deinen Manifestdateien vorsehen oder Falls Du `Helm`-Charts nutzt, dort auch explizit festlegen. 

Eine solche Anforderung sollte natürlich auch sicher gestellt werden. Dafür könnten wir die Kubernetes Manifeste scannen, oder das Policy Tool [Kyverno](https://kyverno.io/) einsetzen. Hier könnten wir mit der Policy [disallow-default-namespace](https://kyverno.io/policies/best-practices/disallow-default-namespace/disallow-default-namespace/) arbeiten um das Anlegen von Kubernetes-Ressourcen im `default`-Namespace zu verhindern. Wie Du Kyverno nutzt bzw. in Kubernetes installiert, erfährst Du im [Blogpost](TODO) über Kyverno.

Nicht zu vergessen ist auch der Teil der Meta-Tags. Gibt es ein Label-Konzept? Hier hilft sicherlich die [Kubernetes Dokumentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/) für den ersten Schritt. Aber neben diese Vorschlägen kann auch  z.B. der Schutzbedarf damit sichtbar gemacht werden. Beispl.: `Schutzbedarf=[normal,hoch,sehrhoch]` Und nicht nur Pods sollte über ein Konzept verfügen auch die Verwendung bei Nodes und Namespaces sollte nachgedacht werden. Hier kann Bei Nodes z.B. über Labels wie die Kennzeichnung von `Infranodes`nachgedacht werden.

>Die Planung MUSS Regelungen zu Netz-, CPU- und Festspeicherseparierung enthalten. Die Separierung SOLLTE auch das Netzzonenkonzept und den Schutzbedarf beachten und auf diese abgestimmt sein.

In diesem Absatz sind gleich zwei Dinge enthalten. TODO

>Anwendungen SOLLTEN jeweils in einem eigenen Kubernetes-Namespace laufen, der alle Programme der Anwendung umfasst. Nur Anwendungen mit ähnlichem Schutzbedarf und ähnlichen möglichen Angriffsvektoren SOLLTEN einen Kubernetes-Cluster teilen.

Hier kann es nun hart werden. Für Anwendungen mit ähnlichem Schutzbedarf darf ein Cluster genutzt werden. Kommt nun eine weitere Anwendung mit einem anderen bsplw. hohen Schutzbedarf sollte hier möglichst ein __eigener Cluster__ bereit gestellt werden. Dann kann sehr schnell die Anzahl der Cluster wachsen lassen und damit eventuell auch das Budget sprengen. Derzeit gibt es aber mit dem Produkt `vCluster` einen spannenden Ansatz auf einem physischen Kubernetescluster mehrere unabhängige Kubernetes-Cluster laufen zu lassen.

### APP.4.4.A2 Planung der Automatisierung mit CI/CD (B)
Der Baustein erscheint eigentlich vom Text zu kurz für einen Blogpost, aber da steckt eine Menge drin. Also legen wir gleich los.

>Wenn eine Automatisierung des Betriebs von Anwendungen in Kubernetes mithilfe von CI/CD stattfindet, DARF diese NUR nach einer geeigneten Planung erfolgen. 

Du willst ein CI/CD Tool wie Flux oder ArgoCD für die Automatisierung nutzen. Denn diese stellen sicher das dein Cluster auch nach installieren eine Anwendung mittele Manifesten oder Helm-Chart auch wirklich den geünschten Stand behält.

>Die Planung MUSS den gesamten Lebenszyklus von Inbetrieb bis Außerbetriebnahme inklusive Entwicklung, Tests, Betrieb, Überwachung und Updates umfassen. 

Lebenszyklus also von der Installation über Updates bis zur Stillegeung. Da hilft das Nutzen von Helm-Charts doch sehr. Die Anwendung ist damit paketiert und kann installiert, aktualisiert und wieder deinstalliert werden.

>Das Rollenund Rechtekonzept sowie die Absicherung von Kubernetes Secrets MÜSSEN Teil der Planung sein.

Und auch hier wieder zwei Teile. Der RBAC Teil das dass Rechte und Rollenkonzept beinhaltet. Und die `Secrets` die z.B. per External-Secret-Operator oder Sealed Secrets oder mit etcd-Encryption gesichert werden können. 

### APP.4.4.A3 Identitäts- und Berechtigungsmanagement bei Kubernetes (B) 

>Kubernetes und alle anderen Anwendungen der Control Plane MÜSSEN jede Aktion eines Benutzenden oder, im automatisierten Betrieb, einer entsprechenden Software authentifizieren und autorisieren, unabhängig davon, ob die Aktionen über einen Client, eine Weboberfläche oder über eine entsprechende Schnittstelle (API) erfolgt.

>Administrative Handlungen DÜRFEN NICHT anonym erfolgen.

>Benutzende DÜRFEN NUR die unbedingt notwendigen Rechte erhalten. Berechtigungen ohne Einschränkungen MÜSSEN sehr restriktiv vergeben werden. Nur ein kleiner Kreis von Personen SOLLTE berechtigt sein, Prozesse der Automatisierung zu definieren. 

>Nur ausgewählte Administrierende SOLLTEN in Kubernetes das Recht erhalten, Freigaben für Festspeicher (Persistent Volumes) anzulegen oder zu ändern.

### APP.4.4.A4 Separierung von Pods (B)

>Der Betriebssystem-Kernel der Nodes MUSS über Isolationsmechanismen zur Beschränkung von Sichtbarkeit und Ressourcennutzung der Pods untereinander verfügen (vergleiche Linux Namespaces und cgroups). Die Trennung MUSS dabei mindestens IDs von Prozessen sowie Benutzenden, Inter-Prozess-Kommunikation, Dateisystem und Netz inklusive Hostname umfassen.

### APP.4.4.A5 Datensicherung im Cluster (B)

>Es MUSS eine Datensicherung des Clusters erfolgen. Die Datensicherung MUSS umfassen: 
>• Festspeicher (Persistent Volumes),
>• Konfigurationsdateien von Kubernetes und den weiteren Programmen der Control Plane,
>• den aktuellen Zustand des Kubernetes-Clusters inklusive der Erweiterungen,
>• Datenbanken der Konfiguration, namentlich hier etcd, 
>• alle Infrastrukturanwendungen, die zum Betrieb des Clusters und der darin befindlichen Dienste notwendig sind und 
>• die Datenhaltung der Code und Image Registries.
> Es SOLLTEN auch Snapshots für den Betrieb der Anwendungen betrachtet werden. Snapshots DÜRFEN die Datensicherung NICHT ersetzen.