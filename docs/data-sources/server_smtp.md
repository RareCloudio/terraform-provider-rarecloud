---
page_title: "rarecloud_server_smtp Data Source - terraform-provider-rarecloud"
description: |-
  Outbound email status of a cloud VM.
---

# rarecloud_server_smtp (Data Source)

Outbound email status of a cloud VM (`GET /v1/services/{id}/smtp`). A new cloud VM can have outbound TCP port 25 blocked until an unblock request is approved; `status` says whether this VM is. Ports 465 and 587 stay open.

Terraform does not request the unblock: use the VM's Networking tab in the console, the `rarecloud server smtp` commands, or `POST /v1/services/{id}/smtp/unblock-request`. After a rejection or a revoked approval, a new request is accepted after 7 days.

## Example Usage

```terraform
data "rarecloud_server_smtp" "mail" {
  server_id = "00000000-0000-0000-0000-000000000000"
}

output "smtp_status" {
  value = data.rarecloud_server_smtp.mail.status
}
```

## Schema

### Required

- `server_id` (String) ID of the cloud VM.

### Read-Only

- `status` (String) `open` (not restricted), `blocked`, `pending`, `approved` or `rejected`.
- `request_id` (String) ID of the latest unblock request. Null when there is none.
- `can_request_after` (String) After a rejection or a revoked approval, when a new request is accepted again (RFC 3339). Null otherwise.
