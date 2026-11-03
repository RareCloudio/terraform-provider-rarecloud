---
page_title: "rarecloud_registry Resource - terraform-provider-rarecloud"
description: |-
  Enables a customer's private container registry account.
---

# rarecloud_registry (Resource)

Enables a customer's private container registry account (`POST /v1/registry`). One registry account per customer.

Destroying this resource CLOSES the account (`DELETE /v1/registry?confirm=<handle>`): images are not deleted immediately, the account stays reopenable with the SAME handle until its grace window (7 days by default) elapses, after which stored images are deleted for good.

## Example Usage

```terraform
resource "rarecloud_registry" "acme" {
  handle = "acme"
  tier   = "starter"
}

output "registry_hostname" {
  value = rarecloud_registry.acme.hostname
}
```

## Schema

### Required

- `handle` (String) Customer-chosen, immutable registry handle: 3-30 lowercase letters/digits. Becomes the image path prefix `<hostname>/<handle>/<repo>`. Some words are reserved. Changing it recreates the resource.
- `tier` (String) Billing tier: `free`, `starter`, `premium`, or `enterprise`. Changing it calls `PATCH /v1/registry` in place. An upgrade (new tier's quota at or above the current one) is always allowed; a downgrade is refused (409, `tier_below_usage`) if the account's last measured usage exceeds the new tier's quota. See the [`rarecloud_registry_tiers`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/data-sources/registry_tiers) data source for each tier's quota and price.

### Read-Only

- `id` (String) Same value as `handle`. The account has no separate id: it is addressed implicitly by the caller's token.
- `hostname` (String) The registry hostname images are pushed to and pulled from, e.g. `registry.rarestack.net`.
- `status` (String) `active`, `suspended` (pushes blocked, pulls still work, for non-payment or abuse), or `closed` (being torn down).
- `created_at` (String) ISO-8601 creation timestamp.

## Import

Import by handle:

```shell
terraform import rarecloud_registry.acme acme
```
