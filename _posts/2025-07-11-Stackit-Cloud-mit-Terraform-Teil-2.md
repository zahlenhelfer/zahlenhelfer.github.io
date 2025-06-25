---
layout: post
title: STACKIT Cloud Teil 2 - die Ressource RabbitMQ
category: terraform
tags:
  - blog
  - de
  - terraform
  - stackit
  - opentofu
  - rabbitmq
permalink: /:year/:month/:day/:title:output_ext
published: false
render_with_liquid: "false"
---

## Die STACKIT-Cloud einfach per Terraform nutzen

Die letzten Monate ist einiges in Schwanken gerade und man hört immer wieder Deutsche bzw. Europäische Cloud-Anbieter müssen her. Und jetzt schaue ich mir dazu mal die STACKIT-Cloud der SchwarzIT an. Aber das ganze natürlich per Terraform bzw. OpenToFu scriptbar.
## Ziel:
- Erstellen und Nutzen des RabbitMQ-Service von STACKIT

>Wichtig: ich nutze OpenToFu, verwendest Du Terraform ist das kein Problem, einfach `alias tofu=terraform` und es sollte auch mit deinem `terraform` Binary funktionieren

1. Projekt erstellen (manuell)
2. Service-Account erstellen (manuell)
3. Bespiel: STACKIT Compute-Engine

Verzeichnisstruktur
provider.tf
main.tf
vm.tf
dns.tf
output.tf
projekt.auto.tfvars

GitHub-Repo