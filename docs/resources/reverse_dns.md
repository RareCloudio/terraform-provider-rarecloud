---
page_title: "rarecloud_reverse_dns Resource - terraform-provider-rarecloud"
description: |-
  The PTR record of a cloud VM's primary public IPv4.
---

# rarecloud_reverse_dns (Resource)

Manages the PTR record of a cloud VM's primary public IPv4 (`GET`, `PUT` and `DELETE /v1/services/{id}/rdns`). The API derives the address from the VM, so you never send an IP.

Changing `hostname` updates the record in place. Changing `service_id` creates a new resource. Destroying it removes only the PTR record, never the VM. A refresh drops the resource from state when no PTR record exists.

The VM must have a primary public IPv4 inside a RareCloud reverse zone; otherwise the API answers `INVALID_PARAM`. If the VM was deleted outside Terraform, refresh and destroy fail with the API's error: remove the resource with `terraform state rm`.

## Example Usage

```terraform
resource "rarecloud_reverse_dns" "web" {
  service_id = rarecloud_server.web.id
  hostname   = "web.example.com"
}
```

## Import

```shell
terraform import rarecloud_reverse_dns.web <server id>
```

The import reads `hostname`, `ip` and `ttl`. A VM with no PTR record has nothing to import.

## Schema

### Required

- `service_id` (String) The `rarecloud_server.id` whose primary IPv4 receives the record. A change creates a new resource.
- `hostname` (String) Fully qualified hostname with at least two labels, for example `mail.example.com`. Write it in lower case without a trailing dot: the API normalizes it, and a value that differs fails the apply with an inconsistent result. A change updates the record in place.

### Read-Only

- `id` (String) Same value as `service_id`.
- `ip` (String) Primary public IPv4 resolved by the API.
- `ttl` (Number) PTR TTL in seconds.
