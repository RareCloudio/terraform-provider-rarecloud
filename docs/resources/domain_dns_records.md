---
page_title: "rarecloud_domain_dns_records Resource - terraform-provider-rarecloud"
description: |-
  Every DNS host record of one domain, managed as one set.
---

# rarecloud_domain_dns_records (Resource)

Every DNS host record of one domain, managed as one set (`GET/PUT /v1/domains/{id}/dns`).

**This resource owns every record of the domain.** The API replaces the whole set on each write, and a refresh reads the whole set: a record added in the console or by another tool shows as drift and **is removed on the next apply**. Declare one resource per domain and put every record in it.

- Create replaces whatever records the domain has with the configured set.
- Update sends the full new set.
- Destroy clears the records: the API accepts an empty set, so the domain is left with no host records.
- DNS is registrar-dependent. A registrar without a DNS API fails with `NOT_IMPLEMENTED`, at create and at refresh.
- A domain the account no longer owns is dropped from state on refresh.

## Example Usage

```terraform
resource "rarecloud_domain_dns_records" "site" {
  domain_id = rarecloud_domain.site.id

  records = [
    { hostname = "@", type = "A", address = "203.0.113.10" },
    { hostname = "www", type = "CNAME", address = "example.com" },
    { hostname = "@", type = "MX", address = "mail.example.com", priority = 10 },
    { hostname = "@", type = "TXT", address = "v=spf1 mx -all" },
  ]
}
```

## Import

```shell
terraform import rarecloud_domain_dns_records.site <domain id>
```

The import reads the domain's full record set. Nothing is left out: a config listing the same records plans no change after the import.

## Schema

### Required

- `domain_id` (String) The domain id. A change forces replacement.
- `records` (Set of Attributes) The complete record set (at most 100). An empty set removes every record.
  - `hostname` (String, Required) Host name, for example `@`, `www` or `mail`.
  - `type` (String, Required) `A`, `AAAA`, `CNAME`, `MX`, `TXT`, `NS`, `SRV` or `CAA`.
  - `address` (String, Required) The record value (address or target).
  - `priority` (Number, Optional) Priority (0 or more) for `MX` and `SRV` records.

### Read-Only

- `id` (String) Same as `domain_id`.
