---
page_title: "rarecloud_kubernetes_node_pool Resource - terraform-provider-rarecloud"
description: |-
  An extra node pool on a managed Kubernetes cluster.
---

# rarecloud_kubernetes_node_pool (Resource)

An extra node pool on a [`rarecloud_kubernetes_cluster`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/kubernetes_cluster), added with `POST /v1/services/{id}/pools`. A cluster has at most 5 pools, its `default_pool` included. Every field except `service_id` is edited in place.

Destroy removes the pool and its nodes (`DELETE /v1/services/{id}/pools/{pool}`). The API refuses to remove a cluster's last pool.

Every pool change makes the API rewrite the cluster's whole pool list, so the provider never runs two pool writes on one cluster at the same time. When the API answers `409 CONFLICT` because the cluster is busy with another change, the write is retried, 6 attempts over about 1.5 minutes.

## Example Usage

```terraform
resource "rarecloud_kubernetes_node_pool" "batch" {
  service_id   = rarecloud_kubernetes_cluster.prod.id
  name         = "batch"
  machine_type = "g-4vcpu-16gb"
  minimum      = 1
  maximum      = 4
}
```

## Schema

### Required

- `service_id` (String) The `rarecloud_kubernetes_cluster` id. Changing it recreates the pool.
- `name` (String) Pool name: lowercase letters, digits and hyphens, starting with a letter, at most 15 characters (checked by the API). A change renames the pool in place (`POST /v1/services/{id}/pools/{pool}/rename`), which replaces its nodes one by one.
- `minimum` (Number) Autoscaler minimum node count (at least 1).
- `maximum` (Number) Autoscaler maximum node count (at least `minimum`, at most 16).

### Optional

- `machine_type` (String) Node machine type name (for example `g-4vcpu-16gb`). Defaults to the API's default machine type. A change is applied in place (`PATCH /v1/services/{id}/pools/{pool}`) and replaces the pool's nodes.
- `volume_size_gb` (Number) Per-node root volume in GB (10 to 1000). Defaults to the machine type's root disk.

### Read-Only

- `id` (String) `<service_id>/<name>`.

## Import

Import by `<service_id>/<pool name>`:

```shell
terraform import rarecloud_kubernetes_node_pool.batch k8s-1a2b3c4d/batch
```
