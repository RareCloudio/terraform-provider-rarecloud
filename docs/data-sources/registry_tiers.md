---
page_title: "rarecloud_registry_tiers Data Source - terraform-provider-rarecloud"
description: |-
  Lists every container registry billing tier.
---

# rarecloud_registry_tiers (Data Source)

Lists every container registry billing tier (`GET /v1/registry/tiers`): storage quota, burst ceiling, monthly price in EUR/USD cents, per-GB overage price in EUR and USD cents, repository limit, and whether the tier currently accepts new enablements.

Takes no input; always returns the full catalog. Use it to validate a [`rarecloud_registry`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/registry) `tier` choice at plan time, or to build a price table.

## Example Usage

```terraform
data "rarecloud_registry_tiers" "all" {}

output "starter_tier_monthly_eur_cents" {
  value = [for t in data.rarecloud_registry_tiers.all.tiers : t.monthly_price_cents_eur if t.tier == "starter"][0]
}

output "starter_tier_max_repos" {
  value = [for t in data.rarecloud_registry_tiers.all.tiers : t.max_repos if t.tier == "starter"][0]
}
```

## Schema

### Read-Only

- `tiers` (Attributes List) One entry per tier (`free`, `starter`, `premium`, `enterprise`). (see [below for nested schema](#nestedatt--tiers))

<a id="nestedatt--tiers"></a>
### Nested Schema for `tiers`

Read-Only:

- `tier` (String) `free`, `starter`, `premium`, or `enterprise`.
- `quota_gb` (Number) Included storage quota, in GB.
- `burst_ceiling_gb` (Number) Hard ceiling above the quota before push is blocked, in GB.
- `monthly_price_cents_eur` (Number) Monthly price in EUR cents.
- `monthly_price_cents_usd` (Number) Monthly price in USD cents.
- `overage_cents_per_gb_month` (Number) Price per GB over quota, per month, in EUR cents.
- `overage_cents_per_gb_month_usd` (Number) Price per GB over quota, per month, in USD cents (converted from EUR at the current rate, two decimals of a cent).
- `max_repos` (Number) The most repositories a registry on this tier may hold.
- `available` (Boolean) Whether a new registry can currently enable at this tier.
