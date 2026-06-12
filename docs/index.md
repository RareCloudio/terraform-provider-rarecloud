---
page_title: "RareCloud Provider"
description: "Manage RareCloud cloud servers and account resources with Terraform."
---

# RareCloud Provider

Manage [RareCloud](https://rarecloud.io) resources: cloud servers, SSH keys,
catalog lookups. The provider wraps the public REST API at
`https://api.rarecloud.io/v1/` and authenticates with a scoped API token.

## Example Usage

```hcl
provider "rarecloud" {
  token = var.rarecloud_token # or env RARECLOUD_TOKEN
}

data "rarecloud_catalog_plan" "small" {
  sku = "s-1vcpu-2gb"
}

resource "rarecloud_server" "web" {
  name   = "web-01"
  plan   = data.rarecloud_catalog_plan.small.sku
  region = "ro-buc"
  image  = "ubuntu-24.04"
}
```

## Schema

- `token` (String, Sensitive) API token. May also be set via `RARECLOUD_TOKEN`.
- `endpoint` (String) Base URL of the RareCloud API. Defaults to `https://api.rarecloud.io`.
