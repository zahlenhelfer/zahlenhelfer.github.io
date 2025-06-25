---
layout: post
title: Eine Projektfactory mit Terraform für die STACKIT Cloud erstellen
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

## Problem: 
Das was Du bei AWS mit Accounts bzw. bei Azure mit der Subscriptions in Kombination mit Ressource-Groups trennen würdest sucht nun ein ähnliches Konstrukt bei STACKIT. Aber natürlich das ganze auch noch per Terraform?

## Lösung: Eine Projektfactory erstellen


## Organisation, Projekte und Service-Accounts
In deinem Cloud-Account gibt es normaler

https://registry.terraform.io/providers/stackitcloud/stackit/latest/docs/resources/resourcemanager_project

Terraform-

```hcl
resource "stackit_resourcemanager_project" "example" {
  parent_container_id = "example-parent-container-abc123"
  name                = "example-container"
  labels = {
    "Label 1" = "foo"
    // "networkArea" = stackit_network_area.foo.network_area_id
  }
  owner_email = "john.doe@stackit.cloud"
}
```