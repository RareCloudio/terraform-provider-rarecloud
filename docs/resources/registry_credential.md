---
page_title: "rarecloud_registry_credential Resource - terraform-provider-rarecloud"
description: |-
  A named robot credential for docker login / pull / push against the container registry.
---

# rarecloud_registry_credential (Resource)

A named robot credential for `docker login` / pull / push against the container registry (`POST /v1/registry/credentials`).

Every attribute is immutable: changing `name`, `scope`, or `expires_at` revokes this credential and mints a new one (there is no update path). Destroying this resource revokes it immediately.

## WARNING: the secret is stored in Terraform state in plain text

`secret` is stored in Terraform state **in plain text, even though it is marked sensitive**. Marking an attribute `Sensitive` only redacts it from CLI output and logs, it does NOT encrypt state. The API shows the secret exactly once, in the create response, and can never return it again, so state is the only place Terraform can keep it.

Protect your state file accordingly: encryption at rest, a remote backend with encryption, and restricted access. If state is lost or the secret leaks, destroy this resource (which revokes the credential) and create a new one, there is no rotate-in-place.

**Import is not supported**, for the same reason: an existing credential's secret cannot be recovered to populate state.

## Example Usage

```terraform
resource "rarecloud_registry_credential" "ci" {
  name  = "ci"
  scope = "push"
}

resource "rarecloud_registry_credential" "readonly_bot" {
  name       = "readonly-bot"
  scope      = "pull"
  expires_at = "2027-01-01T00:00:00Z"
}

output "ci_login" {
  value     = "docker login registry.rarestack.net -u ${rarecloud_registry_credential.ci.username} -p ${rarecloud_registry_credential.ci.secret}"
  sensitive = true
}
```

## Schema

### Required

- `name` (String) Your label for the credential; also half of the login username, `<handle>+<name>`. Pattern `^[a-z0-9][a-z0-9-]{1,30}$`. Immutable.
- `scope` (String) `pull` or `push` (push implies pull), whole-namespace. Immutable.

### Optional

- `expires_at` (String) RFC3339 expiry. Omit for a credential that never expires. Immutable.

### Read-Only

- `id` (String) The credential id (UUID).
- `username` (String) The `docker login` / HTTP Basic username: `<handle>+<name>`.
- `secret` (String, Sensitive) The `docker login` / HTTP Basic password. Shown by the API exactly once, at creation; this is the only copy Terraform will ever have. See the warning above.
- `created_at` (String) ISO-8601 creation timestamp.
- `last_used_at` (String) The last time this credential authenticated to the registry token endpoint, or null if it never has.
- `revoked_at` (String) When this credential was revoked, or null while it is active.
