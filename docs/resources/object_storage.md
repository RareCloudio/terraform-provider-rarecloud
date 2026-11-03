---
page_title: "rarecloud_object_storage Resource - terraform-provider-rarecloud"
description: |-
  Enables Object Storage for the account.
---

# rarecloud_object_storage (Resource)

Enables Object Storage for the account (`POST /v1/object-storage`). One per customer, with nothing to configure.

The namespace handle that starts every bucket name is chosen on the account's first `rarecloud_object_storage_bucket`, not here. Creating a bucket also enables Object Storage by itself, so this resource is optional. It is useful for making the account an explicit dependency and for reading the price card and limits.

## Destroying disables Object Storage

Destroying this resource DISABLES Object Storage (`DELETE /v1/object-storage`). The API refuses that while any bucket or active access key exists, and the apply fails with its message:

```
Delete your 2 bucket(s) and revoke your 1 access key(s) first.
```

Give your buckets and keys `depends_on = [rarecloud_object_storage.this]` so Terraform destroys them before the account. A bucket that still holds objects also needs `force_destroy = true` (see `rarecloud_object_storage_bucket`).

When the last bucket is deleted, the account becomes `dormant` (paused: keys are denied and nothing accrues) until a bucket is created again. That is a status change, not a deletion, and does not affect this resource.

## Example Usage

```terraform
resource "rarecloud_object_storage" "this" {}

output "object_storage_status" {
  value = rarecloud_object_storage.this.status
}
```

## Import

```shell
terraform import rarecloud_object_storage.this <account id>
```

## Schema

### Read-Only

- `id` (String) The Object Storage account id.
- `handle` (String) The namespace prefix of every bucket name (`<handle>-<name>`). Null until the first bucket chooses it, and always null for an account migrated from an existing one.
- `status` (String) `provisioning`, `active`, `dormant`, `suspended`, `deleting` or `failed`.
- `plan` (String) `payg` (billed from the price card) or `committed` (a flat monthly fee for reserved capacity).
- `quota_tb` (Number) Storage quota in TB.
- `max_buckets` (Number) The most buckets this account may have.
- `max_keys` (Number) The most access keys this account may have.
- `regions` (Attributes List) The regions buckets can be created in: `id`, `label`, `endpoint` (S3 endpoint URL), `country` (ISO-2), `eu` (Bool; the UK regions are not in the EU).
- `pricing` (Attributes) The price card the account is billed from, in cents of `currency`: `base_monthly_cents`, `included_gb`, `gb_monthly_cents`, `egress_cents_per_gb`, `cdn_bucket_monthly_cents`, `cdn_traffic_cents_per_gb`, `cdn_cents_per_million_requests`, `cdn_included_requests`, `currency`.
- `created_at` (String) ISO-8601 creation timestamp.
