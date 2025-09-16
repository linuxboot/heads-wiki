---
layout: default
title: Prerequisites for Heads
permalink: /Prerequisites
nav_order: 1
parent: Installing and configuring
---

<!-- markdownlint-disable MD033 -->
<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>
<!-- markdownlint-enable MD033 -->

Prerequisites for Heads
===

Required equipment
---

To install Heads on a physical device, you will need:

* Supported motherboard or laptop ([see below](#supported-devices))
* A heads compatible USB security dongle ([see below](#usb-security-dongles-aka-security-token-aka-smartcard))
* A heads compatible storage device for your public GPG key (USB flash drive)

If your device requires external flashing ([see below](#supported-devices)),
 you will also need:

* [SPI Programmer](https://trmm.net/SPI_flash): green pcb ch341a programmer or raspberry
 pi or bus pirate (green ch341a is recommended for new users and can be found almost
 [anywhere with preassembled clip as a kit](https://www.amazon.com/s?k=ch341a+programmer).
  * [Beware that black ch341a are known to not provide 3.3V as most NOR chips requires and should me manually fixed. If you intend to use a ch341a for initial flashing and occasional unbricking, this is perfectly fine without modification. Otherwise, you should look into other programmers or do the fix yourself. A lot of people suggest against using ch341a unless modified.](https://libreboot.org/docs/install/spi.html#do-not-use-ch341a)
* Wires and a clip SOIC8 to connect your programmer of choice to the board’s
 SPI flash chip(s).
  * The [Pomona 5250](https://www.pomonaelectronics.com/products/test-clips/soic-clip-8-pin)
   is suggested as it is high quality and easier to make contact with the pins.
* A second computer to flash from (Try to use a recommended operating system:
  Qubes or Debian 9 or Fedora 30)

Supported devices
---

**Please see the current [heads source](https://github.com/osresearch/heads/tree/master/boards) for up-to-date supported board configurations.**

*Note* repeatedly untested boards from willing to test board owners were moved to [unmaintained_boards directory and aren't built by CircleCI anymore](https://github.com/linuxboot/heads/tree/master/unmaintained_boards)

If *you have an external programmer* and *are techsavvy enough to bring their support back yourself*, read the [Community page](/community/) and reach out. I will gladly assist in your quest :)

USB Security Dongles (aka security token aka smartcard)
---

*NOTE* - Heads does **NOT** support FIDO2 or U2F authentication.  Be careful when
 purchasing to buy a compatible key.

 *NOTE* - HOTP remote attestation is supported from Librem/NovaCustom/Nitropad platforms by default, 
  Otherwise HOTP is explicitely supported by board configurations having `hotp` in their [board names](/Prerequisites#supported-devices).
  
*NOTE* - The NitroKey 3 comes in three sizes: USB A, A-mini and C. Nk3a mini (USB A-mini) is the one most shipped with novacustom and nitropads.
  - ThinkPads have USB A ports, not C. After that, it's users preferences for the form factor desired. 

|Manufacture|Model line|TOTP|HOTP|
|--|--|:--:|:--:|
|Yubico|[YubiKey 5 Series](https://www.yubico.com/products/yubikey-5-overview/)|X||
|Nitrokey|[Nitrokey Pro 2](https://www.nitrokey.com/products/nitrokeys#comparison)|X|X|
|Nitrokey|[Nitrokey Storage 2](https://www.nitrokey.com/products/nitrokeys#comparison)|X|X|
|Nitrokey|[Nitrokey 3](https://www.nitrokey.com/products/nitrokeys#comparison)|X|X|
|Purism|[Librem Key](https://puri.sm/products/librem-key/)|X|X|

*NOTE* - If you prefer not to use USB security dongles or want simplified security procedures, see the [Purism Boot Modes](/PurismBootModes) documentation for information about Basic and Restricted boot modes that provide different security/usability trade-offs.

Board Architecture Overview
===

**Note**: All current Heads boards use a modern architecture where the Intel Management Engine (ME) is deactivated and the Intel Flash Descriptor (IFD) is unlocked. On older Intel platforms (up to Ivy Bridge/3rd gen), the ME can be neutered (most modules removed), while on newer platforms (Skylake and later), the ME is deactivated using HAP bits or other methods. The historical distinction between "Legacy" and "Maximized" boards is no longer relevant as of 2024, since all supported boards now use the approach that was previously called "maximized."

For users upgrading from very old firmware (pre-2024), see the [Historical Legacy Migration](#historical-legacy-migration) section below.

---

### Historical Legacy Migration

<details>
<summary>Click to expand historical information about legacy vs maximized boards (for users on very old firmware only)</summary>

**This section is preserved for historical reference and applies only to users upgrading from very old Heads firmware (pre-2024). All new installations should use current board configurations.**

**Historical Background**

Originally, Heads supported two types of board configurations:

- **Legacy boards**: Produced incomplete ROM images that required specific flashing procedures and only worked with locked IFD/ME regions
- **Maximized boards**: Produced complete ROM images with neutered/deactivated ME and unlocked IFD, allowing full internal flashing

Legacy boards were [officially deprecated in October 2024](https://github.com/linuxboot/heads/pull/1803), and all current boards now use the approach that was previously called "maximized."

**Migration from Very Old Firmware**

If you are upgrading from very old Heads firmware that predates the 2024 standardization:

1. **Identify your current firmware type**: Check if your boot screen shows "Maximized" in the board name
2. **Verify upgrade compatibility**: Follow the [upgrade verification steps](/Updating#verify-upgradeability-paths-of-the-firmware) in the Upgrading documentation
3. **Contact support if needed**: For NitroPad users with firmware version 1.2 or earlier, contact [Nitrokey support](https://support.nitrokey.com/)

**Technical Details (Historical)**

The historical approach involved:
- **ME neutering/deactivation**: For older Intel platforms (up to Ivy Bridge), reducing the Intel Management Engine to only essential components (BUP and ROMP for xx30 family, BUP only for xx20). For newer platforms, ME is deactivated using HAP bits or other methods
- **IFD unlocking**: Allowing the Intel Flash Descriptor regions to be modified
- **Space optimization**: Reclaiming freed ME space for the BIOS region (expanding from ~7MB to ~11.5MB on x230 family)

This approach is now standard for all Heads boards, adapted to each platform's capabilities.

</details>

Emulated devices
===

For further information, see [Emulating Heads](/Emulating-Heads/)
