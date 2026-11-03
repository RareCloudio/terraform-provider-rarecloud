---
page_title: "rarecloud_sandbox_templates Data Source - terraform-provider-rarecloud"
description: |-
  The AI sandbox templates with their size and hourly price.
---

# rarecloud_sandbox_templates (Data Source)

The AI sandbox templates (`GET /v1/sandboxes/templates`) with their size and hourly price. Takes no input. Use a `slug` as `rarecloud_sandbox.template`. Fails when AI Sandboxes are not available.

## Example Usage

```terraform
data "rarecloud_sandbox_templates" "all" {}

output "sandbox_prices_eur_cents" {
  value = { for t in data.rarecloud_sandbox_templates.all.templates : t.slug => t.hourly_cents_eur }
}
```

## Schema

### Read-Only

- `templates` (Attributes List) One entry per template. (see [below for nested schema](#nestedatt--templates))

<a id="nestedatt--templates"></a>
### Nested Schema for `templates`

Read-Only:

- `slug` (String) Template slug, for example `python`, `node` or `base`.
- `label` (String) Human-readable name.
- `description` (String) What the template contains. Null when not reported.
- `image` (String) The container image the template runs, pinned by digest. Null when not reported.
- `vcpu` (Number) vCPUs. Null when not reported.
- `ram_gb` (Number) Memory in GB. Null when not reported.
- `disk_gb` (Number) Disk in GB. Null when not reported.
- `hourly_cents_eur` (Number) Price per hour of runtime in EUR cents (two decimals of a cent).
- `hourly_cents_usd` (Number) Price per hour of runtime in USD cents, converted at the current rate. Null when the API does not send it.
