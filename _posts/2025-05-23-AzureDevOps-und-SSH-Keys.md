---
layout: post
title: AzuredevOps und der SSH-Key - es kann nur einen geben!
category: azuredevops
tags:
  - blog
  - tip
  - de
  - ssh
  - azuredevops
permalink: /:year/:month/:day/:title:output_ext
published: false
---

## Das Problem: `remote: Public key authentication failed.`

Wer in AzureDevOps SSH-Keys z.B. für Repositories nutzen möchte, könnte hier stolpern.

```
remote: Public key authentication failed.
fatal: Could not read from remote repository.
```


https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops#q-i-have-multiple-ssh-keys-how-do-i-use-the-correct-ssh-key-for-azure-devops

`ssh-keygen -t rsa-sha2-512`

```

## Die Lösung: validieren – bevor deployed wird!
Hier kommt [JSON Schema](https://json-schema.org/) ins Spiel. Seit [Helm 3.5](https://helm.sh/docs/faq/changes_since_helm2/#validating-chart-values-with-jsonschema) ist es möglich eine `values.schema.json` im Chart-Verzeichnis zu hinterlegen. Diese beschreibt die erwartete Struktur. Jetzt kann Helm, die `values.yaml`-Datei beim Rendern validieren – das passiert automatisch bei:

## Ein Beispiel: `image.repository` und `image.pullpolicy`
Beim `image.repository`-Wert soll per RegEx geprüft werden, ob dieser ein Valides Docker-Image ist. Die `image.pullpolicy` kennt nur drei Werte, damit wäre alles anderen falsch.

```yaml
image.repository # Docker-Image Bspl.: "acmeorg/nginx"
image.pullPolicy # Werte: [IfNotPresent, Always, Never]
```

Mit einer einfachen `values.schema.json` bekommt Helm das ganze mit und kann reagieren.

Beispiel der values.schema.json:
```
Host ssh.dev.azure.com
  HostName ssh.dev.azure.com
  IdentityFile ~/.ssh/private_key_for_fabrikam
  IdentitiesOnly yes
```

## Fazit:
- ssh -vvv kann helfen.
- unterstützten Cypher überprüfen
- SSH-Config pflegen

Diesmal war es kein ToolTip sondern eher was für den Alltag. Ich hoffe es hat trotzdem etwas geholfen.