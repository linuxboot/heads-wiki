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

The thinkpad T420 has only one SPI flash chip that holds the BIOS, ME, etc. It is located under the palm rest. Similarly to the T430, to access this chip complete disassembly is required. It is a straightforward process and takes approximately 30 minutes. For this follow the T430/x230 guide. 

**Critical**: Remove all batteries (including CMOS) AND disconnect the AC adapter before starting.

[Here](https://www.coreboot.org/Board:lenovo/t420) is the location of the chip. At some models the location of the dot where the red wire from programmer should go may be misleading. The dot you need is just black.

![T420 SPI flash chip]({{ site.baseurl }}/images/T420_SPI_chip.jpg)

**Note**: See the CH341A guidance in the [SPI Programmer Best Practices]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#2-ch341a-rev-16-budget-option-with-voltage-selector) for vendor-safe programmer recommendations.

You should then follow through with [configuring keys]({{ site.baseurl }}/Configuring-Keys/).
