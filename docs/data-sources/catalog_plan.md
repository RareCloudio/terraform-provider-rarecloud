---
page_title: "rarecloud_catalog_plan Data Source - terraform-provider-rarecloud"
description: |-
  Look up a catalog plan by SKU. Returns specs, billing track availability and prices.
---

# rarecloud_catalog_plan (Data Source)

Looks up a catalog plan (`GET /v1/catalog/products/{sku}`) and returns its specs, the billing tracks it supports and its EUR prices. Use it to validate a `rarecloud_server` `plan` at plan time, or to read a VM's hourly rate.

`sku` accepts either form:

- the public product SKU, the value `rarecloud_server.plan` takes (e.g. `g-2vcpu-8gb`), resolved with a single request;
- the plan SKU (e.g. `openstack.g-2vcpu-8gb.default`) or the internal product SKU (e.g. `openstack.g-2vcpu-8gb`).

Cloud VM hourly rates are fractional cents (for example `0.5466` EUR cents per hour). Read them from `hourly_price_eur`, which carries the exact value. `hourly_price_cents_eur` is kept for existing configurations and is rounded to a whole cent.

A catalog answer the provider cannot read fails the lookup with an error that names the product, instead of reporting `Plan not found`.

## Example Usage

```terraform
data "rarecloud_catalog_plan" "web" {
  sku = "g-2vcpu-8gb"
}

output "web_hourly_eur" {
  value = data.rarecloud_catalog_plan.web.hourly_price_eur # 0.020534
}

resource "rarecloud_server" "web" {
  plan = data.rarecloud_catalog_plan.web.sku
  # ...
}
```

## Schema

### Required

- `sku` (String) Public product SKU (e.g. `g-2vcpu-8gb`, the value `rarecloud_server.plan` takes) or plan SKU (e.g. `openstack.g-2vcpu-8gb.default`).

### Read-Only

- `product_sku` (String) Internal product SKU of the plan (e.g. `openstack.g-2vcpu-8gb`).
- `vcpu` (Number) vCPU count (0 when the plan does not publish it).
- `ram_mb` (Number) RAM in MB (0 when not published).
- `disk_gb` (Number) Disk in GB (0 when not published).
- `bandwidth_gb` (Number) Included monthly transfer in GB (0 when not published).
- `billing_tracks` (List of String) Subset of `["prepay", "cloud"]` indicating which billing models the plan supports.
- `hourly_price_eur` (Number) Exact hourly cloud-billing price in EUR (e.g. `0.005466`). Null if the cloud track is not offered.
- `monthly_price_eur` (Number) Exact monthly price in EUR (e.g. `3.99`). Null if no monthly cycle is offered.
- `hourly_price_cents_eur` (Number) Hourly cloud-billing price in EUR cents, rounded to a whole cent (0 if the cloud track is not offered). Use `hourly_price_eur` for the exact rate.
- `monthly_price_cents_eur` (Number) Monthly price in EUR cents, rounded to a whole cent (0 if no monthly cycle is offered).
