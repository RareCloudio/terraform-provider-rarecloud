---
page_title: "rarecloud_sandbox Resource - terraform-provider-rarecloud"
description: |-
  An AI code sandbox: an ephemeral, isolated execution environment.
---

# rarecloud_sandbox (Resource)

An AI code sandbox (`/v1/sandboxes`): an ephemeral, isolated execution environment, billed per second of runtime.

Sandboxes are immutable: changing `template`, `size`, `image`, `timeout_ms`, `run`, `vcpu` or `ram_gb` recreates the sandbox. A sandbox stops and is deleted automatically at `expires_at`. A refresh then drops it from state, and the next apply creates a new one. Destroying a sandbox that is already gone succeeds.

Run code inside it with the `rarecloud` CLI or the API, not with Terraform.

A size or custom shape the cluster cannot run yet is refused with `409 CONFLICT` (`size_unavailable`). `GET /v1/sandboxes/sizes` lists what is available. Creating more sandboxes than the account's limit is refused too; see `sandboxes` on the `rarecloud_limits` data source.

## Example Usage

```terraform
resource "rarecloud_sandbox" "ci" {
  template   = "python"
  timeout_ms = 300000 # 5 minutes
}

# Your own image, starting its own program and ending when it exits.
resource "rarecloud_sandbox" "job" {
  template = "base"
  image    = "ghcr.io/acme/job:1.0"
  run      = {}
}

# A template needs a command in run mode.
resource "rarecloud_sandbox" "script" {
  template = "python"
  run = {
    command = ["python3", "-c", "print('done')"]
  }
}
```

## Import

Import is not supported.

## Schema

### Optional

- `template` (String) `python`, `node` or `base`. Default `python`. A change recreates the sandbox.
- `size` (String) `xs`, `s`, `m`, `l`, `xl` or `2xl`, independent of the template. Unset, the template's default size applies (`xs` for `base`, `s` for `python` and `node`) and is read back. Cannot be set with `vcpu` or `ram_gb`. A change recreates the sandbox.
- `image` (String) An image to run instead of the template's: a public image (for example `nginx:1.27`) from Docker Hub, `ghcr.io`, `quay.io`, `registry.k8s.io`, `public.ecr.aws`, `gcr.io` or `mcr.microsoft.com`, with no port, or a private image from your own RareCloud registry. Any other registry is refused. A change recreates the sandbox.
- `timeout_ms` (Number) Lifetime in milliseconds. Default 300000, at most 3600000. A value from 1000 to 59999 is raised to 60000 by the API; a smaller one is refused. A change recreates the sandbox.
- `run` (Attributes) Run mode: the sandbox starts a program by itself and ends when it exits (`status` becomes `exited`, see `exit_code`). `run = {}` runs the image's own program; a template needs a non-empty `command`. An exited sandbox disappears 24 hours after it ended, and the next plan recreates it. A change recreates the sandbox. See [below](#nested-schema-for-run).
- `vcpu` (Number) Custom resources: vCPU count, 1 to 8. Set together with `ram_gb`, never with `size`. Priced by the same formula as the sizes: EUR 0.040 per vCPU-hour plus EUR 0.012 per GB-hour. Unset, state reads back the vCPU the sandbox runs with. A change recreates the sandbox.
- `ram_gb` (Number) Custom resources: RAM in GB, 1 to 32 and between 1x and 8x `vcpu`. Set together with `vcpu`. Unset, state reads back the RAM the sandbox runs with. A change recreates the sandbox.

### Read-Only

- `id` (String) Sandbox id.
- `status` (String) `pending`, `running`, `terminated` or `exited` (run mode, the program ended).
- `exit_code` (Number) The program's exit code once a run-mode sandbox has exited. Null otherwise.
- `created_at` (String) ISO-8601 creation timestamp.
- `expires_at` (String) ISO-8601 time at which the sandbox stops and is deleted, computed by the API from `timeout_ms` at creation. Billing stops then at the latest.

<a id="nested-schema-for-run"></a>
### Nested Schema for `run`

Optional:

- `entrypoint` (List of String) Replaces the image's ENTRYPOINT, and its CMD unless `command` is set. An empty list counts as unset.
- `command` (List of String) Replaces the image's CMD; the entrypoint still runs. At most 64 elements, 4096 bytes each, 32 KiB in total. An empty list counts as unset.
