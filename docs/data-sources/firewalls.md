---
page_title: "rarecloud_firewalls Data Source - terraform-provider-rarecloud"
description: |-
  Lists every firewall the caller can see.
---

# rarecloud_firewalls (Data Source)

Lists every firewall the caller can see (`GET /v1/firewalls`): the default firewall first, then the Private network firewall, then the rest by name. Takes no input; filter with a `for` expression.

The list carries rule counts, not rules. For one firewall's rules use the [`rarecloud_firewall`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/data-sources/firewall) data source.

## Example Usage

```terraform
data "rarecloud_firewalls" "all" {}

output "private_network_firewall_id" {
  value = one([for f in data.rarecloud_firewalls.all.firewalls : f.id if f.is_private_network])
}
```

## Schema

### Read-Only

- `firewalls` (Attributes List) One entry per firewall. (see [below for nested schema](#nestedatt--firewalls))

<a id="nestedatt--firewalls"></a>
### Nested Schema for `firewalls`

Read-Only:

- `id` (String) Firewall id.
- `name` (String) Firewall name.
- `is_default` (Boolean) Whether this is the account's default firewall.
- `is_private_network` (Boolean) Whether this is the Private network firewall RareCloud manages on the VMs' private interface (it cannot be attached, detached or deleted).
- `inbound_rule_count` (Number) Number of inbound rules. Null when not reported.
- `outbound_rule_count` (Number) Number of outbound rules. Null when not reported.
- `attached_server_ids` (List of String) Ids of the servers the firewall is attached to. Null when not reported.
