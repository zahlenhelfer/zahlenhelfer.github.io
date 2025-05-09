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
permalink: /:year/:month/:day/:title:output_ext
published: false
---

## Das Problem: JSON-Schema für Helm nicht mehr händisch schreiben und pflegen!
Nach dem letzten Post, stellt sich natürlich die Frage: 
"Gibt es Tools die das Erstellen und Pflegen von JSON-Schema Dateien für Helm vereinfachen"?

## Die Lösung: automatische Schema Generierung aus der `values.yaml`
Hier sei das GitHub-Projekt [helm-values-schema-json](https://github.com/losisin/helm-values-schema-json/tree/main) genannt. 

## Beispiel:  Schema Generierung per GitHub-Action


