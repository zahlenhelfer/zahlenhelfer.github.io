---
layout: post
title: "Tip: --kubelet-insecure-tls NICHT mehr nutzen"
category: kubernetes
tags:
  - blog
  - tip
  - de
  - kubernetes
  - tls
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---

## Das Problem: --kubelet-insecure-tls und der metrics-server
Bei vielen meiner Trainings für das Thema Kubernetes kommt der Punkt wo wir den [metrics-server](https://github.com/kubernetes-sigs/metrics-server) installieren. Sei es um den Horizontal-Pod-Autoscaler zu zeigen oder einfach damit `kubectl top pod` funktioniert. Dabei gibt es jedesmal die Meldung:
```
E0108 13:29:15.336920 1 scraper.go:149] "Failed to scrape node" err="Get \"https://167.71.63.166:10250/metrics/resource\": tls: failed to verify certificate: x509: cannot validate certificate for 167.71.63.166 because it doesn't contain any IP SANs" node="k8s-node-0"
```
Was ist passiert? Nun, der erfahrene Kubernetes-Admin weiss, der Subject-Alternative Name oder auch SAN ist nicht korrekt gesetzt. Der `metrics-server` möchte gerne per IP-Adresse und nicht per DNS Namen die Zertifikate validieren. Dort fehlt gerade bei selbst erstellten Kubernetes-Nodes häufig die IP Adresse im SAN-Teil des  Zertifikats.
## Keine Lösung: `--kubelet-insecure-tls`
Genau dafür gibt es `workflow_dispatch`. Es ist ein Trigger, mit dem Du einen Workflow _manuell_ über die GitHub-Oberfläche starten kannst. Ideal für:
- On-Demand-Deployments
- Wartungsaufgaben
- Skripte einfach manuell starten (debugging)

## Die Lösung: `csr approve`
Das Kubelet kann ein Zertifikat bei der Kubernetes-CA ausstellen lassen. Der ganze Aufwand mit `openssl` kann dabei mitlerweile elegant umgangen werden. Drei einfache Schritte machen es Möglich:
- Die kubelete-config erweitern
- Den Dienst `kubelet` neu starten und damit einen `csr` auslösen
- Den `certificate signing request` in Kubernetes genehmigen

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
Manchmal ist es einfach kein Aufwand die Dinge richtig zu machen. Auch bei diesem Es sind manchmal die kleinen Dinge die den Tag besser machen. Mit `workflow_dispatch` erhältst Du die volle Kontrolle über deine Workflows. Ob für manuelle Deployments oder flexible Skripte - Danke GitHub!
