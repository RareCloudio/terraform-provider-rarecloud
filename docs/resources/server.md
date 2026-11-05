---
page_title: "rarecloud_server Resource - terraform-provider-rarecloud"
description: |-
  A cloud compute VM (`POST /v1/services`, category `cloud-vm`), billed hourly.
---

# rarecloud_server (Resource)

A cloud compute VM (`POST /v1/services`, category `cloud-vm`), billed hourly.

Changing `plan` resizes the VM in place (`POST /v1/services/{id}/resize`, the VM reboots). Changing `tags` updates them in place. Changing `name` does not rename the VM: no API call is made, the next refresh reads the old name back and the plan shows the change again, so set it at creation. Changing `region`, `image`, `ssh_public_key`, `root_password`, `user_data` or `vpc_id` destroys and recreates the VM.

## Example Usage

```terraform
resource "rarecloud_server" "web" {
  name           = "web"
  plan           = "g-2vcpu-8gb"
  region         = "bucharest-ro"
  image          = "ubuntu-24.04"
  ssh_public_key = file("~/.ssh/id_ed25519.pub")
  tags           = ["web"]
}
```

## User data

`user_data` passes the VM a cloud-config or script to run at first boot:

```terraform
resource "rarecloud_server" "web" {
  name           = "web"
  plan           = "g-2vcpu-8gb"
  region         = "bucharest-ro"
  image          = "ubuntu-24.04"
  ssh_public_key = file("~/.ssh/id_ed25519.pub")
  user_data      = file("${path.module}/boot.yaml")
}
```

- The first line must start with `#cloud-config` or `#!`, and the limit is 32 KiB measured as UTF-8. The API refuses anything else, and the apply shows its message. The contents are not checked, so invalid YAML does not fail the apply.
- Any process inside the VM can read the value, so do not put secrets in it.
- **The Terraform state file holds the value as plain text.** `Sensitive` hides it from plan and apply output and nothing more, as with `root_password`. Protect the state file accordingly.
- The VM is reported active without waiting for your script to finish.
- On a snapshot it runs only if the snapshot has cloud-init: run `cloud-init clean` before you take the snapshot. Otherwise nothing runs and the API reports no error.
- The API does not store or return the value, so a refresh keeps what is in state, and a change made outside Terraform is not seen.

## Import

```shell
terraform import rarecloud_server.web <server id>
```

The import reads `name`, `plan` (the flavor the VM runs), `tags`, `ipv4`, `ipv6`, `private_ip`, `hostname`, `status`, `api_access`.

Not imported (null after the import):

- `image`, `ssh_public_key`, `root_password`, `user_data`: boot-time inputs the API does not return.
- `region`: the API reports an internal region name, not the region slug the provider takes (`bucharest-ro`).
- `vpc_id`: not returned by the API.
- `plan`, only when the API cannot name the VM's flavor. The configured plan is then recorded, never sent as a resize.

Set them in the config as they were at creation. The first plan after the import shows them as an in-place update that only records the value in state; it never replaces the resource. From the first apply on, a change replaces the resource as before. To skip that first update, use `lifecycle { ignore_changes = [...] }` for the attributes that differ.

## Schema

### Required

- `name` (String) Display name, set at creation. A change does not rename the VM (see above).
- `plan` (String) Public catalog SKU (e.g. `g-2vcpu-8gb`). A change resizes in place.
- `region` (String) Region slug. A change recreates the VM.
- `image` (String) OS image slug. A change recreates the VM.

### Optional

- `ssh_public_key` (String) OpenSSH public key injected at boot. A change recreates the VM.
- `root_password` (String, Sensitive) Initial root password. A change recreates the VM.
- `user_data` (String, Sensitive) Cloud-config or script run at first boot, at most 32 KiB. Stored as plain text in the state file (see above). A change recreates the VM.
- `vpc_id` (String) `rarecloud_network.id` to launch into. A change recreates the VM.
- `tags` (List of String) Tags, updated in place. At most 50, each 1 to 60 characters without `,` or `/`; tags starting with `managed:`, `k8s:` or `rarecloud` are reserved. Order does not matter. When unset, the VM's tags are read but not managed.

### Read-Only

- `id` (String) Server id.
- `ipv4`, `ipv6`, `private_ip`, `hostname`, `status` (String)
- `api_access` (String) `full` or `read_only`.
- `image_name` (String) Name of the image the VM was installed from (for example `Debian 12`). It follows a reinstall made outside Terraform. Null when the API cannot determine it.
