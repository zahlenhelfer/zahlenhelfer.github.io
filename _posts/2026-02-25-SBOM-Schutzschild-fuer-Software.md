---
layout: post
title: "SBOM: Dein unsichtbarer Schutzschild für sichere Software"
category: kubernetes
tags:
  - blog
  - de
  - security
  - sbom
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---

# SBOM: Dein unsichtbarer Schutzschild für sichere Software

Es gibt Momente im Leben, da ist Timing einfach alles. Da wollte ich eigentlich von den Containerdays schreiben und vor allen die Kubernetes IT-Grundschutz Thematik weiter treiben, aber dann kam die Nachricht: [Neuer npm-Großangriff: Hunderte Pakete mit selbst-vermehrender Malware infiziert](https://www.heise.de/news/Neuer-NPM-Grossangriff-Selbst-vermehrende-Malware-infiziert-Dutzende-Pakete-10651111.html). Ich erinnerte mich sofort an meinen Certified Kubernetes Security Kurs von der Linuxfoundation. Aber da nutzte ich bisher doch immer das SBOM Format und wie war das jetzt nochmal mit dem SBOM. Ein praktischer kurzer Überblick. 
### SBOM - Die Zutatenliste deiner Software

**Software Bill of Materials (SBOM)** ist eine detaillierte Liste aller Komponenten, Bibliotheken und Abhängigkeiten, die in deiner Anwendung stecken. Stell Dir vor, es ist wie die Zutatenliste auf einem Lebensmittel: Ohne sie weißt du nicht, was wirklich drin ist – und ob etwas Gefährliches dabei sein könnte.

## Praxisbeispiel: Log4Shell mit SBOM

Erinnerst du dich an **Log4Shell (CVE-2021-44228)?** Diese kritische Lücke in **Apache Log4j** ermöglichte Angreifern, beliebigen Code auszuführen – und betraf Millionen von Systemen weltweit. Ohne SBOM war die Suche nach betroffenen Anwendungen ein **Albtraum**.
### Ohne SBOM: Chaos und Unsicherheit
- Dein Team nutzt eine komplexe Java-Anwendung mit dutzenden Abhängigkeiten.
- Die Warnung zu Log4Shell kommt – aber **weißt du überhaupt, ob deine Software Log4j verwendet?**
- Manuelle Checks dauern **Tage oder Wochen**, während Angreifer bereits aktiv sind.
- Die Configuration-Items in der CMDB deines ITIL-Systems sind zwar gepflegt, aber doch keine Bibliotheken!
- Die Frage nach dem Impact auf die Firma bzw. Ihre Kunden kannst Du deiner Geschäftsführung gerade sehr schlecht darlegen.

### Mit SBOM: Klare Sicht und schnelle Lösung
1. **Deine SBOM liegt bereit** – entweder im **SPDX-** oder **CycloneDX-Format** (dazu gleich mehr).
2. **Automatisierte Tools** (wie **Dependency-Track, Snyk oder GitHub Advisory Database**) scannen deine SBOM nach der betroffenen Log4j-Version (z. B. **2.14.1**).
3. **Innerhalb von Minuten** siehst du:
   - Welche deiner Anwendungen Log4j nutzen.
   - Ob die anfällige Version im Einsatz ist.
   - Informationen an die Geschäftsführung sind klar und strukturiert
1. **Sofortige Gegenmaßnahmen:**
   - Du spielst den Patch ein oder deaktivierst JNDI-Lookups als Workaround.
   - Deine Systeme sind **bevor Angreifer zuschlagen** geschützt.

**Ergebnis:** Statt wochenlanger Panik und unsicherer Systeme hast du **Kontrolle und Handlungsfähigkeit**.

#### Merke: **Eine SBOM hilft dir  Sicherheitslücken schneller zu finden** 

### SBOM eine Frage des Formats - SPDX vs. CycloneDX

Es ist ein bisschen wie VHS und Video 2000, an dieser Stelle Entschuldigung an alle aus den Jahrgängen 2000+. Nicht jede SBOM ist gleich. Die beiden wichtigsten **Standardformate** sind **SPDX** und **CycloneDX**. Beide haben ihre Stärken – je nachdem, was du brauchst.

#### Merke: **Eine SBOM hilft dir Compliance-Anforderungen zu erfüllen**