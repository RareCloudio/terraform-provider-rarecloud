---
page_title: "rarecloud_billing_alert Resource - terraform-provider-rarecloud"
description: |-
  The account's billing alert: a spend threshold, a runway threshold in days, or both.
---

# rarecloud_billing_alert (Resource)

The account's billing alert (`/v1/billing/alert`): an email when the month-to-date cloud spend reaches `threshold_cents`, when the credit has fewer than `threshold_days` days left, or both.

There is one alert per account, so declare this resource once. Creating it fails if the account already has an alert (for example one set in the console); import that alert instead (`terraform import rarecloud_billing_alert.<name> billing_alert`). A threshold left out of the config is cleared, so the config is the whole alert. Destroying the resource removes the alert. A refresh reads the alert back (changes made in the console show as drift) and drops the resource from state when no alert is configured.

## Example Usage

```terraform
resource "rarecloud_billing_alert" "this" {
  threshold_days  = 7    # credit lasts fewer than 7 days
  threshold_cents = 5000 # month-to-date spend reaches EUR 50
}
```

## Import

```shell
terraform import rarecloud_billing_alert.this billing_alert
```

The id is always `billing_alert`; any other id is refused. The import reads every attribute: a config matching the stored alert plans no change after the import.

## Schema

At least one of `threshold_cents` and `threshold_days` must be set (checked at plan time, as the API requires).

### Optional

- `threshold_cents` (Number) Month-to-date spend threshold in EUR cents, 100 or more.
- `threshold_days` (Number) Runway threshold in days, 1 to 60.
- `enabled` (Boolean) Whether the alert sends emails. Default `true`; `false` keeps the thresholds and mutes the alert.

### Read-Only

- `id` (String) Always `billing_alert`.
