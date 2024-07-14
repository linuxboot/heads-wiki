---
layout: default
title: Asus KGPE-D16
permalink: /kgpe-d16-flashing/
nav_order: 1
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Asus KGPE-D16
====

Prerequisits:
1. Make sure you have compatiable RAM and CPU combination, see HCL https://github.com/linuxboot/heads/issues/1395 if you have a combination RAM and CPU that is not listed on the HCL please report back your results so we can improve the hardware compatability.
2. A 16MB flash chip, the stock 2MB flash chip is not big enough to store heads
3. A flasher see https://github.com/linuxboot/heads-wiki/issues/120#issuecomment-2194771490

Instructions:
Flash the 16MB chip with the flasher and place the newly flashed chip in the board
ch341a example
1. place chip in flasher and pull down the locking mechanism
2. plug flasher into a computer with the flashrom software installed
3. flash chip using "sudo flashrom --programmer ch341a_spi -w heads.rom"
4. remove flasher from usb
5. release the locking mechanism and place the flashchip on the board
