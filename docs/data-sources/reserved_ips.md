---
page_title: "rarecloud_reserved_ips Data Source - terraform-provider-rarecloud"
description: |-
  Lists every reserved IP the caller holds.
---

# rarecloud_reserved_ips (Data Source)

Lists every reserved IP the caller holds (`GET /v1/reserved-ips`), attached or not, oldest first. Takes no input; filter with a `for` expression. The internal floating IP id is not exposed.

## Example Usage

```terraform
data "rarecloud_reserved_ips" "all" {}

output "free_reserved_ips" {
  value = [for r in data.rarecloud_reserved_ips.all.reserved_ips : r.ip if !r.attached]
}
```

## Schema

### Read-Only

- `reserved_ips` (Attributes List) One entry per reserved IP. (see [below for nested schema](#nestedatt--reserved_ips))

<a id="nestedatt--reserved_ips"></a>
### Nested Schema for `reserved_ips`

Read-Only:

- `id` (String) Reserved IP id (the value `rarecloud_reserved_ip.id` holds).
- `ip` (String) The IPv4 address.
- `region` (String) Region slug. Null when not reported.
- `attached` (Boolean) Whether the address is attached to a server right now.
- `attached_server_id` (String) Id of the server it is attached to. Null when unattached.
- `monthly_cents` (Number) Monthly price of the reservation in cents, as the API reports it. Null when not reported.
- `accrued_cents` (Number) What the reservation has accrued since it was reserved, in cents, as the API reports it (the monthly price pro-rated by the hour). A display figure, not an invoice. Null when not reported.
- `created_at` (String) When the address was reserved (RFC 3339).
