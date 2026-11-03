---
page_title: "rarecloud_os_templates Data Source - terraform-provider-rarecloud"
description: |-
  The OS templates a product can be ordered with, before purchase.
---

# rarecloud_os_templates (Data Source)

The OS templates a product can be ordered with, before purchase (`GET /v1/catalog/products/{sku}/os-templates`, public). It serves the legacy and dedicated VPS products; cloud VMs take an image instead (see the `rarecloud_image` data source). An unknown or retired SKU is an error.

## Example Usage

```terraform
data "rarecloud_os_templates" "vps" {
  sku = "kvm-servers-starter"
}

output "os_slugs" {
  value = [for t in data.rarecloud_os_templates.vps.templates : t.slug]
}
```

## Schema

### Required

- `sku` (String) Public product SKU. An unknown or retired SKU is an error.

### Read-Only

- `templates` (Attributes List) The templates offered for the product. Empty when the catalog has none for it. (see [below for nested schema](#nestedatt--templates))

<a id="nestedatt--templates"></a>
### Nested Schema for `templates`

Read-Only:

- `slug` (String) Template slug (the value to order with).
- `display_name` (String) Human-readable name, for example `Ubuntu`.
- `version` (String) Version label, for example `24.04 LTS`. Null when the template has none.
