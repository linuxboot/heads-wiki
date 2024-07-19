---
layout: default
title: Distributions
permalink: /Distributions/
nav_order: 4
parent: Development
---

Distributions
===

Distributions can brand and customize Heads to suit their needs.  Examples of
distributions are:

* [Nitrokey](https://github.com/Nitrokey/heads/)
* [PureBoot](https://source.puri.sm/firmware/pureboot)
* [Dasharo](https://github.com/Dasharo/heads)

Configuration
===

Set brand-specific configuration in `site-local/config`.  For example, to use
text-based whiptail on all boards and export a brand-specific configuration value:

```sh
# Export a brand-specific configuration value
export CONFIG_ACME_HELLO=welcome
# Use text-based whiptail on all devices (enable it and disable fbwhiptail)
CONFIG_SLANG=y
CONFIG_NEWT=y
CONFIG_CAIRO=n
CONFIG_FBWHIPTAIL=n
```

The `site-local` directory is for downstream files included at strategic points
in Heads to simplify carrying patches downstream.  The `site-local` directory
never appears upstream.

Branding
===

The `BRAND_NAME` variable determines the brand used in builds.  The value:

* is used in the user interface ("AcmeBoot Boot Menu")
* is used in ROM build artifacts (`acmeboot-<board>-<version>.rom`)
* names the directory for branded assets (bootsplash)

Add `BRAND_NAME` to `site-local/config`:

```sh
BRAND_NAME=AcmeBoot
```

Create a bootsplash.jpg file for bootsplash at `$BRAND_NAME/bootsplash.jpg`,
for example `AcmeBoot/bootsplash.jpg`.
