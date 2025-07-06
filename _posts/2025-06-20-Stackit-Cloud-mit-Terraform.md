---
layout: post
title: STACKIT Cloud mit Terraform nutzen
category: terraform
tags:
  - blog
  - de
  - terraform
  - stackit
  - opentofu
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---

## Die STACKIT-Cloud einfach per Terraform nutzen

Die letzten Monate ist einiges in Schwanken gerade und man hört immer wieder Deutsche bzw. Europäische Cloud-Anbieter müssen her. Und jetzt schaue ich mir dazu mal die [STACKIT](https://www.stackit.de)-Cloud der SchwarzIT an. Aber das ganze natürlich per Terraform bzw. OpenToFu skriptbar bzw. idempotent.
## Ziel:
- Setup von Terraform mit STACKIT
-  Erstellen und Nutzen eines Webservers mit NGINX

>Wichtig: ich nutze OpenToFu, verwendest Du Terraform ist das kein Problem, einfach `alias tofu=terraform` und es sollte auch mit deinem `terraform` Binary funktionieren

1. Projekt erstellen (manuell)
2. Service-Account erstellen (manuell)
3. Bespiel: STACKIT Compute-Engine

Zum Start erstellt Ihr Euch ein leeres Verzeichnis mit den wichtigsten Dateien:
```console
├── main.tf
├── outputs.tf
├── provider.tf
├── variables.tf
└── vm.tf
```

### main.tf
```hcl
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    stackit = {
      source  = "stackitcloud/stackit"
      version = "0.55.0"
    }
  }
}

provider stackit{

}
```
In der `main.tf` wird festgelegt welche Terraform-Provider bzw. Module genutzt werden. Hier starten wir einfach mir dem aktuellen Provider also quasi Treiber für die STACKIT-Cloud. Dieser wird noch aktiv entwickelt, daher kann die aktuelle Version abweichen.
