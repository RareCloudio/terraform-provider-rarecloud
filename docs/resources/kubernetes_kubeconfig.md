---
page_title: "rarecloud_kubernetes_kubeconfig Resource - terraform-provider-rarecloud"
description: |-
  A long-lived kubeconfig credential for a managed Kubernetes cluster.
---

# rarecloud_kubernetes_kubeconfig (Resource)

A long-lived kubeconfig credential for a [`rarecloud_kubernetes_cluster`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/kubernetes_cluster), created with `POST /v1/services/{id}/kubeconfigs`. Each credential is its own ServiceAccount in the cluster with an `admin` or `view` role and a bound token. The API has no update for a credential, so every argument it knows forces a new credential.

The `kubeconfig` YAML is stored **in Terraform state in plain text**. `Sensitive` only hides it from CLI output and logs, it does not encrypt state. Use encrypted remote state, or the [ephemeral `rarecloud_kubernetes_kubeconfig`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/ephemeral-resources/kubernetes_kubeconfig) to keep it out of state.

## Refresh

Refresh reads the credential from `GET /v1/services/{id}/kubeconfigs` (metadata only; the YAML in state is kept as it is).

- `active`: kept.
- `revoking`: kept, with a warning. The credential was revoked outside Terraform and the API already refuses it; deleting its ServiceAccount from the cluster is still pending (the cluster could not be reached). It stays in state until that cleanup finishes. To get a working credential right away, replace it with `terraform apply -replace=<address>`.
- `revoked` or `expired`: removed from state, so the next apply creates a new credential.
- Not listed, or the cluster is gone (404): removed from state.

## Destroy

Destroy revokes the credential (`DELETE /v1/services/{id}/kubeconfigs/{credential_id}`). The API marks it revoked first and then deletes its ServiceAccount from the cluster. When the cluster cannot be reached (for example while it is hibernated) the API answers `revoking`: the credential is already refused by the API, and the API deletes the ServiceAccount on its own later. Destroy then polls until the credential reads `revoked`, for at most `delete_timeout` (default `5m`). If that runs out, destroy still succeeds, with a warning that names the credential and says the cleanup is pending on the API side. A credential the API answers 404 for counts as already revoked.

## Example Usage

```terraform
resource "rarecloud_kubernetes_kubeconfig" "ci" {
  service_id = rarecloud_kubernetes_cluster.prod.id
  name       = "ci"
  role       = "view"
  ttl        = "90d"
}
```

## Schema

### Required

- `service_id` (String) The `rarecloud_kubernetes_cluster` id. Changing it recreates the credential.
- `name` (String) Credential name, 1 to 64 characters. Changing it recreates the credential.
- `role` (String) `admin` or `view`. Changing it recreates the credential.

### Optional

- `ttl` (String) `30d`, `90d`, `1y` or `never`. Omitted, the API applies `90d` and the attribute stays null. Changing it recreates the credential. The API does not report the preset back, so an imported credential has a null `ttl`: leave it out of the configuration to avoid a replacement.
- `allow_never_expiry` (Boolean) Must be `true` to use `ttl = "never"`, a credential that never expires and lives in state.
- `delete_timeout` (String) How long destroy waits for a `revoking` credential to become `revoked`, as a duration such as `10m`. Default `5m`. Not sent to the API.

### Read-Only

- `id` (String) `<service_id>/<credential_id>`.
- `credential_id` (String) The credential id (a UUID) assigned by the API.
- `created_at` (String) Creation timestamp.
- `expires_at` (String) Expiry timestamp; null for `ttl = "never"`.
- `status` (String) `active`, or `revoking` (see Refresh).
- `kubeconfig` (String, Sensitive) The kubeconfig YAML. Stored in state.

## Import

Import by `<service_id>/<credential_id>`. The YAML is fetched with `GET /v1/services/{id}/kubeconfigs/{credential_id}/download`, which refuses a revoked or expired credential.

```shell
terraform import rarecloud_kubernetes_kubeconfig.ci k8s-1a2b3c4d/0b0e6f3a-5c1d-4e2f-9a8b-7c6d5e4f3a2b
```
