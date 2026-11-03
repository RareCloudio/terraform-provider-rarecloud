---
page_title: "rarecloud_registry_repositories Data Source - terraform-provider-rarecloud"
description: |-
  Every repository in the caller's container registry.
---

# rarecloud_registry_repositories (Data Source)

Every repository in the caller's container registry (`GET /v1/registry/repositories`), sorted by name. The route pages; this data source reads every page. Takes no input. Fails when the registry is not enabled.

## Example Usage

```terraform
data "rarecloud_registry_repositories" "all" {}

output "repository_paths" {
  value = [for r in data.rarecloud_registry_repositories.all.repositories : r.path]
}
```

## Schema

### Read-Only

- `repositories` (Attributes List) One entry per repository. (see [below for nested schema](#nestedatt--repositories))

<a id="nestedatt--repositories"></a>
### Nested Schema for `repositories`

Read-Only:

- `name` (String) Repository name without the registry handle, for example `api`.
- `path` (String) Full repository path including the handle, for example `acme/api` (what goes after the registry host in an image reference).
- `size_bytes` (Number) Stored size in bytes. Null when not reported.
- `image_built_at` (String) Build time of the newest image (the API's `lastPush` field, renamed here because it holds the image's OCI `created` time, not a push time). Null when unknown.
- `newest_tag` (String) Tag of the newest image. Null when none.
