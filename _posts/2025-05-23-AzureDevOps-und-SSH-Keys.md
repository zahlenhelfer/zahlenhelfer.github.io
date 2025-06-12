---
layout: post
title: AzureDevOps und der SSH-Key - es kann nur einen geben!
category: azuredevops
tags:
  - blog
  - tip
  - de
  - ssh
  - azuredevops
permalink: /:year/:month/:day/:title:output_ext
published: true
---

## Das Problem: `remote: Public key authentication failed.`

Wer in AzureDevOps SSH-Keys z.B. für Repositories nutzen möchte, könnte über folgenden Meldung stolpern:
```
remote: Public key authentication failed.
fatal: Could not read from remote repository.
```

Das Spannende ist dabei die Konstellation wann der Fehler auftritt. Auf meinem Arbeitsrechner bekomme ich die Fehlermeldung nicht, auf meinem HomeOffice-Rechner schon.

Und ja, einige sagen  "ist doch klar" - hast Du die `.ssh/config` gepflegt. Ja das habe ich. - Wahrscheinlich ist dein Schlüssel falsch. Nein ist er nicht.
Beispieleintrag aus der `.ssh/config`:
```
Host ssh.dev.azure.com
  HostName ssh.dev.azure.com
  IdentityFile ~/.ssh/key_for_work
```

Versuchen wir also mal per `ssh -vvv -T ssh.dev.azure.com` herauszufinden was los ist.

Ist es der Algorythmus? Nein, obwohl AzureDevOps da schon etwas pingelig ist: `RSA-SHA2-256` oder `RSA-SHA2-512`sind [supported](https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops#set-up-ssh-key-authentication)

## Die Lösung: `IdentitiesOnly yes` bzw.  RTFM
Die FAQ von Microsoft brachte doch tatsächlich die Lösung. [Hier](https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops#q-how-can-i-use-a-nondefault-key-location-that-is-not-sshid_rsa-and-sshid_rsapub) stand geschrieben, das nur 1 Schlüssel angeboten werden darf und auch nur der erste geprüft wird.  Der wichtige Teil für meine `.sshconfig` war dann die letzte Zeile:
```
Host ssh.dev.azure.com
  IdentityFile ~/.ssh/id_rsa_azure
  IdentitiesOnly yes
```

Die Einstellung `IdentitiesOnly yes` stellt sicher, dass SSH keine andere verfügbare Identität zur Authentifizierung verwendet. Diese Einstellung ist besonders wichtig, wenn mehr als eine Identität verfügbar ist.
## Fazit:
- `ssh -vvv bzw. -T kann` helfen.
- unterstützten Cypher überprüfen
- SSH-Config pflegen
- notfalls einmal das [FAQ](https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops#set-up-ssh-key-authentication) von Microsoft befragen

Diesmal war es kein Tool-Tip, sondern eher was aus dem IT-Alltag. Ich hoffe es hat trotzdem etwas geholfen.