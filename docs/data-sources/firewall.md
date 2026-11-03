---
page_title: "rarecloud_firewall Data Source - terraform-provider-rarecloud"
description: |-
  Reads one firewall by id, by name, or the account's Private network firewall.
---

# rarecloud_firewall (Data Source)

Reads one firewall (`GET /v1/firewalls/{id}`): by `id`, by exact `name`, or with `private_network = true` for the account's **Private network** firewall. Set exactly one of the three.

The Private network firewall is created by RareCloud and applied to the private interface of your VMs. Attach, detach and delete of it answer `409 CONFLICT`, through Terraform as through any other client. A customer firewall may also be named "Private network", so select the managed one with `private_network = true`.

## Example Usage

```terraform
data "rarecloud_firewall" "private" {
  private_network = true
}

output "private_network_firewall_id" {
  value = data.rarecloud_firewall.private.id
}
```

## Schema

### Optional

- `id` (String) Select the firewall by id. Always set on read.
- `name` (String) Select the firewall by exact name; an error when no firewall or more than one has it. Always set on read.
- `private_network` (Boolean) Set to `true` to select the account's Private network firewall.

### Read-Only

- `is_default` (Boolean) True for the account's default firewall.
- `is_private_network` (Boolean) True for the account's Private network firewall.
- `inbound_rule` (Set of Attributes) Inbound rules: `id`, `protocol`, `port_range_min`, `port_range_max`, `remote_cidr`, `remote_firewall_id`, `description`. A rule names a range (`remote_cidr`) or a firewall whose members it admits (`remote_firewall_id`); the Private network firewall's managed rule names the firewall itself (your VMs), and each load balancer has one rule from the subnet its VIP sits on.
- `outbound_rule` (Set of Attributes) Outbound rules, same fields.
- `attached_server_ids` (Set of String) Ids of the VMs that use this firewall (for the Private network firewall: on their private interface).
