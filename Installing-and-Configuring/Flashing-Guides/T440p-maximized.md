---
layout: default
title: Lenovo T440p Maximized
permalink: /T440p-maximized-flashing/
nav_order: 6
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Lenovo T440p (Maximized)
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
for ESU dates and [Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-me-and-peripheral-firmware)
for security implications.

## ✅ PROTECTED: TPM GPIO Reset
{: .warning }
Pre-Skylake — dedicated PLTRST# pin.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).

## ⚡ Safety First
**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

## Disassembly

The T440p has **two SPI flash chips** — an 8 MB chip (SPI1) and a 4 MB chip (SPI2). The Heads ROM must be split before flashing:

```shell
dd if=heads-t440p-maximized.rom of=spi1_8mb.rom bs=1M count=8
dd if=heads-t440p-maximized.rom of=spi2_4mb.rom bs=1M skip=8
```

Flash the 8 MB chip (SPI1) with `spi1_8mb.rom` and the 4 MB top chip (SPI2) with `spi2_4mb.rom`.

### Step-by-step disassembly

1. Remove the back cover screws and the main battery.
2. Slide off the back cover.
3. Unplug the CMOS battery, the fan cable, and the black LED cable.
4. Remove all visible screws holding the bottom assembly (the ultrabay screw loosens but does not come out).
5. Pry up around the sides of the bottom assembly to release the clips, then lift it open like a clamshell toward the front — take care not to snap any wires.
6. The two flash chips are now visible near the RAM. The chip labeled **SPI1** is the 8 MB chip; the chip labeled **SPI2** (mounted above it) is the 4 MB chip.

For reference photos and detailed mechanical guidance, consult:
- **[Libreboot T440p external flashing guide](https://libreboot.org/docs/install/t440p_external.html)** — photo disassembly and ROM-splitting instructions
- **[coreboot T440p documentation](https://doc.coreboot.org/mainboard/lenovo/t440p.html)** — chip layout and flashing commands
- The Lenovo hardware maintenance manual is available from [Lenovo support](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-t-series-laptops/thinkpad-t440p) (search for "Hardware Maintenance Manual")

## Flashing

Connect your SPI programmer to the chip(s) being programmed, respecting pin 1 on each chip. Write each half of the split ROM to the corresponding chip:

Use `[flasher]` of your choice (flashrom or flashprog -- see [Tool Interchangeability]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#tool-interchangeability)) with the programmer you selected ([programmer] -- see [Programmer Selection]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#recommended-programmers)):

```shell
[flasher] --programmer [programmer] --chip "W25Q64FV" --write spi1_8mb.rom
[flasher] --programmer [programmer] --chip "W25Q32FV" --write spi2_4mb.rom
```

Replace `[programmer]` with your hardware (e.g. `linux_spi:dev=/dev/spidev0.0`, `ft2232_spi:type=2232H`, or `ch341a_spi`).

### Haswell-specific notes

- **[thinkpad_acpi](https://www.kernel.org/doc/html/latest/admin-guide/laptops/thinkpad-acpi.html)**: After the first boot with Heads, the `thinkpad_acpi` kernel module may need the `fan_control=1` parameter to enable manual fan control. Add `thinkpad_acpi.fan_control=1` to the kernel command line or pass it as a module parameter.
- **[MRC](https://doc.coreboot.org/soc/intel/fsp.html) (Memory Reference Code)**: The T440p uses Haswell MRC training data stored in SPI flash. If the flash layout is corrupted or you are flashing a completely blank board, the machine may not boot until MRC training runs. Heads does not include a built-in MRC cache; the first boot after flashing may take 30–60 seconds as the platform retrains memory. Subsequent boots will reuse cached parameters.
- **GPIO PLTRST#**: As noted above, the T440p has a dedicated PLTRST# pin for TPM GPIO reset, making it one of the better-protected pre-Skylake boards in the Heads threat model.
