## [ACME Provider](README.md)

* [`acme_registration`](resource_acme_registration.md)
* [`acme_certificate`](resource_acme_certificate.md)

# ACME Certificate and Registration Provider

The Automated Certificate Management Environment (ACME) is an evolving standard
for the automation of a domain-validated certificate authority. Clients register
themselves on an authority using a private key and contact information, and
answer challenges for domains that they own by supplying response data issued by
the authority via either HTTP or DNS. Via this process, they prove that they own
the domains in question, and can then request certificates for them via the CA.
No part of this process requires user interaction, a traditional blocker in
obtaining a domain validated certificate.

Currently the major ACME CA is [Let's Encrypt][lets-encrypt], but the ACME
support in Terraform can be configured to use any ACME CA, including an internal
one that is set up using [Boulder][boulder-gh].

[lets-encrypt]: https://letsencrypt.org
[boulder-gh]: https://github.com/letsencrypt/boulder

For more detail on the ACME process, see [here][lets-encrypt-how-it-works]. For
the ACME spec, click [here][about-acme]. Note that the ACME provider may diverge
from the current ACME spec to account for the real-world divergences that are
made by CAs such as Let's Encrypt.

[lets-encrypt-how-it-works]: https://letsencrypt.org/how-it-works/
[about-acme]: https://ietf-wg-acme.github.io/acme/draft-ietf-acme-acme.html

:warning: **NOTE:** The ACME provider as of version 1.0.0 supports ACME v2 only.
For ACME v1 endpoints, version 0.6.0 is required, which can be found
[here][release-v0.6.0].

[release-v0.6.0]: https://github.com/vancluever/terraform-provider-acme/releases/tag/v0.6.0

## Installation Instructions

The ACME provider is currently a 3rd party plugin. See the documentation on [3rd
party plugins][3rd-party-plugins] for installation instructions, and download
the latest release from the [releases page][releases-page].

[3rd-party-plugins]: https://www.terraform.io/docs/configuration/providers.html#third-party-plugins
[releases-page]: https://github.com/vancluever/terraform-provider-acme/releases

### Distributions with direct installation support

If you use [Arch Linux][arch-linux], the
[`terraform-provider-acme-bin`][terraform-provider-acme-bin-arch] package is
available via the AUR and can be installed via an AUR-supported package manager
such as `yay`. Thanks to [**@SamWhited**][samwhited-gh] for this!

[arch-linux]: https://www.archlinux.org/
[terraform-provider-acme-bin-arch]: https://aur.archlinux.org/packages/terraform-provider-acme-bin/
[samwhited-gh]: https://github.com/SamWhited

Example with `yay`:

```
yay -S terraform-provider-acme-bin
```

## Basic Example

The following example can be used to create an account using the
[`acme_registration`](resource_acme_registration.md) resource, and a certificate
using the [`acme_certificate`](resource_acme_certificate.md) resource. The
initial private key is created using the
[`tls_private_key`][resource-tls-private-key] resource, but can be supplied via
other means. DNS validation is performed by using [Amazon Route 53][aws-route-53],
for which appropriate credentials are assumed to be in your environment.

[resource-tls-private-key]: https://www.terraform.io/docs/providers/tls/r/private_key.html
[aws-route-53]: https://aws.amazon.com/route53/

:warning: **NOTE:** The directory URLs in all examples in this provider
reference Let's Encrypt's staging server endpoint. For production use, change
the directory URLs to the production endpoints, which can be found
[here][lets-encrypt-endpoints].

[lets-encrypt-endpoints]: https://letsencrypt.org/docs/acme-protocol-updates/

```hcl
provider "acme" {
  server_url = "https://acme-staging-v02.api.letsencrypt.org/directory"
}

resource "tls_private_key" "private_key" {
  algorithm = "RSA"
}

resource "acme_registration" "reg" {
  account_key_pem = "${tls_private_key.private_key.private_key_pem}"
  email_address   = "nobody@example.com"
}

resource "acme_certificate" "certificate" {
  account_key_pem           = "${acme_registration.reg.account_key_pem}"
  common_name               = "www.example.com"
  subject_alternative_names = ["www2.example.com"]

  dns_challenge {
    provider = "route53"
  }
}
```

## Argument Reference

The following arguments are required:

* `server_url` - (Required) The URL to the ACME endpoint's directory.

:warning: Note that the account key is not a provider-level config value at this
time to allow the management of accounts and certificates within the same
provider.
