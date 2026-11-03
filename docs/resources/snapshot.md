---
page_title: "rarecloud_snapshot Resource - terraform-provider-rarecloud"
description: |-
  A snapshot of a cloud VM's root disk.
---

# rarecloud_snapshot (Resource)

A point-in-time copy of a cloud VM's root disk (`POST /v1/services/{server_id}/snapshots`, `GET` and `DELETE /v1/snapshots/{id}`). Snapshots are a cloud-VM feature only, up to 5 per VM, billed by the storage they occupy (`used_bytes`). The VM keeps running while the snapshot is taken.

Create sends one `Idempotency-Key` across its retries, so a lost answer never produces a second snapshot. It then waits until the snapshot is `ready` (fully independent of its source disk), for up to 30 minutes. A snapshot that reaches `failed` fails the apply with an error naming the snapshot id; it stays in state as tainted, so the next apply replaces it. A snapshot that is not `ready` within 30 minutes is not a failure: the apply succeeds with a warning naming the snapshot id, the snapshot stays in state (not tainted) and keeps being prepared, and the next plan or apply reads its state. Interrupting the wait (Ctrl-C) is handled the same way, with a warning that says the wait was interrupted. If the API refuses the token (HTTP 401 or 403) while the apply waits, the wait stops at once and is handled the same way too: the snapshot was created, so it stays in state untainted, and the warning says the token could not read it back (check its `services:read` scope). If the snapshot is deleted while the apply waits, the wait stops at once with an error. A refresh that finds the snapshot `failed` warns with its id: it still uses one of the VM's snapshot slots until it is deleted.

The API cannot rename a snapshot, so changing `name` or `server_id` creates a new one. The API trims names, so a `name` with leading or trailing whitespace is refused at plan time.

In the API a snapshot outlives the VM it was taken from: deleting the VM does not delete its snapshots. In Terraform, though, `server_id` forces a replacement, so when the referenced `rarecloud_server` is replaced (any change that forces a new server id), Terraform also destroys this snapshot and takes a new one of the new server. To keep a snapshot across a server rebuild, add `lifecycle { ignore_changes = [server_id] }` or set `server_id` to a literal id. A refresh drops the resource from state when the snapshot is gone.

Destroying it deletes the snapshot. The API refuses while VMs or volumes created from the snapshot still exist; the error names them and is shown as the API sent it. Delete those first.

**Restore is not part of this resource.** Restoring a VM from a snapshot replaces the VM's disk and loses everything written since, so it stays an explicit action: use the `rarecloud` CLI or the MCP server.

## Example Usage

```terraform
resource "rarecloud_snapshot" "nightly" {
  server_id = rarecloud_server.web.id
  name      = "before-upgrade"
}
```

## Import

```shell
terraform import rarecloud_snapshot.nightly <snapshot id>
```

The import reads every attribute, `server_id` included (the snapshot's source VM).

## Schema

### Required

- `server_id` (String) `rarecloud_server.id` of the cloud VM to snapshot. Changing it creates a new snapshot.
- `name` (String) Snapshot name, 1 to 64 characters, without leading or trailing whitespace. Changing it creates a new snapshot.

### Read-Only

- `id` (String) Snapshot id.
- `state` (String) `creating`, `securing`, `ready` or `failed`.
- `used_bytes` (Number) Real storage occupied in bytes, the basis for billing. Null until the API has measured it.
- `min_disk_gb` (Number) Smallest disk, in GB, a VM can have to boot from this snapshot.
- `created_at` (String) Creation time (RFC 3339).
