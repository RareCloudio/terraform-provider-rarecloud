---
page_title: "rarecloud_registry_kubernetes_manifest Resource - terraform-provider-rarecloud"
description: |-
  A ready-to-apply Kubernetes pull-secret manifest for a cluster RareCloud does not manage (BYO).
---

# rarecloud_registry_kubernetes_manifest (Resource)

A ready-to-apply Kubernetes pull-secret manifest for a cluster RareCloud does NOT manage (BYO: your own kind/EKS/GKE/bare-metal cluster). For a cluster RareCloud manages, use [`rarecloud_registry_cluster_link`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/registry_cluster_link) instead, which installs and maintains the secret itself.

Creating this resource calls `POST /v1/registry/docker-credentials/kubernetes`, which **mints a new registry credential every time**, counted against the account's credential cap. Every input is therefore immutable: changing any of them replaces the resource, minting a fresh credential and revoking the old one. Destroying this resource revokes the credential.

## Why this is a resource, not a data source

The underlying endpoint mints a brand-new credential (and secret) on every single call, a side effect. Terraform re-reads every data source on every `plan`/refresh; a data source wrapping this endpoint would mint (and immediately orphan) a new credential on every routine `terraform plan`, eventually failing outright once the account's credential cap is hit, and leaking live, unseen secrets. Modeled as a resource instead: `Create` mints the manifest once, `Read` re-validates the underlying credential still exists (without re-minting), `Delete` revokes it.

## WARNING: manifest and secret are stored in Terraform state in plain text

`manifest` and `secret` are stored in Terraform state **in plain text, even though both are marked sensitive** (`manifest` embeds the same secret, base64-encoded, as a `dockerconfigjson`). Marking an attribute `Sensitive` only redacts it from CLI output and logs, it does NOT encrypt state. The API shows both exactly once, in the create response, and can never return them again.

Protect your state file accordingly: encryption at rest, a remote backend with encryption, and restricted access. **Import is not supported**, for the same reason: an existing credential's manifest/secret cannot be recovered to populate state.

## Example Usage

```terraform
resource "rarecloud_registry_kubernetes_manifest" "byo" {
  scope      = "pull"
  expiry     = "90d"
  namespace  = "apps"
  secret_name = "rarecloud-registry"
}

output "pull_secret_manifest" {
  value     = rarecloud_registry_kubernetes_manifest.byo.manifest
  sensitive = true
}
```

Apply the manifest to your own cluster with `kubectl`, then reference it from a Pod's `imagePullSecrets`:

```shell
terraform output -raw pull_secret_manifest | kubectl apply -f -
```

## Schema

### Optional

All of the following are server-defaulted when omitted (see each description) and immutable: changing any of them replaces the resource.

- `scope` (String) `pull` or `push`. Defaults to `pull`.
- `expiry` (String) `30d`, `90d`, `1y`, or `never`. Defaults to `30d`.
- `namespace` (String) The namespace the manifest targets. Must be a valid DNS-1123 label. Defaults to `default`.
- `secret_name` (String) The Secret's name in the manifest. Must be a valid DNS-1123 label. Defaults to `rarecloud-registry`.

### Read-Only

- `id` (String) The credential id (UUID).
- `credential_name` (String) The credential's auto-generated name, e.g. `byo-a1b2c3d4`.
- `username` (String) The `docker login` / HTTP Basic username: `<handle>+<credential_name>`.
- `expires_at` (String) When the credential stops working, or null if `expiry` was `never`.
- `created_at` (String) ISO-8601 creation timestamp.
- `manifest` (String, Sensitive) The commented Kubernetes Secret manifest text (`type: kubernetes.io/dockerconfigjson`), ready for `kubectl apply -f`. See the warning above.
- `secret` (String, Sensitive) The raw credential secret backing the manifest. See the warning above.
