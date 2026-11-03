---
page_title: "rarecloud_domain Resource - terraform-provider-rarecloud"
description: |-
  A domain registration, placed as an order (`POST /v1/domains`).
---

# rarecloud_domain (Resource)

A domain registration, placed as an order (`POST /v1/domains`).

Registration is asynchronous: the order starts `Pending` and the registrar fulfils it once it is paid. Destroying the resource only removes it from state; the domain stays registered. `nameservers`, `id_protection` and `contacts` change in place. Changing `domain` (other than its letter case) forces replacement, a new registration. Changing `years` or `dns_management` on an existing domain fails at plan time.

## Example Usage

```terraform
resource "rarecloud_domain" "site" {
  domain        = "example.com"
  years         = 1
  nameservers   = ["ns1.example.net", "ns2.example.net"]
  id_protection = true

  contacts = {
    registrant = {
      first_name = "Ana"
      last_name  = "Popescu"
      email      = "ana@example.com"
      country    = "RO"
      phone      = "+40.723000000"
    }
  }
}
```

## Contacts

`contacts.registrant` is the registrant (owner) WHOIS contact, the only contact the API manages (`GET/PUT /v1/domains/{id}/contacts`); the registrar's admin, billing and tech contacts are left as they are. Only the fields you set are sent, and only those are refreshed: a field you leave out keeps the registrar's value and never shows a diff. A refresh of an active domain reads the managed fields back, so a change made in the console shows as drift (`email` and `country` compare without letter case, `phone` by its digits). Leaving `contacts` out of the config sends nothing and reads nothing.

The register order does not carry contacts. Set at creation, they are recorded in state with a warning; once the domain is active the next plan shows any difference from the registrar's contact, and an apply sets it. A change fails while the register order is still pending. If the registrar cannot report contacts, the refresh keeps the state value and prints a warning.

To manage the domain's DNS host records, use [`rarecloud_domain_dns_records`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/domain_dns_records).

## Import

```shell
terraform import rarecloud_domain.site <domain id>
```

The import reads `domain`, `nameservers` (live, from the registrar, for an active domain), `id_protection`, `status`, `expiry_date`, `registrar`, `api_access`.

Not imported (null after the import):

- `years`, `dns_management`: purchase-time inputs the API does not return.
- `order_id`, `invoice_id`: belong to the register order, not to the domain record.
- `contacts`: not read at import (only the fields you set are managed). If your config sets them, the first apply sends them to the registrar in place.

Set `years` and `dns_management` as they were at registration, or leave them out. The first plan after the import records them in place: no register order is placed and the plan-time check on these two attributes applies from the first apply on. `nameservers` and `id_protection` are read from the live domain; if you do not manage them, leave them out of the config: after the import they mirror the live value in state until you manage them, no change is planned and nothing is sent. Each refresh of an active domain reads them (one extra `GET /v1/domains/{id}/nameservers` call, and a warning if that read fails). On a domain Terraform created without them they stay null and are not read. With only `domain` in the config, the plan right after the import shows no change at all. To skip the first update, use `lifecycle { ignore_changes = [years, dns_management] }`.

## Schema

### Required

- `domain` (String) Domain to register. A change (other than letter case) forces replacement.

### Optional

- `years` (Number) Registration period in years (1 to 10, default 1). Purchase-time only.
- `nameservers` (List of String) Nameservers (up to 5; at least 2 for an in-place change), changed in place.
- `id_protection` (Boolean) WHOIS ID protection, changed in place.
- `dns_management` (Boolean) DNS management. Purchase-time only.
- `contacts` (Attributes) WHOIS contacts, changed in place. See [Contacts](#contacts).
  - `registrant` (Attributes, Required) The registrant contact: `first_name`, `last_name`, `organisation`, `email`, `address1`, `address2`, `city`, `state`, `postcode`, `country` (2-letter ISO code), `phone` (all optional String). `first_name` and `last_name` must not be empty when set.

### Read-Only

- `id` (String) Domain id (the order id until the domain record exists).
- `order_id`, `invoice_id`, `status`, `expiry_date`, `registrar` (String)
- `api_access` (String)
