---
layout: post
title: BSI Grundschutz - APP.4.4.A3 Identitäts- und Berechtigungsmanagement bei Kubernetes (B) - Teil 4
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

# BSI Grundschutz - APP.4.4.A3 Identitäts- und Berechtigungsmanagement bei Kubernetes (B) - Teil 4

>DISCLAIMER: Ich bin weder Auditor noch Anwalt - daher keine Gewähr!

Weiter geht es heute mit dem nächsten Baustein. Dieser erscheint eigentlich vom Text zu kurz für einen Blogpost, aber da steckt eine Menge drin. Also legen wir gleich los.
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