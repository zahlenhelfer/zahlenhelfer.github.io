---
layout: post
title: JSON-Schema für Helm per GitHubActions automatisch erstellen
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
published: false
---

## Das Problem: JSON-Schema für Helm nicht mehr händisch schreiben und pflegen!
Nach dem [letzten Post](https://zahlenhelfer.github.io/2025/05/02/JSONSchemaForTheHelp.html) über JSON-Schemas, stellt sich natürlich die Frage: 
"Gibt es Tools die das Erstellen und Pflegen von diesen Dateien für Helm vereinfachen können?"

## Die Lösung: automatische Schema Generierung aus der `values.yaml`
Hier sei das GitHub-Projekt [helm-values-schema-json](https://github.com/losisin/helm-values-schema-json/tree/main) genannt. Dort gibt es sowohl für für die Kommandozeile ein Tool, aber noch automatisierter eine GitHub-Action die das ganze dann noch bequemer erledigt. Ob also PreCommit-Hook oder entspannt im GitHub - Du kannst es Dir aussuchen. Das Tool liest die `values.yaml` aus und erstellt ein entsprechendes JSON-Schema. Wenn Du dabei auch noch Annotationen in der `values.yaml` hinzufügst, wird es noch magischer.

## Beispiel:  Schema Generierung per GitHub-Action

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