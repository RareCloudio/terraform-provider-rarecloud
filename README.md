# Terraform Provider: RareCloud

Official Terraform provider for [RareCloud](https://rarecloud.io). It wraps the public API at https://api.rarecloud.io/v1/.

Documentation: https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs

```hcl
terraform {
  required_providers {
    rarecloud = {
      source  = "RareCloudio/rarecloud"
      version = "~> 0.5"
    }
  }
}
```

Create an API token at https://console.rarecloud.io/account/api. This repo hosts the releases the Terraform Registry ingests. Issues and feature requests are welcome here.
