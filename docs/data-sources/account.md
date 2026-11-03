---
page_title: "rarecloud_account Data Source - terraform-provider-rarecloud"
description: |-
  Reads the authenticated account's profile and its cloud billing state.
---

# rarecloud_account (Data Source)

Reads the account the provider's token belongs to: the profile (`GET /v1/account`) and the cloud billing state (`GET /v1/billing/state`) in one data source.

Takes no input; it always describes the caller's own account. Use it to name or tag resources after the operator, to pick a currency, or to gate a plan on what the account owes (`open_invoice_id`) and on the deadline that sets (`billing_power_off_at`). `billing_state` reports the outcome, after the fact.

The balance figures are INFORMATION. Nothing there enforces anything: a runway reaching zero suspends nothing. What does enforce is the invoice. An account's cloud resources are powered off when an invoice we issued for its cloud usage is still unpaid at the end of its payment window, and paying it, or credit arriving, brings everything back. `open_invoice_due_at` is where that window OPENS rather than a deadline: the invoice is due the day it is issued and the account then has 7 days to pay. `billing_power_off_at` is the real deadline, about a week later, and it is the attribute to gate on: `billing_state` only flips once the power-off has already happened, and the resources are deleted 14 days after it.

The `billing_*` and `open_invoice_*` attributes are read from a second API call and are **fail-soft**: if that call fails, the data source still returns the profile, emits a warning, and leaves all of them null. They are also null when the billing backend itself reports "unknown" (which is not the same as zero), `billing_runway_hours` is null whenever there is no hourly burn or no balance to divide, and `billing_power_off_at` together with every `open_invoice_*` attribute is null on an account that owes nothing. A token without the `billing:read` scope cannot read any of them: expect nulls and one warning per plan.

## Example Usage

```terraform
data "rarecloud_account" "me" {}

output "account_email" {
  value = data.rarecloud_account.me.email
}

output "days_of_runway" {
  # Null while nothing is burning credit, or while the billing state is unknown.
  value = try(floor(data.rarecloud_account.me.billing_runway_hours / 24), null)
}

output "owes_us" {
  # Null means the account owes nothing. A number is an invoice number: left
  # unpaid, this account's cloud resources are powered off at
  # billing_power_off_at, NOT on open_invoice_due_at, which is only where the
  # 7 day payment window opens.
  value = data.rarecloud_account.me.open_invoice_id
}

output "power_off_deadline" {
  # The real deadline to gate on: the instant those resources stop. Null while
  # the account owes nothing.
  value = data.rarecloud_account.me.billing_power_off_at
}
```

## Schema

### Read-Only

- `id` (String) The account id.
- `email` (String) The account's email address.
- `first_name` (String) Given name on the account.
- `last_name` (String) Family name on the account.
- `country` (String) Two-letter country code on the account.
- `preferred_currency` (String) The account's billing currency, e.g. `EUR`. Also the currency of `billing_available_cents`.
- `email_verified` (Boolean) Whether the email address has been verified.
- `billing_state` (String) Cloud billing state: `ok` or `suspended`. An account is suspended once an invoice we issued for its cloud usage has run out its 7 day payment window; paying it brings everything back. Null when the billing state could not be read.
- `billing_available_cents` (Number) Spendable balance in `preferred_currency` cents. Null when the billing backend could not be read, which is not the same as zero.
- `billing_runway_hours` (Number) How many hours that balance lasts at the account's current hourly burn. Informational: a runway reaching zero suspends nothing, an unpaid cloud invoice does. Null when there is no burn or no balance to divide.
- `billing_power_off_at` (String) RFC 3339 instant the account's cloud resources are actually powered off because `open_invoice_id` is unpaid. The invoice is due the day it is issued and the account then has 7 days to pay, so this lands roughly a week after `open_invoice_due_at`, held to the account's own 09:00 to 20:00 local window. THIS is the attribute to gate a plan on if you want the real deadline: `open_invoice_due_at` is only the start of the payment window, and `billing_state` only flips after the power-off has already happened. Null when nothing of the account's is unpaid, or when the billing state could not be read.
- `open_invoice_id` (Number) Invoice number of the oldest unpaid invoice we issued for this account's cloud usage. This is the attribute to test for existence, and `billing_power_off_at` is the deadline to gate on: both are readable the moment the invoice is raised, while `billing_state` only flips after the power-off has already happened. Null when the account owes nothing, or when the billing state could not be read. A renewal for a VPS, proxy, hosting or domain is never reported here.
- `open_invoice_amount_cents` (Number) Amount of that invoice, in `open_invoice_currency` cents. Null when there is no open invoice, or when the amount could not be read, which is not the same as zero.
- `open_invoice_currency` (String) Currency the invoice is denominated in, e.g. `EUR`. Read from the invoice itself rather than assumed from `preferred_currency`. Null when there is no open invoice or the currency could not be read.
- `open_invoice_due_at` (String) RFC 3339 instant of the day that invoice is due, which is the day it was issued. It is the START of the payment window rather than a deadline: the account has 7 days from it to pay, and nothing happens on the day itself. `billing_power_off_at` is the instant the cloud resources are actually powered off, and they are deleted 14 days after that. Null when there is no open invoice.
