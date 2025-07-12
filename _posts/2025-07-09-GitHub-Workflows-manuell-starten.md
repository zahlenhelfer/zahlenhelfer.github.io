---
layout: post
title: "Tip: GitHub-Workflows in der GUI manuell starten"
category: kubernetes
tags:
  - blog
  - gha
  - github
  - tip
  - de
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---

## Das Problem: GitHub-Workflows nochmal starten
GitHub Actions ist ein mächtiges Tool zur Automatisierung von Softwareprozessen. Standardmäßig werden Workflows durch Ereignisse wie `Push` oder `Pull Requests` ausgelöst. Aber was, wenn man einen Workflow manuell starten möchte? 

## Die Lösung: `workflow_dispatch`
Genau dafür gibt es `workflow_dispatch`.

## Ein Beispiel:  Schema Generierung per GitHub-Action
Aber als erstes suche Dir deinen GitHub-Workflow (`.github/workflows/helm-json-values.yaml`)

```yaml
name: some work to do
on:
  push:
    branches:
      - main
...
```

Hier wird bei jedem `push` auf den `main`-Branch das Projekt ausgecheckt und aktuell nur die `values.yaml` verarbeitet. Die verwendete GitHub-Action unterstützt noch wesentlich mehr Spielarten, das Ganze ist hier bewusst einfach gehalten. Die erstellte `values.schema.json`wird dann mittels `git-push: true`commited. Denkt dabei bitte daran das der `commit` aus der Action auch die Berechtigung benötigt.  Schaut einfach mal in Eurem Repo unter `Settings -> Code and automation -> Actions -> General` und dann im Bereich `Workflow-Permissions`.

![GitHub-Settings](assets/images/gh-workflow-dispatch.png)

Hier sollte im einfachsten Fall einfach ein `Read and write permissions` ausgewählt sein.

Weitere Ideen zu diesem Feature findest Du im original [Blog-Post](https://github.blog/changelog/2020-07-06-github-actions-manual-triggers-with-workflow_dispatch/) von GitHub.
## Fazit
Es sind manchmal die kleinen Dinge die den Tag besser machen. Mit `workflow_dispatch` erhältst Du die volle Kontrolle über deine Workflows. Ob für manuelle Deployments oder flexible Skripte - Danke GitHub!

---

### 🧩 Was ist `workflow_dispatch`?

`workflow_dispatch` ist ein Trigger, mit dem du einen Workflow _manuell_ über die GitHub-Oberfläche starten kannst. Ideal für:

- On-Demand-Deployments
- Wartungsaufgaben
- Skripte mit auswählbaren Eingaben

### 🛠 Beispiel-Workflow

```yaml
name: Manuell starten

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Umgebung'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

jobs:
  run-script:
    runs-on: ubuntu-latest
    steps:
      - name: Zeige Eingabe
        run: echo "Ausgewählte Umgebung: ${{ github.event.inputs.environment }}"
```

Dieser Workflow kann über die GitHub-Oberfläche manuell gestartet werden, inklusive Auswahl der Umgebung per Dropdown.

### 🚀 So startest du den Workflow

1. Gehe zum Repository auf GitHub.
    
2. Klicke auf den Reiter **Actions**.
    
3. Wähle den gewünschten Workflow aus der Liste.
    
4. Klicke auf **Run workflow**.
    
5. Gib die Eingaben an (falls definiert).
    
6. Bestätige mit **Run**.
    
