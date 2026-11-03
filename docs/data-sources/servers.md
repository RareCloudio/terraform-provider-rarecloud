---
page_title: "rarecloud_servers Data Source - terraform-provider-rarecloud"
description: |-
  Lists every cloud VM the caller owns.
---

# rarecloud_servers (Data Source)

Lists every cloud VM the caller owns (`GET /v1/services?category=cloud-vm`). Takes no input; filter with a `for` expression.

The API reads each service family on a best-effort basis and names a family it could not read in the `X-Partial-Results` header. A list read that way is incomplete, so this data source fails with an error instead of returning it; retry shortly. Billing figures (hourly rate, accrued cost, traffic) are not exposed. `suspended_by` tells an admin suspension (contact support) from a billing one (top up to resume).

## Example Usage

```terraform
data "rarecloud_servers" "all" {}

output "running_server_ips" {
  value = [for s in data.rarecloud_servers.all.servers : s.ipv4 if s.power_status == "running"]
}
```

## Schema

### Read-Only

- `servers` (Attributes List) One entry per cloud VM, in the order the API returns them. (see [below for nested schema](#nestedatt--servers))

<a id="nestedatt--servers"></a>
### Nested Schema for `servers`

Read-Only:

- `id` (String) Server id (the value `rarecloud_server.id` holds).
- `name` (String) Server name.
- `product_name` (String) The plan (flavor) name the VM runs on.
- `status` (String) Lifecycle status: `active`, `pending`, `suspended`, `error`, and so on.
- `status_reason` (String) Why the server is in its current status, when the API gives a reason. Null when the API leaves it out.
- `suspended_by` (String) Who suspended the server: `admin` (abuse or legal, contact support) or `billing` (unpaid, top up to resume). Null when the server is not suspended or the API leaves it out.
- `suspended_reason` (String) The reason an admin gave for the suspension. Null when none was given or the API leaves it out.
- `power_status` (String) Power state: `running`, `stopped` or `unknown`. Null when the API does not report it.
- `region` (String) Region slug. Null when not reported.
- `ipv4` (String) Primary IPv4 address (the public one when the VM has it). Null when none.
- `ipv6` (String) IPv6 address. Null when none.
- `private_ip` (String) Address on the private (tenant) network. Null when none.
- `hostname` (String) Hostname. Null when not reported.
- `cpu` (String) vCPU count as the API words it (for example `2 vCPU`). Null when unknown.
- `ram` (String) Memory as the API words it (for example `8 GB`). Null when unknown.
- `disk` (String) Root disk size as the API words it (for example `80 GB`). Null when unknown.
- `tags` (List of String) The server's tags. Null when the API sends none.
- `created_at` (String) Creation time (RFC 3339).
- `image_name` (String) Name of the image the VM was installed from (for example `Debian 12`). The list reports it only for VMs that recorded it at create or reinstall; null otherwise (read `rarecloud_server.image_name` for older VMs).
