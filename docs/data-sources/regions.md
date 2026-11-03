---
page_title: "rarecloud_regions Data Source - terraform-provider-rarecloud"
description: |-
  Lists the public RareCloud regions.
---

# rarecloud_regions (Data Source)

Lists the public RareCloud regions (`GET /v1/catalog/regions`), optionally filtered. Use it to find the slug `rarecloud_server.region` takes.

## Example Usage

```terraform
data "rarecloud_regions" "cloud" {
  line = "cloud"
}

output "cloud_region_slugs" {
  value = [for r in data.rarecloud_regions.cloud.regions : r.slug]
}
```

## Schema

### Optional

- `backend` (String) Only regions supporting this backend, for example `openstack`, `gardener`, `whmcs` or `proxy`.
- `line` (String) `cloud` for cloud VM and Kubernetes regions, `legacy` for legacy VPS regions.

### Read-Only

- `regions` (Attributes List) One entry per region. (see [below for nested schema](#nestedatt--regions))

<a id="nestedatt--regions"></a>
### Nested Schema for `regions`

Read-Only:

- `slug` (String) Region slug, for example `bucharest-ro`.
- `display_name` (String) Human-readable name.
- `country_code` (String) Country code.
- `city` (String) City.
- `region_group` (String) Geographic group, for example `europe`.
- `available_backends` (List of String) Backends available in the region.
- `active` (Boolean) Whether the region is active.
