---
layout: default
title: Lenovo X220 Maximized
permalink: /X220-maximized-flashing/
nav_order: 12
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Lenovo X220 (Maximized)
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

The X220 has a single 8 MB SPI flash chip located under the palm rest. Disassembly requires keyboard and palm rest removal — similar to the X230. Full disassembly to access the SPI chip is documented in coreboot:

→ **[coreboot Sandy Bridge series](https://doc.coreboot.org/mainboard/lenovo/Sandy_Bridge_series.html)** (8MB SPI, in-circuit=yes, socketed=no)  
→ **[coreboot X2xx series disassembly](https://doc.coreboot.org/mainboard/lenovo/x2xx_series.html)** (keyboard + palmrest removal)  
→ **[coreboot T420](https://doc.coreboot.org/mainboard/lenovo/t420.html)** (similar Sandy Bridge disassembly)  
→ **[Libreboot X220 external flashing guide](https://libreboot.org/docs/install/x220_external.html)** (disassembly photos, flashing commands)  
→ **[X220 Hardware Maintenance Manual](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x220/downloads/driver-list/component?name=Documentation%2FManuals)**

**Critical**: Remove all batteries (including CMOS) AND disconnect the AC adapter before starting.

## Flashing

The X220 uses a single 8 MB SPI flash chip -- no ROM splitting needed. Follow the X230 flashing guide for flashing commands, substituting the X220 board name. Use `[flasher]` of your choice (flashrom or flashprog -- see [Tool Interchangeability]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#tool-interchangeability)) with the programmer you selected ([programmer] -- see [Programmer Selection]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#programmer-selection)):

```shell
sudo [flasher] --programmer [programmer] --chip YYY --write ~/heads/build/x86/x220-maximized/x220-maximized.rom
```

See the [SPI Programmer Best Practices]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for programmer recommendations.
