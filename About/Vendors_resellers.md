---
layout: default
title: Heads Vendors and Resellers
permalink: /Vendors/
nav_order: 1
parent: About
---

# Heads Vendors and Resellers

Several vendors and resellers offer Heads preinstalled. Other hardware vendors
maintain coreboot platforms and contribute Heads ports without yet offering
Heads as a factory option. The availability stated for each vendor is therefore
important: an upstream port does not imply that Heads can be selected when
ordering a machine.

The vendors are listed alphabetically.

## Heads preinstalled

### HardenedVault (VaultBoot)
HardenedVault provides VaultBoot (a variant of Heads) preinstalled on their
devices.

- **Website:** [HardenedVault](https://hardenedvault.net)
- **Products:** Servers

### Nitrokey (Heads)
Nitrokey offers Heads preinstalled on some of their devices. They also sell
older refurbished laptop models with Intel ME neutralized/deactivated and Nitrokey USB
security dongles. Additionally, Nitrokey resells some of NovaCustom's laptops.

- **Website:** [Nitrokey](https://www.nitrokey.com)
- **Products:** Laptops, phones, servers, workstations, mini-PCs, USB security
  dongles, and older refurbished laptop models with ME neutralized/deactivated

### NovaCustom (Heads)
NovaCustom offers devices with Heads preinstalled. They focus on providing
customizable and secure devices for their customers. NovaCustom buys Clevo
laptops in bulk, ensuring BootGuard keys are not fused at the last manufacturing
steps. They also resell Nitrokey 3 USB security dongles bundled with their
Heads-based firmware devices. They provide recent laptop models with ME deactivated.

- **Website:** [NovaCustom](https://novacustom.com)
- **Products:** Laptops and USB security dongles

### Purism (PureBoot Heads distribution)
Purism offers laptops, tablets, mini PCs, and servers with PureBoot (a
distribution of Heads) preinstalled. BootGuard is unfused to ensure firmware
remains user-controlled. They provide recent laptop models with ME deactivaated.
Purism makes and sells the Librem Key, which is a clone of the Nitrokey Pro 2. 
The Librem Key is made in the USA.

- **Website:** [Purism](https://puri.sm)
- **Products:** Laptops, phones, tablets, mini PCs, servers, and USB security
  dongles

## Vendor-maintained ports

### Star Labs

Star Labs develops Linux laptops, tablets and mini PCs with coreboot firmware.
The StarLite Mk V Heads port is
[under upstream review](https://github.com/linuxboot/heads/pull/2164). Heads is
not currently offered preinstalled; users must build and flash it themselves.

- **Website:** [Star Labs](https://starlabs.systems)
- **Products:** Laptops, tablets and mini PCs

## General Information

Many of these vendors offer additional services and features, including:

- **OS Preinstallation Options:** Vendors may offer preinstallation of various
  operating systems, including QubesOS, PureOS, and others.
- **Anti-Interdiction Mechanisms:** Vendors provide anti-interdiction services
  to ensure the security and integrity of the devices during shipping.
- **QubesOS Certification:** Some devices may be QubesOS certified, ensuring
  compatibility and security.
- **CSME/ME Status:** Some vendors offer options to neutralize or disable Intel
  CSME/ME. "Neutralized" means most parts of the ME are removed, while
  "disabled" means the ME is deactivated. Users should verify these options on
  the respective vendor websites. For more information, refer to Purism's blog
  post on this topic: [Deep Dive into Intel ME Disablement](https://puri.sm/posts/deep-dive-into-intel-me-disablement/).
- **Blob Status:** The newer the platform, the more it relies on proprietary
  blobs. Users should consider their threat model when choosing a device. For
  more information, refer to the [threat modeling page](/Heads-threat-model/).
- **HOTP Security Dongles:** Purism and Nitrokey are makers of HOTP-compatible
  security dongles. USB security dongles are used for both remote attestation
  and to authenticate and sign boot content. Heads relies on HOTP for tamper
  evidence. Users should verify the specific offerings on the respective vendor
  websites.

Please verify the specific offerings and services on the respective vendor
websites.
