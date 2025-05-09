---
layout: post
title: Bessere Helm-Charts mit JSON Schema Validierung
category: kubernetes
tags:
  - blog
  - helm
  - json
  - k8s
  - tip
  - de
permalink: /:year/:month/:day/:title:output_ext
published: true
---

## Das Problem: falsche Datentypen, Felder oder Werte in der `values.yaml`

Helm ist das bevorzugte Paketmanagement-Tool für Kubernetes. Doch gerade bei der Arbeit mit der `values.yaml`-Datei schleichen sich leicht Fehler ein. Beispiele dafür sind:

- Typ-Validierung: Der Wert `image.tag` ist ein String, daher "1.5" und nicht 1.5
- Bereichs-Validierung: Der Wert für `replicaCount` sollte z.B. zwischen 1 und 10 liegen.
- Einschränkungs-Validierung: Die `image.pullPolicy` hat die gültigen Werte `IfNotPresent`, `Always` und `Never`

```yaml
# values.yaml
replicaCount: 2
image:
  tag: "1.5"
  pullPolicy: IfNotPresent
```

## Die Lösung: validieren – bevor deployed wird!
Hier kommt [JSON Schema](https://json-schema.org/) ins Spiel. Seit [Helm 3.5](https://helm.sh/docs/faq/changes_since_helm2/#validating-chart-values-with-jsonschema) ist es möglich eine `values.schema.json` im Chart-Verzeichnis zu hinterlegen. Diese beschreibt die Struktur. Jetzt kann Helm, die `values.yaml`-Datei beim Rendern validieren und notfalls meckern – das passiert automatisch bei:

- `helm install`
- `helm upgrade`
- `helm template`
- `helm lint`

## Ein Beispiel: `image.repository` und `image.pullpolicy`
Beim `image.repository`-Wert soll per RegEx geprüft werden, ob dieser ein Valides Docker-Image ist. Die `image.pullpolicy` kennt nur drei Werte, damit wäre alles anderen falsch.

```yaml
image.repository # Docker-Image Bspl.: "acmeorg/nginx"
image.pullPolicy # Werte: [IfNotPresent, Always, Never]
```

Mit einer einfachen `values.schema.json` bekommt Helm das ganze mit und kann reagieren.

Beispiel der values.schema.json:
```json
{
  "$schema": "http://json-schema.org/schema#",
  "type": "object",
  "required": [
    "image"
  ],
  "properties": {
    "image": {
      "type": "object",
      "required": [
        "repository",
        "pullPolicy"
      ],
      "properties": {
        "repository": {
          "type": "string",
          "pattern": "^[a-z0-9-_./]+$"
        },
        "pullPolicy": {
          "type": "string",
          "pattern": "^(Always|Never|IfNotPresent)$"
        }
      }
    }
  }
}
```

Jetzt noch kurz Testen:

Test 1:
```yaml
# values.yaml
image:
  repository: true
  pullPolicy: enforced
```

```console
$ helm lint .
==> Linting .
[ERROR] values.yaml: - image.repository: Invalid type. Expected: string, given: boolean
- image.pullPolicy: Does not match pattern '^(Always|Never|IfNotPresent)$'
```

Test 2: 

```yaml
# values.yaml
image:
  repository: "ghcr.io/zahlenhelfer/container-nginx-demo"
  pullPolicy: IfNotPresent
```

```console
$ helm lint .
==> Linting .
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

## Fazit:
- Frühzeitige Fehlererkennung
- Bessere Dokumentation der erwarteten Werte
- Einfachere Nutzung für Dev-Teams

JSON Schema bringt Struktur und Sicherheit in die Welt der Helm-Charts. Wer produktionsreife Charts pflegt, sollte die Validierung unbedingt nutzen.