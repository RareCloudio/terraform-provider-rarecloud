---
page_title: "rarecloud_object_storage_regions Data Source - terraform-provider-rarecloud"
description: |-
  The regions Object Storage buckets can be created in.
---

# rarecloud_object_storage_regions (Data Source)

The regions Object Storage buckets can be created in (`GET /v1/object-storage/regions`). Works before Object Storage is enabled.

## Example Usage

```terraform
data "rarecloud_object_storage_regions" "all" {}

locals {
  eu_regions = [for r in data.rarecloud_object_storage_regions.all.regions : r.id if r.eu]
}
```

## Schema

### Read-Only

- `regions` (Attributes List)
  - `id` (String) Region id, the value `rarecloud_object_storage_bucket.region` takes.
  - `label` (String) Human-readable location.
  - `endpoint` (String) The S3 endpoint URL for this region.
  - `country` (String) ISO 3166-1 alpha-2 country code.
  - `eu` (Boolean) Whether the region is inside the EU. The UK regions are not.
