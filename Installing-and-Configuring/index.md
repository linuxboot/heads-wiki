---
layout: default
title: Installing and configuring
permalink: /Install-and-Configure
nav_order: 4
has_children: yes
has_toc: false
---



## Required equipment:

To install Heads on a physical device, you will need:

* Supported motherboard or laptop ([see below](#supported-devices))
* A OTP security key ([see below](#security keys))
* [Spi Programmer](https://trmm.net/SPI_flash): ch341a programmer or raspberry pi or bus pirate (ch341a is recommended for new users and can be found almost [anywhere](https://www.amazon.com/s?k=ch341a+programmer). *NOTE* - Not needed if a Purism or OCP device
* Wires and a clip to connect your programmer of choice to the board’s SPI flash chip(s)
* If you plan to use a ch341a programmer you will need another computer to flash from (Try to use a recommended operating system: Qubes or Debian 9 or Fedora 30)
* If a X230 or T430, USB flash drive.

## Supported devices

|Device| Board name| Notes|
|--|--|--|
|Asus KGPE-D16|`kgpe-d16`||
|Dell R630|`r630`||
|Intel S2600wf|`s2600wf`||
|Lenovo Thinkpad T420|`t420`||
|Lenovo Thinkpad T430|`t430-flash`|flashed image|
|Lenovo Thinkpad T430|`t430`||
|Lenovo Thinkpad X220|`x220`||
|Lenovo Thinkpad X230|`x230-flash`|flashed image|
|Lenovo Thinkpad X230|`x230-hotp-verification`|with hotp verification|
|Lenovo Thinkpad X230|`x230`| |
|Open Compute Project	Leopard node|`leopard`||
|Open Compute Project	TiogaPass node|`tioga`||
|Open Compute Project	Winterfell node|`winterfell`||
|Purism Librem 13 v2|`librem_13v2`||
|Purism Librem 13 v4|`librem_13v4`||
|Purism Librem 15 v3|`librem_15v3`||
|Purism Librem 15 v4|`librem_15v4`||
|Purism Librem Mini|`librem_mini`||

## Emulated devices

For further information, see [Emulating Heads](/Emulating-Heads)

|Device| Board name| Notes|
|--|--|--|
|QEMU development image|`qemu-coreboot-fbwhiptail`||
|QEMU development image|`qemu-coreboot`||
|QEMU development image|`qemu-linuxboot`||


## security Keys

*NOTE* - Heads does **NOT** support FIDO2 or U2F authentication.  Be careful when
 purchasing to buy a compatible key.

|Manufacture|Model line|TOTP|HOTP|
|--|--|--|--|
|Yubikey|[YubiKey 5 Series](https://www.yubico.com/products/yubikey-5-overview)|X||
|Nitrokey|[Nitrokey Pro 2](https://www.nitrokey.com/#comparison)|X|X|
|Nitrokey|[Nitrokey Storage 2](https://www.nitrokey.com/#comparison)|X|X|
|Purism|[Librem Key](https://puri.sm/products/librem-key/)|X|X|
