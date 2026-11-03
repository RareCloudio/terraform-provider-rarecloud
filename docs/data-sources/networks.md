---
page_title: "rarecloud_networks Data Source - terraform-provider-rarecloud"
description: |-
  Lists every private network (VPC) the caller owns.
---

# rarecloud_networks (Data Source)

Lists every private network (VPC) the caller owns (`GET /v1/networks`), the default one included. Takes no input; filter with a `for` expression.

## Example Usage

```terraform
data "rarecloud_networks" "all" {}

output "default_network_id" {
  value = one([for n in data.rarecloud_networks.all.networks : n.id if n.is_default])
}
```

## Schema

### Read-Only

- `networks` (Attributes List) One entry per network. (see [below for nested schema](#nestedatt--networks))

<a id="nestedatt--networks"></a>
### Nested Schema for `networks`

Read-Only:

- `id` (String) Network id (the value `rarecloud_network.id` and `rarecloud_server.vpc_id` hold).
- `name` (String) Network name.
- `cidr` (String) The network's IPv4 range, for example `10.1.0.0/16`. Null when not reported.
- `is_default` (Boolean) Whether this is the account's default network.
