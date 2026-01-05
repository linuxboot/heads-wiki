---
layout: default
title: Heads Vendors and Resellers
permalink: /Vendors/
nav_order: 1
parent: About
---

# Heads Vendors and Resellers

For those who prefer not to manually flash Heads firmware on their devices,
several vendors and resellers offer laptops, workstations, and servers with
Heads preinstalled. These vendors provide a range of secure and privacy-focused
devices, making it easier for potential users to get started with Heads without
the hassle of searching for options online. By choosing one of these vendors or
resellers, users who don't want to open their devices and flash manually can
easily get a device with Heads preinstalled. This provides a straightforward
solution for those who would benefit from Heads' secure firmware but need a more
accessible option. Additionally, many of these vendors offer customization
options, preinstallation of various operating systems, and anti-interdiction
mechanisms to ensure the security and integrity of the devices.

The vendors are listed alphabetically.

## Vendors and Resellers

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
