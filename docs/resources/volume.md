---
page_title: "rarecloud_volume Resource - terraform-provider-rarecloud"
description: |-
  A block volume, billed per GB.
---

# rarecloud_volume (Resource)

A block volume, billed per GB.

Setting or clearing `server_id` attaches or detaches the volume in place. Changing `size_gb` or `name` recreates the volume.

## Example Usage

```terraform
resource "rarecloud_volume" "data" {
  name      = "data"
  size_gb   = 50
  server_id = rarecloud_server.web.id
}
```

## Import

```shell
terraform import rarecloud_volume.data <volume id>
```

The import reads every attribute: `name`, `size_gb` (from the volume size the API reports), `server_id`, `status`, `api_access`.

Nothing is left out: a config matching the real object plans no change after the import.

## Schema

### Required

- `size_gb` (Number) Size in GB (1 to 2048). A change recreates the volume.

### Optional

- `name` (String) Display name, at most 253 characters. Defaults to `volume`. A change recreates the volume.
- `server_id` (String) `rarecloud_server.id` to attach to.

### Read-Only

- `id` (String)
- `status` (String)
- `api_access` (String)
