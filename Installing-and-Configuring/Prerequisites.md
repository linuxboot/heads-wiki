---
layout: default
title: Prerequisites for Heads
permalink: /Prerequisites
nav_order: 1
parent: Installing and configuring
---

Prerequisites for Heads
===

Required equipment
---

To install Heads on a physical device, you will need:

* Supported motherboard or laptop ([see below](#supported-devices))
* A heads compatible USB security dongle ([see below](#USB Security Dongles))
* A heads compatible storage device for your public GPG key (USB flash drive)

If your device requires external flashing ([see below](#supported-devices)),
 you will also need:

* [SPI Programmer](https://trmm.net/SPI_flash): ch341a programmer or raspberry
 pi or bus pirate (ch341a is recommended for new users and can be found almost
 [anywhere](https://www.amazon.com/s?k=ch341a+programmer).
* Wires and a clip SOIC8 to connect your programmer of choice to the board’s
 SPI flash chip(s).
  * The [Pomona 5250](https://www.pomonaelectronics.com/products/test-clips/soic-clip-8-pin)
   is suggested as it is high quality and easier to make contact with the pins.
* A second computer to flash from (Try to use a recommended operating system:
  Qubes or Debian 9 or Fedora 30)

Supported devices
---

Please see the current [heads source](https://github.com/osresearch/heads/tree/master/boards) for supported devices.

|Device| Board name|Firmware base|Requires external flashing| ME should be cleaned|Notes|
|--|--|--|:--:|:--:|--|
|Asus KGPE-D16|`kgpe-d16`|coreboot|X|||
|Dell R630|`r630`|linuxboot||X||
|Intel S2600wf|`s2600wf`|linuxboot||X||
|Lenovo Thinkpad T420|`t420`|coreboot|X|X||
|Lenovo Thinkpad T430|`t430-flash`|coreboot|X|X|initial flashed image|
|Lenovo Thinkpad T430|`t430`|coreboot|X|X||
|Lenovo Thinkpad X220|`x220`|coreboot|X|X||
|Lenovo Thinkpad X230|`x230-flash`|coreboot|X|X|initial flashed image|
|Lenovo Thinkpad X230|`x230-hotp-verification`|coreboot|X|X|with hotp verification|
|Lenovo Thinkpad X230|`x230`|coreboot|X|X||
|Open Compute Project Leopard node|`leopard`|linuxboot|||
|Open Compute Project TiogaPass node|`tioga`|linuxboot||||
|Open Compute Project Winterfell node|`winterfell`|linuxboot||||
|Purism Librem 13 v2|`librem_13v2`|coreboot||||
|Purism Librem 13 v4|`librem_13v4`|coreboot||||
|Purism Librem 15 v3|`librem_15v3`|coreboot||||
|Purism Librem 15 v4|`librem_15v4`|coreboot||||
|Purism Librem Mini|`librem_mini`|coreboot||||

Emulated devices
---

For further information, see [Emulating Heads](/Emulating-Heads/)

|Device| Board name|  Firmware base|
|--|--|--|--|
|QEMU development image|`qemu-coreboot-fbwhiptail`|coreboot|
|QEMU development image|`qemu-coreboot`|coreboot|
|QEMU development image|`qemu-linuxboot`|linuxboot|

USB Security Dongles
---

*NOTE* - Heads does **NOT** support FIDO2 or U2F authentication.  Be careful when
 purchasing to buy a compatible key.

 *NOTE* - HOTP is currently only supported with Librem devices and the ThinkPad
  x230 rom with HOTP support

|Manufacture|Model line|TOTP|HOTP|
|--|--|:--:|:--:|
|Yubico|[YubiKey 5 Series](https://www.yubico.com/products/yubikey-5-overview/)|X||
|Nitrokey|[Nitrokey Pro 2](https://www.nitrokey.com/#comparison)|X|X|
|Nitrokey|[Nitrokey Storage 2](https://www.nitrokey.com/#comparison)|X|X|
|Purism|[Librem Key](https://puri.sm/products/librem-key/)|X|X|
