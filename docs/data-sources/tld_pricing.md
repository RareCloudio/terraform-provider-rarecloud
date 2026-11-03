---
page_title: "rarecloud_tld_pricing Data Source - terraform-provider-rarecloud"
description: |-
  Register, transfer and renew prices for one TLD.
---

# rarecloud_tld_pricing (Data Source)

The register, transfer and renew prices of one TLD (`GET /v1/domains/tld-pricing`), for checking a price before a `rarecloud_domain`. Requires the `domains:read` scope on the token. A TLD without pricing is an error.

## Example Usage

```terraform
data "rarecloud_tld_pricing" "com" {
  tld = "com"
}

output "com_register_price" {
  value = "${data.rarecloud_tld_pricing.com.register} ${data.rarecloud_tld_pricing.com.currency}"
}
```

## Schema

### Required

- `tld` (String) TLD, for example `com` or `io`. Letter case and a leading dot are ignored.

### Read-Only

- `register` (Number) First-period registration price. 0 if not offered.
- `transfer` (Number) Transfer-in price. 0 if not offered.
- `renew` (Number) Renewal price. 0 if not offered.
- `currency` (String) ISO currency code, for example `EUR`.
