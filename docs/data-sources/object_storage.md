---
page_title: "rarecloud_object_storage Data Source - terraform-provider-rarecloud"
description: |-
  The account's Object Storage status.
---

# rarecloud_object_storage (Data Source)

The account's Object Storage status (`GET /v1/object-storage`). When Object Storage is not enabled, `enabled` is `false` and every other attribute is null. This is not an error.

## Example Usage

```terraform
data "rarecloud_object_storage" "current" {}

output "namespace" {
  value = data.rarecloud_object_storage.current.handle
}
```

## Schema

### Read-Only

- `enabled` (Boolean) Whether Object Storage is enabled.
- `id` (String) The Object Storage account id.
- `handle` (String) The namespace prefix of every bucket name. Null until the first bucket chooses it.
- `status` (String) `provisioning`, `active`, `dormant`, `suspended`, `deleting` or `failed`.
- `plan` (String) `payg` or `committed`.
- `quota_tb` (Number) Storage quota in TB.
- `reserved_tb` (Number) Reserved capacity on a `committed` plan, else null.
- `committed_monthly_cents` (Number) The flat monthly fee on a `committed` plan, else null.
- `current_month_accrued_cents` (Number) This month's accrual so far, in cents. Null when billing could not be read, which is not the same as zero.
- `active_bytes`, `deleted_bytes`, `egress_bytes_mtd` (Number) Usage at the last measurement. Null when not measured yet.
- `usage_measured_at` (String) When usage was last measured.
- `failure_reason` (String) Why provisioning failed, when `status` is `failed`.
- `max_buckets`, `max_keys` (Number) Account limits.
- `regions` (Attributes List) Same shape as the `rarecloud_object_storage_regions` data source.
- `pricing` (Attributes) The price card, same shape as on the `rarecloud_object_storage` resource.
- `created_at` (String) ISO-8601 creation timestamp.
