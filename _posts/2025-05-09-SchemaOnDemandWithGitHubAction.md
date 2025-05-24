---
layout: post
title: JSON-Schema für Helm-Charts automatisch per GitHub-Actions erstellen
category: kubernetes
tags:
  - blog
  - helm
  - json
  - k8s
  - gha
  - github
  - tip
  - de
  - schema
permalink: /:year/:month/:day/:title:output_ext
published: true
---

## Das Problem: JSON-Schema für Helm nicht mehr händisch schreiben und pflegen!
Nach dem [letzten Post](https://zahlenhelfer.github.io/2025/05/02/JSONSchemaForTheHelp.html) über JSON-Schemas mit Helm-Charts, stellt sich natürlich die Frage: 
"Gibt es Tools die das Erstellen und Pflegen von diesen Dateien für Helm vereinfachen können?"

## Die Lösung: automatische Schema Generierung aus der `values.yaml`
Hier sei das GitHub-Projekt [helm-values-schema-json](https://github.com/losisin/helm-values-schema-json/tree/main) genannt. Dort gibt es  für für die Kommandozeile ein Tool.  Aber noch schöner ist die GitHub-Action, die das ganze dann noch bequemer erledigt. Ob Du also einen PreCommit-Hook nutzt oder entspannt dem GitHub-Runner die Arbeit überlässt,  Du tust deiner Chart Qualität etwas gutes. Das Tool liest dabei die `values.yaml` aus und erstellt ein entsprechendes JSON-Schema. Wenn Du dabei auch noch Annotationen in der `values.yaml` hinzufügst, wird es noch magischer! Lass uns schauen wie es funktionier.

## Ein Beispiel:  Schema Generierung per GitHub-Action
Aber als erstes erstmal den GitHub-Workflow im Projekt erstellen (`.github/workflows/helm-json-values.yaml`)

```yaml
name: Generate values schema json
on:
  push:
    branches:
      - main

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.ref }}
      - name: Generate values schema json
        uses: losisin/helm-values-schema-json-action@v1
        with:
          input: values.yaml
          git-push: true
          git-commit-message: "chore: update values.schema.json"
```

Hier wird bei jedem `push` auf den `main`-Branch das Projekt ausgecheckt und aktuell nur die `values.yaml` verarbeitet. Die verwendete GitHub-Action unterstützt noch wesentlich mehr Spielarten, das Ganze ist hier bewusst einfach gehalten. Die erstellte `values.schema.json`wird dann mittels `git-push: true`commited. Denkt dabei bitte daran das der `commit` aus der Action auch die Berechtigung benötigt.  Schaut einfach mal in Eurem Repo unter `Settings -> Code and automation -> Actions -> General` und dann im Bereich `Workflow-Permissions`. 

![[gh-workflow-settings.png]]
Hier sollte im einfachsten Fall einfach ein `Read and write permissionsWorkflows have read and write permissions in the repository for all scopes.`ausgewählt sein.

## Mehr Magie in der values.yaml
Die erstellte `values.schema.json` ist recht generisch. Tolle Features wie die Werteliste `Always, Never, IfNotPresent` bei der `pullPolicy` aus dem [letzten Post](https://zahlenhelfer.github.io/2025/05/02/JSONSchemaForTheHelp.html)  sucht man vergebens. Hier können wir mit einem einfachen Trick nachschärfen.

```yaml
image:
  repository: ghcr.io/stevedetm/restart-pod-job # @schema pattern: ^[a-z0-9./-]+$
  pullPolicy: IfNotPresent # @schema enum: [IfNotPresent, Always, Never]
```

Direkt hinter der jeweiligen Wertzuweisung wird per `#` ein Kommentar erstellt. 
Die Annotation `@schema enum: [IfNotPresent, Always, Never]` weist dann entsprechend gültige Werte zu. Bei dem Repository können wir einen regulären Ausdruck hinterlegen. Weitere Annotationen wie `maxLength` findest Du in der [Dokumentation](https://github.com/losisin/helm-values-schema-json/tree/main/docs) der [GitHub](https://github.com/losisin/helm-values-schema-json)-Projektes.
## Fazit
Mit einer kleinen Automatisierung wird ganz automatisch eine lästige Pflegearbeit abgenommen. Sicherlich ist es immer noch nötig die `values.yaml` sinnvoll zu Annotieren, aber das sollte zu Dokumentationszwecken ja eh gemacht werden, oder?