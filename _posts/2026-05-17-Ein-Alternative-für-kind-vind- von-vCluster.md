---
layout: post
title: "Tip: vind - Die Alternative für kind?"
category: kubernetes
tags:
  - blog
  - de
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---

## Install vcluster
```shell
brew install loft-sh/tap/vcluster
```

## Set Docker as the driver
```perl
vcluster use driver docker
```

## Erster Cluster
```shell
vcluster create my-cluster
```


# Eine bestimmte Version
```shell
vcluster create old-cluster \
  --set controlPlane.distro.k8s.version=v1.32.0
```




## Das Problem: Einen lokalen Kubernetes-Cluster, aber mit Persistenz.
Du hast den Docker Desktop deinstalliert, Podman installiert, einen Alias  gesetzt (`alias docker=podman`) - und alles scheint zu funktionieren. Dann startest Du Trivy zur Schwachstellenanalyse:
```shell
$ trivy image mein-image:1.0
...
error: docker-credential-desktop: executable file not found in $PATH
...
```
Trivy sucht nach dem `docker-credential-desktop` - dem Credential Helper von Docker Desktop. Diesen findet es natürlich nicht mehr. Nach der Deinstallation gibt es ihn schlichtweg nicht mehr. Ja, ein `brew` Paket gibt es zwar, aber da kein Docker Desktop vorhanden ist, brauchen wir den credential-helper auch nicht nachzuinstallieren.
## Die Ursache: ~/.docker/config.json
Podman nutzt eine eigene Authentifizierungsdatei. Wenn diese nicht vorhanden ist, greift Podman auf die Standard-Dockerkonfiguration zurück, nämlich auf `~/.docker/config.json`. Tools wie Trivy verhalten sich allerdings genauso.
Ein Blick in die Datei verrät das Problem sofort:
```json
...
  {
    "credsStore": "desktop"
  },
...
```
Dieser Eintrag wurde von Docker Desktop angelegt und verweist auf den
docker-credential-desktop-Helper. Der ist weg - der Eintrag aber nicht.
## Die Lösung: Ein Buchstabe macht den Unterschied
Die Lösung ist überraschend simpel. Öffne die Datei `~/.docker/config.json` und ändere den Schlüsselnamen von `credsStore` zu `credStore`.
```json
...
  {
    "credStore": "desktop"
  }
...
```
Da `credStore` (ohne s) in der Docker-Konfiguration keinen gültigen Schlüssel enthält, wird der Eintrag einfach ignoriert. Trivy versucht damit nicht mehr, den fehlenden Credential Helper aufzurufen.
## Fazit
Der Wechsel von Docker zu Podman auf macOS verläuft meist reibungslos – bis man auf versteckte Abhängigkeiten stößt. Trivy liest die Docker-Konfiguration still und leise mit, auch wenn man Docker längst hinter sich gelassen hat. Ein einzelner Buchstabe löst das Problem.