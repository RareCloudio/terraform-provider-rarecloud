---
page_title: "RareCloud Provider"
description: |-
  Manage RareCloud cloud infrastructure with Terraform.
---

# RareCloud Provider

Manage [RareCloud](https://rarecloud.io) resources: cloud VMs, volumes, private networks, firewalls, load balancers, reserved IPs, reverse DNS, snapshots, managed Kubernetes, AI sandboxes, Object Storage, the container registry, domains and the billing alert. The provider wraps the public API at `https://api.rarecloud.io/v1/`.

Requires Terraform 1.0 or later. The ephemeral `rarecloud_kubernetes_kubeconfig` requires Terraform 1.10 or later.

## Example Usage

```terraform
terraform {
  required_providers {
    rarecloud = {
      source  = "RareCloudio/rarecloud"
      version = "~> 0.5"
    }
  }
}

provider "rarecloud" {
  # Or set RARECLOUD_TOKEN in the environment.
  token = var.rarecloud_token
}

variable "rarecloud_token" {
  type      = string
  sensitive = true
}

resource "rarecloud_server" "web" {
  name   = "web-01"
  plan   = "g-2vcpu-8gb"
  region = "bucharest-ro"
  image  = "ubuntu-24.04"
}
```

## Authentication

Create an API token at https://console.rarecloud.io/account/api, with the scopes your configuration needs. Set it with the `token` attribute or the `RARECLOUD_TOKEN` environment variable. The attribute wins when both are set. The provider fails without a token, even for public catalog data sources.

## Secrets in state

`rarecloud_registry_credential`, `rarecloud_registry_kubernetes_manifest`, `rarecloud_object_storage_key` and `rarecloud_kubernetes_kubeconfig` keep secrets in Terraform state in plain text. `Sensitive` only hides them from output. Use an encrypted remote backend and restrict access to state.

## Resources made read-only in the console

A resource can be made read-only for API tokens in the console. `rarecloud_server`, `rarecloud_volume`, `rarecloud_network`, `rarecloud_load_balancer` and `rarecloud_domain` report this in `api_access`. While it is `read_only`, an apply that would change or destroy the resource fails with `RESOURCE_PROTECTED`. The switch can only be changed in the console.

## Safe retries on create

Each create sends one `Idempotency-Key`. When the API's answer is lost, the provider retries up to 3 times with the same key, and the API runs the create once. A one-time secret comes only in the first answer: if that answer was lost, the resource is saved without it and a warning tells you to replace it.

## Schema

### Optional

- `token` (String, Sensitive) API token (`rc_pat_...`). May also be set with `RARECLOUD_TOKEN`.
- `api_endpoint` (String) Base URL of the API, without `/v1`. Default `https://api.rarecloud.io`. May also be set with `RARECLOUD_API`.
