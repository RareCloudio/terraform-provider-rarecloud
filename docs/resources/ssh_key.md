---
page_title: "rarecloud_ssh_key Resource - terraform-provider-rarecloud"
description: |-
  An account-level SSH public key (`/v1/account/ssh-keys`).
---

# rarecloud_ssh_key (Resource)

An account-level SSH public key (`/v1/account/ssh-keys`).

Changing `name` or `public_key` recreates the key. The API stores the key trimmed, so a change in surrounding whitespace only (such as the trailing newline `file()` keeps) is recorded in place.

## Example Usage

```terraform
resource "rarecloud_ssh_key" "laptop" {
  name       = "laptop"
  public_key = file("~/.ssh/id_ed25519.pub")
}
```

## Import

```shell
terraform import rarecloud_ssh_key.laptop <ssh key id>
```

The import reads every attribute: `name`, `public_key`, `fingerprint`.

Nothing is left out: a config matching the real object plans no change after the import.

## Schema

### Required

- `name` (String) Display name. A change recreates the key.
- `public_key` (String) OpenSSH public key. A change recreates the key.

### Read-Only

- `id` (String)
- `fingerprint` (String)
