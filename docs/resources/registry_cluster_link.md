---
page_title: "rarecloud_registry_cluster_link Resource - terraform-provider-rarecloud"
description: |-
  Links an owned Kubernetes cluster to the container registry.
---

# rarecloud_registry_cluster_link (Resource)

Links one of the account's OWN Kubernetes clusters to the container registry (`POST /v1/registry/clusters/{service_id}`), installing a small controller (the RareCloud registry secret-syncer) that fans a pull credential out to every namespace, present and future. Ownership of `service_id` is checked server-side.

A cluster that is not reachable yet is not an error: the link is recorded as `linking` and finished by the hourly reconciler.

Destroying this resource unlinks the cluster (`DELETE /v1/registry/clusters/{service_id}`): everything the syncer installed is removed and the minted pull credential is revoked. `disclosure` says what the installed controller can read and write across the cluster; read it before applying.

Import is not supported.

For a cluster RareCloud does NOT manage (BYO), use [`rarecloud_registry_kubernetes_manifest`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/registry_kubernetes_manifest) instead.

## Example Usage

```terraform
resource "rarecloud_registry_cluster_link" "prod" {
  service_id = rarecloud_kubernetes_cluster.prod.id
}

output "cluster_link_disclosure" {
  value = rarecloud_registry_cluster_link.prod.disclosure
}
```

## Schema

### Required

- `service_id` (String) The `rarecloud_kubernetes_cluster.id` to link. Must be a cluster the caller owns. Immutable: linking a different cluster is a different resource.

### Read-Only

- `id` (String) Same value as `service_id`.
- `status` (String) `linking`, `active`, `rotating`, `unlinking`, or `error` (see `last_error`).
- `cluster_name` (String) The cluster's display name, joined from the account's owned clusters list. Falls back to `service_id` if the cluster can no longer be found.
- `installed_at` (String) When the secret-syncer was last installed cleanly, or null before the first successful install.
- `syncer_image` (String) The digest-pinned secret-syncer image currently installed, or null before the first install.
- `syncer_ready` (Boolean) Whether the secret-syncer Deployment last reported itself Available. A syncer that crash-loops right after a successful install reports `status = "active"` with `syncer_ready = false`.
- `syncer_checked_at` (String) When `syncer_ready` was last (re)probed, or null if it never has been.
- `last_error` (String) A short, secret-free reason the last attempt did not complete, or null.
- `credential_name` (String) The pull credential minted for this cluster, e.g. `k8s-<service_id>`. Null before the first successful link. Refreshed on every read from the current list item, so it reflects a later rotation too, not just what was minted at link time.
- `secret_name` (String) The Kubernetes Secret name installed in the cluster (`rarecloud-registry-credentials` today). Captured once at link time.
- `installs` (List of String) The Kubernetes objects applied by the initial link call. Captured once at link time.
- `disclosure` (String) What the installed controller can read and write across the cluster, captured once at link time.
