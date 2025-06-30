---
layout: post
title: Helm-Chart Dependencies mit GitHub-Dependabot managen
category: kubernetes
tags:
  - blog
  - helm
  - gha
  - github
  - tip
  - de
  - dependabot
permalink: /:year/:month/:day/:title:output_ext
published: true
render_with_liquid: "false"
---

## Das Problem: Helm-Chart Dependencies automatisch prüfen und pflegen
Sicherlich nutzen viele von Euch Umbrella-Chart als Pattern für eure Helm-Charts. Immer wieder stellt sich da die Frage: "Wie bekomme ich Updates bei meinen Helm-Charts (Dependencies) mit?"

## Die Lösung: automatische PRs bei Helm-Chart updates
Seit April wurde ein [GitHub-Feature](https://github.com/dependabot/dependabot-core/issues/2237) für Dependabot fertig gestellt, dass für uns sehr hilfreich ist. Dependabot hat nun Unterstützung für Helm-Charts.

## Ein Beispiel:  Automatische PR bei Updates per Dependabot
Als erstes die GitHub-Setting des Repos aufrufen. Nun suchst Du in der Leiste nach `Settings -> Security-> Advanced Security`.

![GitHub-Security Settings](assets/images/gh-adv-security.png)

Auf der Seite suchst Du nach dem Eintrag `Dependabot version updates` und bestätigst mit `Configure`

![Dependabot Settings](assets/images/gh-dependabot-version.png)

Jetzt öffnet sich der GitHub-Editor und Du kannst die folgenden Zeilen in die `dependabot.yml` Datei übertragen.
```yaml
version: 2
updates:
  # Enable version updates for Helm
  - package-ecosystem: "helm"
    # Look for a `Chart.yaml` in the `root` directory
    directory: "/"
    # Check for updates once a week
    schedule:
      interval: "weekly"
```

Jetzt werden wöchentlich deine Chartabhänigkeiten geprüft und falls nötig ein PR mit dem Update erstellt.
![Dependabot Settings](assets/images/gh-pr-dependabot.png)
Nimmst Du diesen an, wird das Update ausgeführt und gemerged. 

Viel Spass!
## Fazit
- fire-and-forget Einstellung
- durch Pullrequests nicht invasiv und steuerbar
- wäre aber auch mit Tools wie [renovate](https://www.mend.io/renovate/) darstellbar
