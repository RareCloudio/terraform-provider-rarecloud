# Terraform Provider: RareCloud

Official Terraform provider for [RareCloud](https://rarecloud.io): manage
cloud servers, SSH keys and account resources on the AI-first European cloud.

Wraps the public REST API at https://api.rarecloud.io/v1/.

> **Status: first release coming soon.** Until a release appears here and the
> provider is published to the Terraform Registry, the configuration below
> will not resolve.

## Usage

```hcl
terraform {
  required_providers {
    rarecloud = {
      source  = "RareCloudio/rarecloud"
      version = ">= 0.1.0"
    }
  }
}

provider "rarecloud" {
  # Or set RARECLOUD_TOKEN in the environment.
  token = var.rarecloud_token
}

resource "rarecloud_server" "web" {
  name   = "web-01"
  plan   = "s-1vcpu-2gb"
  region = "ro-buc"
  image  = "ubuntu-24.04"
}
```

Create API tokens in the console: https://console.rarecloud.io/account
(API tokens tab). Use a token scoped to what your Terraform runs need.

## About this repo

This repo hosts the official binary releases that the Terraform Registry
ingests. Issues and feature requests are welcome here.
