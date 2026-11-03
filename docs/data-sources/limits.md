---
page_title: "rarecloud_limits Data Source - terraform-provider-rarecloud"
description: |-
  The caller's effective cloud quotas with live usage.
---

# rarecloud_limits (Data Source)

The caller's effective cloud quotas with live usage (`GET /v1/limits`), plus the container registry and AI sandbox figures. Takes no input. Use it for plan-time checks, for example that a new pool fits in the remaining `cores`.

A `limit` of `-1` means unlimited. When the service that owns a quota cannot be read, its row has `unavailable = true`, `state = "unknown"` and null `used` and `limit`: unknown, not zero.

## Example Usage

```terraform
data "rarecloud_limits" "current" {}

locals {
  cores = one([for r in data.rarecloud_limits.current.resources : r if r.resource == "cores"])
}

output "cores_left" {
  value = local.cores.limit == -1 ? null : local.cores.limit - local.cores.used
}
```

## Schema

### Read-Only

- `resources` (Attributes List) One row per quota dimension (`cores`, `ram` in MB, `instances`, `volumes`, `gigabytes`, `floatingip`, `network`, `router`, `subnet`, `port`, `security_group`). (see [below for nested schema](#nestedatt--resources))
- `nearing_limit` (Boolean) True when any readable row is at `warn` or `danger`.
- `registry` (Attributes) The container registry's storage and repository figures. Null when the caller has no registry or when they could not be read. (see [below for nested schema](#nestedatt--registry))
- `sandboxes` (Attributes) Running AI sandboxes against the account's limit. Null when the product is off or when the figures could not be read. (see [below for nested schema](#nestedatt--sandboxes))

<a id="nestedatt--resources"></a>
### Nested Schema for `resources`

Read-Only:

- `resource` (String) Quota dimension.
- `used` (Number) Amount in use. Null when the owning service could not be read (`unavailable`).
- `limit` (Number) Effective limit; `-1` means unlimited. Null when `unavailable`.
- `unavailable` (Boolean) True when the owning service could not be read; `used` and `limit` are then null and `state` is `unknown`.
- `state` (String) `ok`, `warn` (from 80 %), `danger` (from 95 %) or `unknown`.
- `ratio` (Number) Fraction of the limit in use, 0 to 1. Null when unlimited or unknown.

<a id="nestedatt--registry"></a>
### Nested Schema for `registry`

Read-Only:

- `tier` (String) Registry tier.
- `status` (String) Registry account status.
- `stored_bytes` (Number) Bytes stored at the last hourly measurement. Null before the first one.
- `sampled_at` (String) When that measurement was taken (RFC 3339). Null before the first one.
- `quota_gb` (Number) Storage included in the tier, in decimal GB.
- `burst_ceiling_gb` (Number) Where pushes stop, in decimal GB.
- `repo_count` (Number) Repositories in use. Null when not known.
- `max_repos` (Number) The tier's repository limit.

<a id="nestedatt--sandboxes"></a>
### Nested Schema for `sandboxes`

Read-Only:

- `used` (Number) Sandboxes running now.
- `limit` (Number) The account's limit.
- `state` (String) `ok`, `warn` or `danger`.
- `cluster_cap` (Number) Set only when the platform lets fewer run than `limit` for now. Null otherwise.
