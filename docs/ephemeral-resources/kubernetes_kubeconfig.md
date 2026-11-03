---
page_title: "rarecloud_kubernetes_kubeconfig Ephemeral Resource - terraform-provider-rarecloud"
description: |-
  A kubeconfig for a managed Kubernetes cluster that is never written to plan or state.
---

# rarecloud_kubernetes_kubeconfig (Ephemeral Resource)

A kubeconfig for a [`rarecloud_kubernetes_cluster`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/kubernetes_cluster) that exists only during the current Terraform run and is **never written to plan or state**.

-> **Requires Terraform 1.10 or later.** Ephemeral resources are a Terraform 1.10 feature; older versions reject the `ephemeral` block. The rest of the provider keeps working on older versions.

Two modes:

- **Fetch** (`credential_id` given): downloads an existing credential (`GET /v1/services/{id}/kubeconfigs/{credential_id}/download`). It is left as it is at the end of the run.
- **Mint** (`name` and `role` given): creates a new credential (`POST /v1/services/{id}/kubeconfigs`) and revokes it at the end of the run (`DELETE /v1/services/{id}/kubeconfigs/{credential_id}`). When the API answers `revoking` (the cluster could not be reached; the credential is already refused by the API and its ServiceAccount is deleted later), the end of the run waits up to `revoke_timeout` (default `5m`) for `revoked`, and otherwise ends with a warning naming the credential. A failed revoke is a warning too.

Terraform does not guarantee the end-of-run step (a crash or an interrupted run skips it), so a minted credential always expires: `ttl` defaults to `30d` and `never` is not accepted. Minting happens on every plan and apply that opens the resource, and the API limits how fast credentials can be created.

## Example Usage

```terraform
ephemeral "rarecloud_kubernetes_kubeconfig" "deploy" {
  service_id = rarecloud_kubernetes_cluster.prod.id
  name       = "terraform-run"
  role       = "admin"
  ttl        = "30d"
}

# Locals and provider blocks accept ephemeral values (Terraform 1.10+).
locals {
  kubeconfig = yamldecode(ephemeral.rarecloud_kubernetes_kubeconfig.deploy.kubeconfig)
}

provider "kubernetes" {
  host                   = local.kubeconfig.clusters[0].cluster.server
  cluster_ca_certificate = base64decode(local.kubeconfig.clusters[0].cluster["certificate-authority-data"])
  token                  = local.kubeconfig.users[0].user.token
}
```

## Schema

### Required

- `service_id` (String) The `rarecloud_kubernetes_cluster` id.

### Optional

- `credential_id` (String) Fetch this existing credential. Leave it out to mint a new one; it then reports the minted credential's id.
- `name` (String) Name of the minted credential, 1 to 64 characters. Required to mint.
- `role` (String) `admin` or `view` for the minted credential. Required to mint.
- `ttl` (String) Lifetime of the minted credential: `30d` (default), `90d` or `1y`.
- `revoke_timeout` (String) How long the end of the run waits for a `revoking` credential to become `revoked`, as a duration such as `2m`. Default `5m`. Not sent to the API.

### Read-Only

- `kubeconfig` (String, Sensitive) The kubeconfig YAML. Not stored in plan or state.
- `expires_at` (String) Expiry timestamp; null for a credential that never expires.
