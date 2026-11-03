---
page_title: "rarecloud_image Data Source - terraform-provider-rarecloud"
description: |-
  Looks up a catalog OS image by slug.
---

# rarecloud_image (Data Source)

Looks up an OS image in the catalog by `slug` (`GET /v1/catalog/images`). The slug is the value `rarecloud_server.image` takes. A slug the catalog does not list is an error, so this checks an image choice at plan time.

## Example Usage

```terraform
data "rarecloud_image" "ubuntu" {
  slug = "ubuntu-24.04"
}

resource "rarecloud_server" "web" {
  name   = "web-01"
  plan   = "g-2vcpu-8gb"
  region = "bucharest-ro"
  image  = data.rarecloud_image.ubuntu.slug
}
```

## Schema

### Required

- `slug` (String) Image slug, for example `ubuntu-24.04`.

### Read-Only

- `display_name` (String) Human-readable name, for example `Ubuntu 24.04 LTS`.
- `os_family` (String) Operating system family, for example `ubuntu`.
- `version` (String) Version, for example `24.04`.
- `active` (Boolean) Whether the image is active in the catalog.
- `available_backends` (List of String) Backends the image can be installed on, for example `openstack`.
