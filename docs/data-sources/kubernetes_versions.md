---
page_title: "rarecloud_kubernetes_versions Data Source - terraform-provider-rarecloud"
description: |-
  The Kubernetes versions a new managed cluster can run.
---

# rarecloud_kubernetes_versions (Data Source)

The Kubernetes versions a new managed cluster can run (`GET /v1/catalog/kubernetes-versions`, public), in the order the catalog lists them. Takes no input. An empty list (not an error) when the catalog offers none.

## Example Usage

```terraform
data "rarecloud_kubernetes_versions" "available" {}

output "kubernetes_versions" {
  value = data.rarecloud_kubernetes_versions.available.versions
}
```

## Schema

### Read-Only

- `versions` (List of String) Version strings, for example `1.35.6`. Empty when the catalog offers none.
