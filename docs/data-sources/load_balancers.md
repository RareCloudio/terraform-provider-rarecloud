---
page_title: "rarecloud_load_balancers Data Source - terraform-provider-rarecloud"
description: |-
  Lists every load balancer the caller owns.
---

# rarecloud_load_balancers (Data Source)

Lists every load balancer the caller owns (`GET /v1/load-balancers`), the ones a Kubernetes cluster created included (tag `managed:kubernetes`). Takes no input; filter with a `for` expression. Members and billing figures are not part of the list.

## Example Usage

```terraform
data "rarecloud_load_balancers" "all" {}

output "load_balancer_ips" {
  value = { for lb in data.rarecloud_load_balancers.all.load_balancers : lb.name => lb.ipv4 }
}
```

## Schema

### Read-Only

- `load_balancers` (Attributes List) One entry per load balancer. (see [below for nested schema](#nestedatt--load_balancers))

<a id="nestedatt--load_balancers"></a>
### Nested Schema for `load_balancers`

Read-Only:

- `id` (String) Load balancer id (the value `rarecloud_load_balancer.id` holds).
- `name` (String) Load balancer name.
- `status` (String) Lifecycle status: `active`, `pending`, `error`, `deleting`, and so on.
- `status_reason` (String) Why the status is `error`, when the API can say it in one line. Null otherwise.
- `region` (String) Region slug. Null when not reported.
- `ipv4` (String) The public address to point DNS at (the private VIP when there is no public one). Null when not reported.
- `tags` (List of String) The tags the API reports: `listeners:<n>`, `ops:<operating status>`, `vip:<private VIP>`, and `managed:kubernetes` for one a cluster created. Null when none.
- `created_at` (String) Creation time (RFC 3339).
