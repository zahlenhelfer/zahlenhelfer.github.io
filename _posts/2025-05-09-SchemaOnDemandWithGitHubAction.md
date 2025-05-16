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
published: false
---

## Das Problem: JSON-Schema für Helm nicht mehr händisch schreiben und pflegen!
Nach dem [letzten Post](https://zahlenhelfer.github.io/2025/05/02/JSONSchemaForTheHelp.html) über JSON-Schemas mit Helm-Charts, stellt sich natürlich die Frage: 
"Gibt es Tools die das Erstellen und Pflegen von diesen Dateien für Helm vereinfachen können?"

## Die Lösung: automatische Schema Generierung aus der `values.yaml`
Hier sei das GitHub-Projekt [helm-values-schema-json](https://github.com/losisin/helm-values-schema-json/tree/main) genannt. Dort gibt es  für für die Kommandozeile ein Tool.  Aber noch schöner ist die GitHub-Action, die das ganze dann noch bequemer erledigt. Ob Du also einen PreCommit-Hook nutzt oder entspannt dem GitHub-Runner die Arbeit überlässt,  Du tust deiner Chart Qualität etwas gutes. Das Tool liest dabei die `values.yaml` aus und erstellt ein entsprechendes JSON-Schema. Wenn Du dabei auch noch Annotationen in der `values.yaml` hinzufügst, wird es noch magischer! Lass uns schauen wie es funktionier.

## Beispiel:  Schema Generierung per GitHub-Action
Aber als erstes erstmal den GitHub-Workflow erstellen (`.github/.yaml`)

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