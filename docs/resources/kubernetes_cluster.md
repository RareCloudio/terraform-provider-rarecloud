---
page_title: "rarecloud_kubernetes_cluster Resource - terraform-provider-rarecloud"
description: |-
  A managed Kubernetes cluster with its default node pool.
---

# rarecloud_kubernetes_cluster (Resource)

A managed Kubernetes cluster, created with `POST /v1/services` (category `cloud-k8s`). It is billed by the hour for what it runs (worker nodes, the HA control plane when enabled, load balancers it creates); there is no order or invoice at create.

Create is asynchronous on the API side. The provider waits until the cluster reads `active` (default 45 minutes, `create_timeout`). If the build fails (`error`) or the wait times out, the apply fails with the API's status and reason, and the cluster stays in state as tainted, so the next apply replaces it instead of leaving it running untracked.

The cluster owns its first node pool, `default_pool`. Add more pools with [`rarecloud_kubernetes_node_pool`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/kubernetes_node_pool) (at most 5 per cluster).

**Destroy deletes the cluster for good** (`DELETE /v1/services/{id}`): there is no cancellation period and no undo. Its nodes and everything running on them are removed, and hourly billing stops as its resources are deleted. Destroy waits until the API no longer finds the cluster (default 30 minutes, `delete_timeout`).

## Example Usage

```terraform
resource "rarecloud_kubernetes_cluster" "prod" {
  name = "prod"
  plan = "g-2vcpu-8gb"

  default_pool = {
    name    = "default"
    minimum = 1
    maximum = 3
  }
}
```

For more pools and for credentials, see [`rarecloud_kubernetes_node_pool`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/kubernetes_node_pool) and [`rarecloud_kubernetes_kubeconfig`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/kubernetes_kubeconfig).

## Schema

### Required

- `name` (String) Display name, 1 to 40 characters. The API cannot rename a cluster: changing it recreates the cluster.
- `default_pool` (Attributes) The cluster's first node pool. Every field is edited in place. See [below](#nested-schema-for-default_pool).

### Optional

- `plan` (String) The cluster's default node machine type, as a catalog SKU or a machine type name (for example `g-2vcpu-8gb`; see `rarecloud_catalog_plan`). The API resolves it and rejects a value it does not know; the provider does not validate it. Used only at create, for pools that set no `machine_type`; when omitted, the API's default machine type is used. Changing it recreates the cluster. The API does not report it back, so it is null after `terraform import`, and setting it then does not recreate the cluster.
- `k8s_version` (String) Kubernetes version, one of `GET /v1/catalog/kubernetes-versions` (the API rejects any other). Defaults to the API's current default. There is no in-place upgrade: changing it recreates the cluster.
- `high_availability` (Boolean) HA control plane, billed as an add-on. `false` to `true` is applied in place (`POST /v1/services/{id}/high-availability`). The API cannot remove it, so `true` to `false` recreates the cluster.
- `create_timeout` (String) How long create waits for `active`, as a duration such as `60m`. Default `45m`. Not sent to the API.
- `delete_timeout` (String) How long destroy waits for the cluster to be gone, as a duration such as `45m`. Default `30m`. Not sent to the API.

### Read-Only

- `id` (String) The cluster id (`k8s-...`).
- `region` (String) Region the cluster runs in (chosen by the API).
- `status` (String) `pending`, `active`, `error`, `suspended`, `hibernated` or `terminating`.
- `api_endpoint` (String) URL of the cluster's Kubernetes API server.

<a id="nested-schema-for-default_pool"></a>
### Nested Schema for `default_pool`

Required:

- `name` (String) Pool name: lowercase letters, digits and hyphens, starting with a letter, at most 15 characters (checked by the API). A change renames the pool in place (`POST /v1/services/{id}/pools/{pool}/rename`), which replaces its nodes one by one.
- `minimum` (Number) Autoscaler minimum node count (at least 1).
- `maximum` (Number) Autoscaler maximum node count (at least `minimum`, at most 16).

Optional:

- `machine_type` (String) Node machine type name (for example `g-4vcpu-16gb`). Defaults to the machine type resolved from `plan`. A change is applied in place (`PATCH /v1/services/{id}/pools/{pool}`) and replaces the pool's nodes.
- `volume_size_gb` (Number) Per-node root volume in GB (10 to 1000). Defaults to the machine type's root disk.

## Pool writes

Every pool change makes the API rewrite the cluster's whole pool list, so the provider never runs two pool writes on one cluster at the same time (the default pool and every `rarecloud_kubernetes_node_pool` on it share one lock). When the API answers `409 CONFLICT` because the cluster is busy with another change, the write is retried, 6 attempts over about 1.5 minutes.

## Import

Import by cluster id. The default pool is read from the cluster's first pool; `plan` stays null.

```shell
terraform import rarecloud_kubernetes_cluster.prod k8s-1a2b3c4d
```
