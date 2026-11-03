---
page_title: "rarecloud_load_balancer Resource - terraform-provider-rarecloud"
description: |-
  An L4 (TCP) load balancer with a public IPv4, billed hourly.
---

# rarecloud_load_balancer (Resource)

An L4 (TCP) load balancer with a public IPv4, billed hourly.

Create waits up to 15 minutes for the background build to reach `active`, and destroy waits up to 15 minutes until it is gone. Changing `name`, `port`, `member_server_ids` or `health_check` recreates the load balancer.

## Example Usage

```terraform
resource "rarecloud_load_balancer" "front" {
  name              = "front"
  port              = 443
  member_server_ids = [rarecloud_server.web.id]
}
```

## Import

```shell
terraform import rarecloud_load_balancer.front <load balancer id>
```

The import reads `name`, `port` (the port of the pool members), `status`, `ipv4`, `members`, `api_access`.

Not imported (null after the import):

- `member_server_ids`: the API lists pool members by private IP, not by server id.
- `health_check`: not returned by the API.
- `port`, only when the load balancer has no pool members yet.

Set them in the config as they were at creation. The first plan after the import shows them as an in-place update that only records the value in state; it never replaces the resource. From the first apply on, a change replaces the resource as before. To skip that first update, use `lifecycle { ignore_changes = [...] }` for the attributes that differ.

## Schema

### Required

- `name` (String) Display name. A change recreates the load balancer.
- `port` (Number) Listener and member port, 1 to 65535. A change recreates the load balancer.
- `member_server_ids` (List of String) VMs to balance across (at least one). A change recreates the load balancer.

### Optional

- `health_check` (Boolean) Health-check the members (on by default). A change recreates the load balancer.

### Read-Only

- `id` (String)
- `status` (String) `pending`, `active`, `error` or `deleting`.
- `ipv4` (String)
- `members` (Attributes List) Live pool members (`id`, `address`, `port`, `status`).
- `api_access` (String)
