---
page_title: "rarecloud_object_storage_key Resource - terraform-provider-rarecloud"
description: |-
  An S3 access key for Object Storage.
---

# rarecloud_object_storage_key (Resource)

An S3 access key for Object Storage (`POST /v1/object-storage/keys`), scoped to some or all buckets, read-only or read-write.

Keys cannot be edited. Changing `name`, `buckets` or `access` revokes the key and creates a new one, with a new secret. Destroying the resource revokes the key immediately.

## WARNING: the secret is stored in Terraform state in plain text

`secret_access_key` is stored in Terraform state **in plain text, even though it is marked sensitive**. `Sensitive` only redacts it from CLI output and logs. It does not encrypt state. The API returns the secret once, in the create answer, and never again, so state is the only place Terraform can keep it. The provider never asks for it again on refresh.

Protect your state file: encryption at rest, a remote backend with encryption, restricted access. If the secret leaks, replace the key (`terraform apply -replace=rarecloud_object_storage_key.ci`).

The secret is null in two cases, each with a warning:

- **Import.** The secret of an existing key cannot be recovered.
- **A create whose first answer was lost on the network.** The provider retries safely with the same Idempotency-Key and receives a replay without the secret.

In both cases, replace the key to get a usable secret.

## Example Usage

```terraform
resource "rarecloud_object_storage_key" "ci" {
  name    = "ci"
  buckets = ["*"] # every bucket, including future ones
  access  = "readwrite"
}

resource "rarecloud_object_storage_key" "site_reader" {
  name    = "site-reader"
  buckets = [rarecloud_object_storage_bucket.site.id]
  access  = "read"
}

output "ci_secret" {
  value     = rarecloud_object_storage_key.ci.secret_access_key
  sensitive = true
}
```

## Import

```shell
terraform import rarecloud_object_storage_key.ci <key id>
```

## Schema

### Required

- `name` (String) Your label for the key, 1-80 characters. Changing it replaces the key.
- `buckets` (Set of String) Bucket ids, or `["*"]` for every bucket. `"*"` cannot be combined with ids. Changing it replaces the key, which also happens when a bucket it names is replaced.
- `access` (String) `read` or `readwrite`. Changing it replaces the key.

### Read-Only

- `id` (String) The key id (UUID).
- `access_key_id` (String) The S3 access key id.
- `secret_access_key` (String, Sensitive) The S3 secret access key, from the create answer only. Null after an import.
- `secret_preview` (String) A masked preview of the secret.
- `status` (String) `active`. A key revoked outside Terraform leaves state, and the next apply creates a new one.
- `created_at` (String) ISO-8601 creation timestamp.
