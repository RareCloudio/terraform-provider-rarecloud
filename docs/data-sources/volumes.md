---
page_title: "rarecloud_volumes Data Source - terraform-provider-rarecloud"
description: |-
  Lists every block volume the caller owns.
---

# rarecloud_volumes (Data Source)

Lists every block volume the caller owns (`GET /v1/volumes`). Takes no input; filter with a `for` expression.

`size_gb` is read from the API's disk size and `server_id` from its `attached:<server id>` tag, the same way the [`rarecloud_volume`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/volume) resource reads them. Billing figures are not exposed.

## Example Usage

```terraform
data "rarecloud_volumes" "all" {}

output "unattached_volume_ids" {
  value = [for v in data.rarecloud_volumes.all.volumes : v.id if v.server_id == null]
}
```

## Schema

### Read-Only

- `volumes` (Attributes List) One entry per volume. (see [below for nested schema](#nestedatt--volumes))

<a id="nestedatt--volumes"></a>
### Nested Schema for `volumes`

Read-Only:

- `id` (String) Volume id (the value `rarecloud_volume.id` holds).
- `name` (String) Volume name.
- `status` (String) Lifecycle status.
- `region` (String) Region slug. Null when not reported.
- `size_gb` (Number) Size in GB. Null when the API does not report it.
- `server_id` (String) Id of the server the volume is attached to. Null when unattached.
- `tags` (List of String) The tags the API reports, for example `bootable`, `type:<volume type>`, `attached:<server id>`, `unattached`, `managed:kubernetes`.
- `created_at` (String) Creation time (RFC 3339).
