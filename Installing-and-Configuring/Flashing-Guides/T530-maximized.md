---
layout: default
title: Lenovo T530 Maximized
permalink: /T530-maximized-flashing/
nav_order: 9
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Lenovo T530 (Maximized)
===

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details> 

## ⚠️ EOL: No microcode updates
{: .warning }
This board's CPU generation has reached End of Servicing Updates.
See [per-board EOL/ESU status](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status)
for ESU dates and [Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-microcode-updates-and-transient-execution-vulnerabilities)
for security implications.

## ✅ PROTECTED: TPM GPIO Reset
{: .warning }
Pre-Skylake — dedicated PLTRST# pin.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).

## ⚡ Safety First
**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

## Disassembly
The T530 shares the same board and SPI layout as the W530. See:

→ **[coreboot W530/T530 documentation](https://doc.coreboot.org/mainboard/lenovo/w530.html)** (dual SPI flash, /CS sharing trick)
→ **[Lenovo Ivy Bridge series](https://doc.coreboot.org/mainboard/lenovo/Ivy_Bridge_series.html)** (8MB + 4MB, socketed=no)
→ **[Internal flashing](https://doc.coreboot.org/mainboard/lenovo/ivb_internal_flashing.html)** (SMM_BWP/BLE exploit for BIOS < 2014)
→ **[ThinkPad W530/T530 HMM](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-t-series-laptops/thinkpad-t530/downloads/driver-list/component?name=Documentation%2FManuals)** (Hardware Maintenance Manual — shared for T530 and W530)

The T530 and W530 use a **dual SPI flash** layout (8 MB bottom + 4 MB top), with both chips sharing the same /CS line. The MOSI/MISO lines are linked between the two chips through zero-ohm resistors. When externally flashing one chip, the **other chip's CS# must be pulled HIGH** (to VCC, 3.3V) via a 47 Ω resistor to disable it and prevent bus contention. The same trick is used on the W541 — see the **[libreboot W541 external flashing guide](https://libreboot.org/docs/install/w541_external.html)** for detailed wiring instructions.

The physical disassembly (keyboard removal, palmrest removal, then flip the board) is the same as the T430 since they share the same chassis family. See the **[T430 disassembly guide]({{ site.baseurl }}/T430-maximized-flashing/#disassembly)** for step-by-step photos.

On older BIOS versions (pre-2014) with SMM_BWP=0 and BLE=0, **internal flashing** is possible without disassembly — see the coreboot internal flashing link above.

## Flashing

The SPI flash layout and flashing commands are the same as the T430. The **top** chip is 4 MB (contains BIOS and reset vector) and the **bottom** chip is 8 MB (contains ME firmware and flash descriptor).

Before connecting the programmer to either chip, ensure the **other chip's CS# is pulled HIGH** to VCC via a 47 Ω resistor to prevent bus contention. See the libreboot W541 guide above for wiring details.

Then follow the **[T430 flashing procedure]({{ site.baseurl }}/T430-maximized-flashing/#flashing)** step-by-step:

Use `[flasher]` of your choice (flashrom or flashprog -- see [Tool Interchangeability]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#tool-interchangeability)) with the programmer you selected ([programmer] -- see [Programmer Selection]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#programmer-selection)):

```shell
# Identify the chip
sudo [flasher] --programmer [programmer]

# Create a backup of the top (4 MB) chip and verify it
sudo [flasher] --read ~/top.bin --programmer [programmer] --chip <chip> && \
    sudo [flasher] --verify ~/top.bin --programmer [programmer] --chip <chip>

# Write the top ROM
sudo [flasher] --programmer [programmer] --chip <chip> --write ~/heads/build/x86/t530-maximized/t530-maximized-top.rom

# Repeat for the bottom (8 MB) chip
sudo [flasher] --read ~/bottom.bin --programmer [programmer] --chip <chip> && \
    sudo [flasher] --verify ~/bottom.bin --programmer [programmer] --chip <chip>

sudo [flasher] --programmer [programmer] --chip <chip> --write ~/heads/build/x86/t530-maximized/t530-maximized-bottom.rom
```

After flashing, force power off by holding the power button for 10 seconds. Two reboots are sometimes needed after flash since memory training data is wiped.

Then follow through with **[configuring keys]({{ site.baseurl }}/Configuring-Keys/)**.
