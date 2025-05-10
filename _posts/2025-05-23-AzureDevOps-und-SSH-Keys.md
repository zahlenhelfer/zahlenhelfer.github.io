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
published: false
---

## Das Problem: `remote: Public key authentication failed.`

Wer in AzureDevOps SSH-Keys z.B. für Repositories nutzen möchte, könnte über folgenden Meldung stolpern:
```
remote: Public key authentication failed.
fatal: Could not read from remote repository.
```

Das Spannede ist die Konstellation. Auf meinem Arbeitsrechner bekomme ich die Fehlermeldung nicht, auf meinem HomeOffice-Rechner schon.

Und ja, einige sagen  "ist doch klar" - hast Du die `.ssh/config` gepflegt. Ja das habe ich. wahrscheinlich dein Schlüssel versuch mal per ssh -vvv herauszufinden was los ist.

Beispieleintrag aus der `.ssh/config`:
```
Host ssh.dev.azure.com
  HostName ssh.dev.azure.com
  IdentityFile ~/.ssh/key_for_work
```

https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops#set-up-ssh-key-authentication
To generate key files using the RSA algorithm supported by Azure DevOps (either RSA-SHA2-256 or RSA-SHA2-512), run one of the following commands from a PowerShell or another shell such as `bash` on your client

https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops#q-i-have-multiple-ssh-keys-how-do-i-use-the-correct-ssh-key-for-azure-devops

`ssh-keygen -t rsa-sha2-512`

```
ssh -T git@ssh.dev.azure.com
```

### Q: How can I use a nondefault key location, that is, not ~/.ssh/id_rsa and ~/.ssh/id_rsa.pub?

**A:** To use a key stored in a different place than the default, perform these two tasks:

1. The keys must be in a folder that only you can read or edit. If the folder has wider permissions, SSH doesn't use the keys.
    
2. You must let SSH know the location of the key, for example, by specifying it as an "Identity" in the SSH config:
    
    Copy
    
    ```
    Host ssh.dev.azure.com
      IdentityFile ~/.ssh/id_rsa_azure
      IdentitiesOnly yes
    ```
    

The `IdentitiesOnly yes` setting ensures that SSH doesn't use any other available identity to authenticate. This setting is particular important if more than one identity is available.

[](https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops#q-i-have-multiple-ssh-keys-how-do-i-use-the-correct-ssh-key-for-azure-devops)

### Q: I have multiple SSH keys. How do I use the correct SSH key for Azure DevOps?

**A:** Generally, when you configure multiple keys for an SSH client, the client attempts to authenticate with each key sequentially until the SSH server accepts one.

However, this approach doesn't work with Azure DevOps due to technical constraints related to the SSH protocol and the structure of our Git SSH URLs. Azure DevOps accepts the first key provided by the client during authentication. If this key is invalid for the requested repository, the request fails without attempting any other available keys, resulting in the following error:

Copy

```
remote: Public key authentication failed.
fatal: Could not read from remote repository.
```

For Azure DevOps, you need to configure SSH to explicitly use a specific key file. The procedure is the same as when using a key stored in a [nondefault location](https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops#non-default-keys). Tell SSH to use the correct SSH key for the Azure DevOps host.

[](https://learn.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops#q-how-do-i-use-different-ssh-keys-for-different-organizations-on-azure-devops)
```

IdentitiesOnly yes

```
## Fazit:
- ssh -vvv kann helfen.
- unterstützten Cypher überprüfen
- SSH-Config pflegen

Diesmal war es kein ToolTip sondern eher was für den Alltag. Ich hoffe es hat trotzdem etwas geholfen.