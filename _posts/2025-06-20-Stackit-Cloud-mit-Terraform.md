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

Deutsche Cloud-Anbieter, ja, nein, keine Zeit....Ausreden zählen nicht mehr.

## Ziel:
- Setup von Terraform mit STACKIT
- Erstellen eines Webservers mit NGINX


>Wichtig: ich nutze OpenToFu, verwendest Du Terraform ist das kein Problem, einfach `alias tofu=terraform` und es sollte auch mit deinem `terraform` Binary funktionieren

1. Projekt erstellen (manuell)
2. Service-Account erstellen (manuell)
3. Bespiel: STACKIT Compute-Engine
4. Bespiel: STACKIT DNS-Zone

Verzeichnisstruktur
provider.tf
main.tf
vm.tf
dns.tf
output.tf
projekt.auto.tfvars