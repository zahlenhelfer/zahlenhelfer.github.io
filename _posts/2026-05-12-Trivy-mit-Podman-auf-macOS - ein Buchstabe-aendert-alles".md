---
layout: post
title: "Tip: Trivy mit Podman auf macOS - ein Buchstabe ändert alles"
category: kubernetes
tags:
  - blog
  - de
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---
## Das Problem: Pod ist installiert und Trivy läuft nicht mehr
Du hast Docker Desktop deinstalliert, Podman installiert, einen Symlink `alias docker=podman` gesetzt - und alles scheint zu funktionieren. Dann startest Du Trivy zur Schwachstellenanalyse:
```shell
$ trivy image mein-image:1.0
error: docker-credential-desktop: executable file not found in $PATH
```
Trivy sucht nach dem `docker-credential-desktop` - dem Credential Helper von Docker Desktop. Diesen findet es natürlich nicht mehr. Den gibt es nach der Deinstallation schlicht nicht mehr. Ja, ein 'brew' Paket gibt es zwar, aber der da kein Docker Desktop, brauchen wir den credential-helper auch nicht mehr.
## Die Ursache: ~/.docker/config.json
Podman nutzt eine eigene Authentifizierungsdatei. Wenn diese nicht vorhanden ist, fällt Podman auf die Standard-Docker-Konfiguration zurück:
  `~/.docker/config.json` und Tools wie Trivy verhalten sich genauso.

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
Die Lösung ist überraschend simpel. Öffne `~/.docker/config.json` und ändere den Schlüsselnamen von `credsStore` zu `credStore`:
```json
  {
    "credStore": "desktop"
  }
```
Da `credStore` (ohne s) kein gültiger Schlüssel in der Docker-Konfiguration ist, wird der Eintrag einfach ignoriert. Trivy versucht damit nicht mehr, den fehlenden Credential Helper aufzurufen.
## Fazit
Der Wechsel von Docker zu Podman auf macOS geht meist glatt - bis man auf versteckte Abhängigkeiten stößt. Trivy liest still und leise die Docker-Konfiguration mit, auch wenn man Docker längst hinter sich gelassen hat. Ein einzelner Buchstabe löst das Problem.