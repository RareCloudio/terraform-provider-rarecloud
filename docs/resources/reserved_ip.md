---
page_title: "rarecloud_reserved_ip Resource - terraform-provider-rarecloud"
description: |-
  A reserved (static) public IPv4, billed at a flat rate while reserved.
---

# rarecloud_reserved_ip (Resource)

A reserved (static) public IPv4, billed at a flat rate while reserved.

Setting or clearing `server_id` attaches or detaches the IP in place. Destroying the resource releases the IP, which cannot be undone.

## Example Usage

```terraform
resource "rarecloud_reserved_ip" "web" {
  server_id = rarecloud_server.web.id
}
```

## Import

```shell
terraform import rarecloud_reserved_ip.web <reserved ip id>
```

The import reads every attribute: `ip`, `server_id`, `region`, `attached`.

Nothing is left out: a config matching the real object plans no change after the import.

## Schema

### Optional

- `server_id` (String) `rarecloud_server.id` to attach the IP to. The VM must have a private (VPC) network interface.

### Read-Only

- `id` (String)
- `ip` (String)
- `region` (String)
- `attached` (Boolean)
