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
Durch Tools wie [act](https://github.com/nektos/act) kommt es bestimmt nicht mehr so häufig vor, aber manchmal möchtest Du einfach deinen Workflow in der GitHub-UI schnell noch einmal starten.

## Die Lösung: automatische Schema Generierung aus der `values.yaml`
Hier sei das GitHub-Projekt [helm-values-schema-json](https://github.com/losisin/helm-values-schema-json/tree/main) genannt. Dort gibt es  für für die Kommandozeile ein Tool.  Aber noch schöner ist die GitHub-Action, die das ganze dann noch bequemer erledigt. Ob Du also einen PreCommit-Hook nutzt oder entspannt dem GitHub-Runner die Arbeit überlässt,  Du tust deiner Chart Qualität etwas gutes. Das Tool liest dabei die `values.yaml` aus und erstellt ein entsprechendes JSON-Schema. Wenn Du dabei auch noch Annotationen in der `values.yaml` hinzufügst, wird es noch magischer! Lass uns schauen wie es funktionier.

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
Es sind manchmal die kleinen Dinge die den Tag besser machen. Natürlich ging es auch ohne den dispatcher-Knopf irgendwie. Aber so ist es schon einfacher. Danke GitHub!