---
page_title: "rarecloud_firewall Resource - terraform-provider-rarecloud"
description: |-
  A stateful firewall (security group).
---

# rarecloud_firewall (Resource)

A stateful firewall (security group).

New firewalls get the API's default rules: inbound ICMP and TCP 22, 80 and 443 (IPv4); outbound every TCP port except 25, plus UDP, ICMP, GRE, ESP, AH, SCTP, VRRP and IP-in-IP (IPv4 and IPv6) and protocol 41 over IPv6. 6in4 tunnels (protocol 41 over IPv4) cannot be allowed by security groups; ask support on a ticket. A new cloud VM can have outbound TCP 25 blocked until an unblock request is approved; see the `rarecloud_server_smtp` data source. On such a VM, a rule covering TCP 25 (including `all` or a wider range) on an attached firewall answers 409 `SMTP_EGRESS_NOT_APPROVED`. So does attaching a firewall that already opens TCP 25, and creating a VM while the Private network firewall opens it. Leave `inbound_rule` / `outbound_rule` unset to keep whatever the firewall has; set them to manage that direction exactly. Rules are matched by content, not order. Changing `name` recreates the firewall.

## Example Usage

```terraform
resource "rarecloud_firewall" "web" {
  name = "web"

  inbound_rule = [
    { protocol = "tcp", port_range_min = 22, port_range_max = 22, remote_cidr = "0.0.0.0/0" },
    { protocol = "tcp", port_range_min = 443, port_range_max = 443, remote_cidr = "0.0.0.0/0" },
  ]

  attached_server_ids = [rarecloud_server.web.id]
}
```

## Import

```shell
terraform import rarecloud_firewall.web <firewall id>
```

The import reads every attribute: `name`, `inbound_rule`, `outbound_rule`, `attached_server_ids`, `is_default`, `is_private_network`. Declared rules are matched to the imported ones by content, so a config listing the same rules plans no change.

Nothing is left out: a config matching the real object plans no change after the import.

## Schema

### Required

- `name` (String) Display name. A change recreates the firewall.

### Optional

- `inbound_rule` (Set of Attributes) Inbound rules: `protocol` (`tcp`, `udp`, `icmp` or `all`), `port_range_min`, `port_range_max` (omit for ICMP or all), `remote_cidr` (unrestricted when omitted), `description`, and the read-only rule `id`. Unset, the rules are read but not managed.
- `outbound_rule` (Set of Attributes) Outbound rules, same shape.
- `attached_server_ids` (Set of String) VMs to attach the firewall to. Unset, attachments made elsewhere are read and never detached.

### Read-Only

- `id` (String)
- `is_default` (Boolean) True for the account's default firewall, which cannot be deleted.
- `is_private_network` (Boolean)
