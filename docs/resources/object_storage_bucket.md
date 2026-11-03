---
page_title: "rarecloud_object_storage_bucket Resource - terraform-provider-rarecloud"
description: |-
  An Object Storage bucket.
---

# rarecloud_object_storage_bucket (Resource)

An Object Storage bucket (`POST /v1/object-storage/buckets`).

A bucket's real name, the one S3 clients address and the one on invoices, is `<handle>-<name>`. It is exported as `full_name`. `name` is your part of it.

## The namespace (`handle`)

The handle is the account's namespace, the prefix of every bucket name.

- On the account's **first** bucket it is **required**, and choosing it is permanent.
- On later buckets, leave it out (the provider fills it in) or repeat the same value.
- It can never change. A different value fails at plan time, before anything is destroyed or created.

An account migrated from an existing one has no namespace. Leave `handle` out there.

## Changes

- `name` or `region`: the bucket is **replaced** (destroyed and created again).
- `versioning`, `public`: updated in place (`PATCH`).
- `force_destroy`: changes no infrastructure and makes no API call. It only matters when the bucket is destroyed.

## Destroying a non-empty bucket

Like the AWS provider, destroying a bucket that still holds objects fails with the API's message unless `force_destroy = true`:

```
The bucket is not empty. Delete its contents first, or repeat with purge=true to delete everything in it.
```

With `force_destroy = true` the provider deletes with `purge=true`, which deletes **every object in the bucket** first, irreversibly. To destroy a bucket created without it, set `force_destroy = true`, apply, then destroy.

## Example Usage

```terraform
resource "rarecloud_object_storage_bucket" "assets" {
  name       = "assets"
  region     = "eu-central-1"
  handle     = "acme" # first bucket only; becomes "acme-assets"
  versioning = true
}

resource "rarecloud_object_storage_bucket" "site" {
  name          = "site"
  region        = "eu-central-1"
  force_destroy = true

  depends_on = [rarecloud_object_storage_bucket.assets]
}
```

Access keys are managed with [`rarecloud_object_storage_key`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/object_storage_key).

## Import

```shell
terraform import rarecloud_object_storage_bucket.assets <bucket id>
```

`name` and `handle` are split from the bucket's full name using the account's namespace. `force_destroy` is imported as `false`.

## Schema

### Required

- `name` (String) Your part of the bucket name: 3-40 lower-case letters, digits and hyphens, starting and ending with a letter or digit (no dots). Changing it replaces the bucket.
- `region` (String) Region id, see the `rarecloud_object_storage_regions` data source. Changing it replaces the bucket.

### Optional

- `handle` (String) The account's namespace: 3-16 characters, a lower-case letter then lower-case letters or digits (`^[a-z][a-z0-9]{2,15}$`). Required on the first bucket; later, leave it out or repeat the same value. Immutable.
- `versioning` (Boolean) Keep every version of every object. Default `false`. Updates in place.
- `public` (Boolean) Public delivery is not available yet: leave it unset or `false`, because `true` answers 503 `BACKEND_UNAVAILABLE`. Default `false`. Updates in place.
- `force_destroy` (Boolean) When `true`, destroying the bucket first deletes every object in it. Default `false`.

### Read-Only

- `id` (String) The bucket id (UUID).
- `full_name` (String) The bucket's real name, `<handle>-<name>`.
- `endpoint` (String) The region's S3 endpoint URL.
- `bucket_url` (String) The bucket's virtual-hosted-style URL.
- `public_url` (String) The public CDN URL while `public` is `true`, else null.
- `status` (String) `creating`, `active`, `deleting` or `failed`.
- `created_at` (String) ISO-8601 creation timestamp.
- `imported` (Boolean) True for a bucket brought over from a migrated account. The API lists it but refuses to change or delete it.
