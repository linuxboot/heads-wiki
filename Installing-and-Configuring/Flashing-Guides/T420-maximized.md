---
layout: default
title: Lenovo T420 Maximized
permalink: /T420-maximized-flashing/
nav_order: 4
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Lenovo T420 (Maximized)
===

## ⚠️ Safety First

**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

[T420 Hardware Maintenance Manual](https://download.lenovo.com/pccbbs/mobiles_pdf/t420_and_t420i_ug_en.pdf)  

The thinkpad T420 has only one SPI flash chip that hold the BIOS, ME, etc. It is located under the palm rest. Similarly to the T430, to access this chip complete disassembly is required. It is a straightforward process and takes approximately 30 minutes. For this follow the T430/x230 guide. 

**Critical**: Remove the battery AND disconnect the AC adapter before starting.

[Here](https://www.coreboot.org/Board:lenovo/t420) is the location of the chip. At some models the location of the dot where the red wire from programmer should go may be misleading. The dot you need is just black.

![T420 SPI flash chip]({{ site.baseurl }}/images/T420_SPI_chip.jpg)

**Note**: For safety and reliability, we recommend using [Tigard or CH341A rev 1.6+]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) instead of older CH341A programmers.

You should then follow through with [configuring keys]({{ site.baseurl }}/Configuring-Keys/).
