---
page_title: "rarecloud_kubernetes_kubeconfig_admin Data Source - terraform-provider-rarecloud"
description: |-
  The short-lived admin kubeconfig of a managed Kubernetes cluster.
---

# rarecloud_kubernetes_kubeconfig_admin (Data Source)

The short-lived admin (cluster-admin) kubeconfig of a [`rarecloud_kubernetes_cluster`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/resources/kubernetes_cluster), from `GET /v1/services/{id}/kubeconfig`. Each read issues a new one, and the API records every issue in the account's audit log.

~> **Security:** a data source is written to state in plain text on every plan and refresh (`Sensitive` only hides it from output). The kubeconfig expires on its own (`expires_at`), but for admin access prefer the [ephemeral `rarecloud_kubernetes_kubeconfig`](https://registry.terraform.io/providers/RareCloudio/rarecloud/latest/docs/ephemeral-resources/kubernetes_kubeconfig), which keeps the credential out of state.

## Example Usage

```terraform
data "rarecloud_kubernetes_kubeconfig_admin" "prod" {
  service_id = rarecloud_kubernetes_cluster.prod.id
}
```

## Schema

### Required

- `service_id` (String) The `rarecloud_kubernetes_cluster` id.

### Read-Only

- `kubeconfig` (String, Sensitive) The admin kubeconfig YAML. Stored in state.
- `expires_at` (String) When the kubeconfig's credential expires.
