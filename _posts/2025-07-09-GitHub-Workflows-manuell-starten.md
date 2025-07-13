---
layout: post
title: "Tip: GitHub-Workflows mit workflow_dispatch manuell starten"
category: github
tags:
  - blog
  - gha
  - github
  - tip
  - de
permalink: /:year/:month/:day/:title:output_ext
published: true
render_with_liquid: "false"
---

## Das Problem: GitHub-Workflows manuell starten
GitHub Actions ist ein mächtiges Tool zur Automatisierung von Softwareprozessen. Standardmäßig werden Workflows durch Ereignisse wie `Push` oder `Pull Requests` ausgelöst. Aber was, wenn man einen Workflow einfach manuell starten möchte?
## Die Lösung: `workflow_dispatch`
Genau dafür gibt es `workflow_dispatch`. Es ist ein Trigger, mit dem Du einen Workflow _manuell_ über die GitHub-Oberfläche starten kannst. Ideal für:
- On-Demand-Deployments
- Wartungsaufgaben
- Skripte einfach manuell starten (debugging)
 
## Beispiel:  ein manueller Workflow
Erstellen wir einen Github-Workflow.  z.B.`.github/workflows/manual.yml`

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
        run: echo "${{ github.event.inputs.environment }}"
```

Hier wird nun nur noch bei manueller Eingabe der Workflow gestartet. Das kannst Du natürlich mit `push` bzw. `pr` kombinieren. Jetzt kannst Du den gewünschten Workflow im Reiter **Actions** auswählen. Klicke dann auf **Run workflow**. 
![GitHub-Workflow-Dispatch](assets/images/gh-workflow-dispatch.png)
Gib die Eingaben an (falls wie im Beispiel definiert) und Bestätige mit **Run**. Das war es auch schon. Wenn der Workflow dann durchgelaufen ist, kann Du wie gewohnt die Logs inspizieren.
![GitHub-Workflow-Log](assets/images/gh-workflow-log.png)
Ganz praktisch und einfach. Weitere Ideen zu diesem Feature findest Du im original [Blog-Post](https://github.blog/changelog/2020-07-06-github-actions-manual-triggers-with-workflow_dispatch/) von GitHub.
## Fazit
Es sind manchmal die kleinen Dinge die den Tag besser machen. Mit `workflow_dispatch` erhältst Du die volle Kontrolle über deine Workflows. Ob für manuelle Deployments oder flexible Skripte - Danke GitHub!
