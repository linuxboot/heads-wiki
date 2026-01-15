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

**All USB Security dongles used with Heads must support OpenPGP** for storing your private key and signing `/boot` contents.

**HOTP verification is optional** but provides automatic firmware verification at boot. Without HOTP, you'll use TPMTOTP (manual verification with your phone). Most [board configurations](/Prerequisites#supported-devices) are available in both HOTP and non-HOTP variants, though some vendors only support HOTP-enabled configurations.

### USB Security dongle compatibility:

**Compatible dongles** must support the specialized HOTP verification protocol developed by Nitrokey. For technical details about this protocol, see the [Nitrokey HOTP verification project](https://github.com/Nitrokey/nitrokey-hotp-verification).

*NOTE* - Heads does **NOT** support FIDO2 or U2F authentication.  Be careful when
 purchasing to buy a compatible key.

 *NOTE* - HOTP remote attestation is supported from Librem/NovaCustom/Nitropad platforms by default, 
  Otherwise HOTP is explicitely supported by board configurations having `hotp` in their [board names](/Prerequisites#supported-devices).
  
*NOTE* - The NitroKey 3 comes in three sizes: USB A, A-mini and C. Nk3a mini (USB A-mini) is the one most shipped with novacustom and nitropads.
  - ThinkPads have USB A ports, not C. After that, it's users preferences for the form factor desired. 

### Supported USB Security dongles:

### Supported USB Security dongles:
|Manufacturer|Model|OpenPGP|HOTP verification|Compatible|
|--|--|:--:|:--:|:--:|
|Yubico|[YubiKey 5 Series](https://www.yubico.com/products/yubikey-5-overview/)|✅|❌|OpenPGP only|
|Nitrokey|[Nitrokey Pro 2](https://www.nitrokey.com/products/nitrokeys#comparison)|✅|✅|Full support|
|Nitrokey|[Nitrokey Storage 2](https://www.nitrokey.com/products/nitrokeys#comparison)|✅|✅|Full support|
|Nitrokey|[Nitrokey 3](https://www.nitrokey.com/products/nitrokeys#comparison)|✅|✅|Full support|
|Purism|[Librem Key](https://puri.sm/products/librem-key/)|✅|✅|Full support|

**Notes**:
- **OpenPGP only**: Can be used with non-HOTP board configurations (manual TPMTOTP verification)
- **Full support**: Can be used with both HOTP and non-HOTP board configurations

*NOTE* - If you prefer not to use USB security dongles or want simplified security procedures, see the [Purism Boot Modes](/PurismBootModes) documentation for information about Basic and Restricted boot modes that provide different security/usability trade-offs.

## EC firmware & customizations 🔧

The Embedded Controller (EC) is responsible for platform functions such as keyboard hotkeys, keyboard layout enforcement, battery/charging policies, and thermal control. On many supported ThinkPad boards the EC can only be updated as part of the vendor BIOS update process. Because of that, **apply any EC changes you require via vendor firmware before performing the initial Heads flash.**

Common EC customizations and caveats:
- **Keyboard mappings / key swaps** (e.g., allowing an X220 keyboard layout on an X230).
- **Battery whitelisting or vendor-specific battery policies** that can prevent booting with third-party batteries.
- **Power, charging, or thermal behavior** changes that alter how the system charges or manages thermals.

Recommended workflow:
1. If you do not need EC changes: update the vendor BIOS/firmware first, then proceed with SPI backups and Heads flashing.
2. If you do need EC changes: apply and verify them using vendor tools/firmware before flashing Heads; ensure the system boots normally under vendor firmware.

Important pre-update step — apply vendor BIOS/EC updates first:
- Prepare a USB bootable disk following El Torito instructions (for example: https://askubuntu.com/questions/651281/write-bootable-bios-update-iso-to-usb-stick), boot the prepared USB disk, and run the vendor's BIOS/firmware upgrade utility to apply EC updates. Doing this ensures vendor EC changes (keyboard mappings, battery whitelists, power/thermal policies) are applied prior to installing Heads and reduces the risk of unexpected behavior.
- Be sure the device is on AC power and the battery is charged before starting firmware updates; record the vendor firmware version and keep a copy of the firmware image and your SPI backups in a safe place.

3. Always back up BIOS/EC images and SPI dumps before making firmware changes.

References:
- ThinkPad EC examples: https://github.com/hamishcoleman/thinkpad-ec

**Note:** Heads/Coreboot will not modify an EC. If a board requires a custom EC blob, follow the board-specific build instructions and include the blob at build time.

(Add comprehensive SPI Programmer Best Practices guide and update existing flashing guides)
===

**Note**: All current Heads boards use a modern architecture where the Intel Management Engine (ME) is deactivated and the Intel Flash Descriptor (IFD) is unlocked. On older Intel platforms (up to Ivy Bridge/3rd gen), the ME can be neutered (most modules removed), while on newer platforms (Skylake and later), the ME is deactivated using HAP bits or other methods. The historical distinction between "Legacy" and "Maximized" boards is no longer relevant as of 2024, since all supported boards now use the approach that was previously called "maximized."

### Supported USB Security dongles:
|Manufacturer|Model|OpenPGP|HOTP verification|Compatible|
|--|--|:--:|:--:|:--:|
|Yubico|[YubiKey 5 Series](https://www.yubico.com/products/yubikey-5-overview/)|✅|❌|OpenPGP only|
|Nitrokey|[Nitrokey Pro 2](https://www.nitrokey.com/products/nitrokeys#comparison)|✅|✅|Full support|
|Nitrokey|[Nitrokey Storage 2](https://www.nitrokey.com/products/nitrokeys#comparison)|✅|✅|Full support|
|Nitrokey|[Nitrokey 3](https://www.nitrokey.com/products/nitrokeys#comparison)|✅|✅|Full support|
|Purism|[Librem Key](https://puri.sm/products/librem-key/)|✅|✅|Full support|

**Notes**:
- **OpenPGP only**: Can be used with non-HOTP board configurations (manual TPMTOTP verification)
- **Full support**: Can be used with both HOTP and non-HOTP board configurations

*NOTE* - If you prefer not to use USB security dongles or want simplified security procedures, see the [Purism Boot Modes](/PurismBootModes) documentation for information about Basic and Restricted boot modes that provide different security/usability trade-offs.

## EC firmware & customizations 🔧

The Embedded Controller (EC) is responsible for platform functions such as keyboard hotkeys, keyboard layout enforcement, battery/charging policies, and thermal control. On many supported ThinkPad boards the EC can only be updated as part of the vendor BIOS update process. Because of that, **apply any EC changes you require via vendor firmware before performing the initial Heads flash.**

Common EC customizations and caveats:
- **Keyboard mappings / key swaps** (e.g., allowing an X220 keyboard layout on an X230).
- **Battery whitelisting or vendor-specific battery policies** that can prevent booting with third-party batteries.
- **Power, charging, or thermal behavior** changes that alter how the system charges or manages thermals.

Recommended workflow:
1. If you do not need EC changes: update the vendor BIOS/firmware first, then proceed with SPI backups and Heads flashing.
2. If you do need EC changes: apply and verify them using vendor tools/firmware before flashing Heads; ensure the system boots normally under vendor firmware.

Important pre-update step — apply vendor BIOS/EC updates first:
- Prepare a USB bootable disk following El Torito instructions (for example: https://askubuntu.com/questions/651281/write-bootable-bios-update-iso-to-usb-stick), boot the prepared USB disk, and run the vendor's BIOS/firmware upgrade utility to apply EC updates. Doing this ensures vendor EC changes (keyboard mappings, battery whitelists, power/thermal policies) are applied prior to installing Heads and reduces the risk of unexpected behavior.
- Be sure the device is on AC power and the battery is charged before starting firmware updates; record the vendor firmware version and keep a copy of the firmware image and your SPI backups in a safe place.

3. Always back up BIOS/EC images and SPI dumps before making firmware changes.

References:
- ThinkPad EC examples: https://github.com/hamishcoleman/thinkpad-ec

**Note:** Heads/Coreboot will not modify an EC. If a board requires a custom EC blob, follow the board-specific build instructions and include the blob at build time.

(Add comprehensive SPI Programmer Best Practices guide and update existing flashing guides)

Emulated devices
===

For further information, see [Emulating Heads](/Emulating-Heads/)
