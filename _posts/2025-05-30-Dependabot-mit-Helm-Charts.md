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
published: false
render_with_liquid: "false"
---

## Das Problem: Helm-Chart Dependencies automatisch prüfen und pflegen
Sicherlich nutzen viele von Euch Umbrella-Chart als Pattern für eure Helm-Charts. Immer wieder stellt sich da die Frage: "Wie bekomme ich Updates bei meinen Helm-Chart dependencies mit?"

## Die Lösung: automatische PRs bei Helm-Chart updates
Seit April


Hier sei das GitHub-Projekt [helm-values-schema-json](https://github.com/losisin/helm-values-schema-json/tree/main) genannt. Dort gibt es  für für die Kommandozeile ein Tool.  Aber noch schöner ist die GitHub-Action, die das ganze dann noch bequemer erledigt. Ob Du also einen PreCommit-Hook nutzt oder entspannt dem GitHub-Runner die Arbeit überlässt,  Du tust deiner Chart Qualität etwas gutes. Das Tool liest dabei die `values.yaml` aus und erstellt ein entsprechendes JSON-Schema. Wenn Du dabei auch noch Annotationen in der `values.yaml` hinzufügst, wird es noch magischer! Lass uns schauen wie es funktionier.

## Ein Beispiel:  Automatische PR bei Updates per Dependabot
Aber als erstes erstmal den GitHub-Workflow im Projekt erstellen (`.github/dependabot.yaml`)

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

Hier wird bei jedem `push` auf den `main`-Branch das Projekt ausgecheckt und aktuell nur die `values.yaml` verarbeitet. Die verwendete GitHub-Action unterstützt noch wesentlich mehr Spielarten, das Ganze ist hier bewusst einfach gehalten. Die erstellte `values.schema.json`wird dann mittels `git-push: true`commited. Denkt dabei bitte daran das der `commit` aus der Action auch die Berechtigung benötigt.  Schaut einfach mal in Eurem Repo unter `Settings -> Code and automation -> Actions -> General` und dann im Bereich `Workflow-Permissions`.

![GitHub-Settings](assets/images/gh-workflow-settings.png)

Hier sollte im einfachsten Fall einfach ein `Read and write permissions` ausgewählt sein.

## Mehr Magie in der values.yaml
Die erstellte `values.schema.json` ist recht generisch. Tolle Features wie die Werteliste `Always, Never, IfNotPresent` bei der `pullPolicy` aus dem [letzten Post](https://zahlenhelfer.github.io/2025/05/02/JSONSchemaForTheHelp.html)  sucht man vergebens. Hier können wir mit einem einfachen Trick nachschärfen.

```yaml
image:
  repository: ghcr.io/stevedetm/restart-pod-job # @schema pattern: ^[a-z0-9./-]+$
  pullPolicy: IfNotPresent # @schema enum: [IfNotPresent, Always, Never]
```

Direkt hinter der jeweiligen Wertzuweisung wird per `#` ein Kommentar erstellt. 
Die Annotation `@schema enum: [IfNotPresent, Always, Never]` weist dann entsprechend gültige Werte zu. Bei dem Repository können wir einen regulären Ausdruck hinterlegen. Weitere Annotationen wie `maxLength` und mehr findest Du in der [Dokumentation](https://github.com/losisin/helm-values-schema-json/tree/main/docs) der [GitHub](https://github.com/losisin/helm-values-schema-json)-Projektes.
## Fazit
Mit einer kleinen Automatisierung wird eine lästige Pflegearbeit abgenommen. Sicherlich ist es immer noch nötig die `values.yaml` sinnvoll zu Annotieren, aber das sollte zu Dokumentationszwecken ja eh gemacht werden, oder?