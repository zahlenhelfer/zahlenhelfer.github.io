---
layout: post
title: BSI Grundschutz trifft auf Kubernetes-Cluster - Teil 1
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

Das IT-Grundschutzkompendium befasst sich auch mit Kubernetes. Die Damen und Herren des BSI haben dem Produkt "IT‑Grundschutz‑Bausteine" sogar einen eigenen Baustein gewidmet - [APP.4.4 Kubernetes](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Grundschutz/IT-GS-Kompendium_Einzel_PDFs_2022/06_APP_Anwendungen/APP_4_4_Kubernetes_Edition_2022.pdf?__blob=publicationFile&v=3). Ich möchte in dieser kleinen Serie gerade die Bausteine beleuchten und mögliche Implementierungen aufzeigen.

>DISCLAIMER: Ich bin weder Auditor noch Anwalt - daher keine Gewähr!

Bevor ich jetzt die 3.242te-Einführung in den IT-Grundschutz mache, gehen wir in dieser Serie die Bausteine einfach durch. 

Aber ein minimales Verständnis vorab für die Dinge kann nicht Schaden:
- Die BSI-Standards 200-1 bis 200-4
- Das IT-Grundschutz-Kompendium insb. die IT‑Grundschutz‑Bausteine

### BSI‑Standard 200‑1 oder "Was muss ich tun"

- Definiert allgemeine Anforderungen an ein ISMS und ist kompatibel zur ISO 27001 und ISO 27002 [Quelle: BSI](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/BSI-Standards/bsi-standards_node.html?utm_source=chatgpt.com).
- Betont einen ganzheitlichen Ansatz über technische, organisatorische und personelle Ebenen hinweg [Quelle: BSI](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Grundschutz/BSI_Standards/standard_200_1.pdf?__blob=publicationFile&v=2&utm_source=chatgpt.com).
- Legt besonderen Wert auf die Verantwortung und Mitwirkung der Führungsebene [Quelle: BSI](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Grundschutz/BSI_Standards/standard_200_1.pdf?__blob=publicationFile&v=2&utm_source=chatgpt.com).

### BSI‑Standard 200‑2 oder "Wie muss ich es tun"

- Beschreibt die IT‑Grundschutz‑Methodik und bietet drei Vorgehensweisen: Basis-, Standard- und Kern-Absicherung [Quelle Wikipedia](https://de.wikipedia.org/wiki/IT-Grundschutz?utm_source=chatgpt.com).
- Stellt die methodische Grundlage für ISMS-Aufbau gemäß IT‑Grundschutz dar [Quelle Wikipedia](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/BSI-Standards/bsi-standards_node.html?utm_source=chatgpt.com).
- Ist strukturell nah an 200‑1, was das parallele Nutzen beider Dokumente erleichtert [Quelle Wikipedia](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/BSI-Standards/bsi-standards_node.html?utm_source=chatgpt.com).

### BSI‑Standard 200‑3 - "Wie gehe ich mit Risiken um"

- Konsolidiert alle risikobezogenen Arbeitsschritte zur Umsetzung des IT‑Grundschutzes [Quelle BSI](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/BSI-Standards/bsi-standards_node.html?utm_source=chatgpt.com).
- Nutzt eine vereinfachte, auf Basis des Kompendiums angelegte Risikoanalyse [Quelle Wikipedia](https://de.wikipedia.org/wiki/IT-Grundschutz?utm_source=chatgpt.com).
- Senkt den Aufwand für risiko­bezogene Maßnahmen und eignet sich besonders für Anwender, die bereits mit IT‑Grundschutz arbeiten [Quelle BSI](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/BSI-Standards/bsi-standards_node.html?utm_source=chatgpt.com).

### BSI‑Standard 200‑4 - "Wie bleibt es am Leben"
- Führt praxisnah in den Aufbau eines Business Continuity Management Systems (BCMS) ein [Quelle BSI](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/BSI-Standards/bsi-standards_node.html?utm_source=chatgpt.com).
- Bietet ein normatives Mapping zur ISO 22301:2019 und richtet sich an Einsteiger wie erfahrene BCM-Nutzer [Quelle BSI](https://de.wikipedia.org/wiki/IT-Grundschutz?utm_source=chatgpt.com).

### Leitfaden zur Basis-Absicherung
- Liefert einen kompakten Einstieg in die IT‑Grundschutz‑Methodik, ideal für kleine und mittlere Institutionen [Quelle BSI](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/BSI-Standards/bsi-standards_node.html?utm_source=chatgpt.com).

### IT-Grundschutz-Bausteine
- Die IT‑Grundschutz‑Bausteine sind ein strukturiertes, praxisnahes und umfassendes Baukastensystem zur ganzheitlichen Informationssicherheit – modular, regelmäßig aktualisiert und mit umsetzungsstarken Leitfäden.

### Struktur der IT-Grundschutz-Bausteine
- **Definition & Ziel**  
    Die Bausteine bilden das Kernstück des IT‑Grundschutz-Kompendiums und enthalten Anforderungen sowie Empfehlungen zu Sicherheit für IT-Systeme, Prozesse, Netze, Gebäude und Anwendungen [bsi.bund.de+15bsi.bund.de+15bsi.bund.de+15](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/IT-Grundschutz-Kompendium/IT-Grundschutz-Bausteine/Bausteine_Download_Edition_node.html?utm_source=chatgpt.com).
    
- **Struktur & Aufbau**  
    Sie sind modular aufgebaut und folgen einem einheitlichen Schema mit Beschreibung, Gefährdungen und drei Anforderungsklassen: Basis, Standard, erhöhter Schutzbedarf [bsi.bund.de+1tenfold-security.com+1](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/Zertifizierte-Informationssicherheit/IT-Grundschutzschulung/Online-Kurs-IT-Grundschutz/Lektion_5_Modellierung/Lektion_5_01/Lektion_5_01_node.html?utm_source=chatgpt.com).
    
- **Schichtenmodell**  
    Es gibt zehn thematische Schichten: ISMS (Sicherheitsmanagement), ORP (Organisation & Personal), CON (Konzeption), OPS (Betrieb), DER (Detektion/Reaktion), APP (Anwendungen), SYS (Systeme), IND (industrielle IT), NET (Netze) und INF (Infrastruktur) [ausbildung-in-der-it.de+8bsi.bund.de+8tenfold-security.com+8](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/IT-Grundschutz-Kompendium/it-grundschutz-kompendium_node.html?utm_source=chatgpt.com).
    
- **Reduzierter Analyseaufwand**  
    Durch vordefinierte Risiken und empfohlene Maßnahmen entfällt oft eine individuelle Risikoanalyse – ein Soll‑Ist‑Abgleich genügt meist [tenfold-security.com](https://www.tenfold-security.com/bsi-it-grundschutz/?utm_source=chatgpt.com).
    
- **Aktualisierung & Community**  
    Das Kompendium erscheint jährlich (z. B. Edition 2023 mit 111 Bausteinen) und beinhaltet Community-Drafts sowie nutzererstellte Bausteine [bsi.bund.de+6bsi.bund.de+6bsi.bund.de+6](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/IT-Grundschutz-Kompendium/it-grundschutz-kompendium_node.html?utm_source=chatgpt.com).

- **Umsetzungshinweise**  
    Zu vielen Bausteinen gibt es vertiefende Hinweise zur konkreten Umsetzung der empfohlenen Maßnahmen [it-grundschutzkompendium.de+3bsi.bund.de+3tenfold-security.com+3](https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/IT-Grundschutz-Kompendium/it-grundschutz-kompendium_node.html?utm_source=chatgpt.com).