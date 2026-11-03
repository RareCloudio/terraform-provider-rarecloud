---
page_title: "rarecloud_network Resource - terraform-provider-rarecloud"
description: |-
  A private network (VPC). The CIDR is allocated by the API (`10.N.0.0/16`).
---

# rarecloud_network (Resource)

A private network (VPC). The CIDR is allocated by the API (`10.N.0.0/16`).

Changing `name` recreates the network. Launch VMs into it with `rarecloud_server.vpc_id`.

## Example Usage

```terraform
resource "rarecloud_network" "backend" {
  name = "backend"
}
```

## Import

```shell
terraform import rarecloud_network.backend <network id>
```

The import reads every attribute: `name`, `cidr`, `is_default`, `api_access`.

Nothing is left out: a config matching the real object plans no change after the import.

## Schema

### Required

- `name` (String) Network name. A change recreates the network.

### Read-Only

- `id` (String)
- `cidr` (String)
- `is_default` (Boolean)
- `api_access` (String)
