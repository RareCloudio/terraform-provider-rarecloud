---
page_title: "rarecloud_product Data Source - terraform-provider-rarecloud"
description: |-
  One catalog product by SKU.
---

# rarecloud_product (Data Source)

One catalog product by SKU (`GET /v1/catalog/products/{sku}/details`, public): what it is, where it is offered, and its plan SKUs. Plan rows, prices and billing cycles are the job of the [`rarecloud_catalog_plan`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/data-sources/catalog_plan) data source: pass it one of `plan_skus`. The free-form product metadata and the billing cycles and config options are not modelled. An unknown or retired SKU is an error.

## Example Usage

```terraform
data "rarecloud_product" "vm" {
  sku = "g-2vcpu-8gb"
}

output "vm_regions" {
  value = data.rarecloud_product.vm.available_regions
}
```

## Schema

### Required

- `sku` (String) Public product SKU (for example `g-2vcpu-8gb`). An unknown or retired SKU is an error.

### Read-Only

- `backend` (String) The backend that provisions the product, for example `openstack` or `whmcs`.
- `kind` (String) Product kind, for example `cloud_compute`, `cloud_volume`, `legacy_vps` or `hosting`.
- `display_name` (String) Human-readable name.
- `description` (String) Description. Null when the product has none.
- `category` (String) Catalog category. Null when the product has none.
- `active` (Boolean) Whether the product is active in the catalog.
- `usable_for` (List of String) What the product can be used for, for example `compute` and `k8s`. Null when the catalog does not say.
- `available_regions` (List of String) Region slugs where the product's backend is offered.
- `plan_skus` (List of String) SKUs of the product's visible plans (each one works with `rarecloud_catalog_plan`).
