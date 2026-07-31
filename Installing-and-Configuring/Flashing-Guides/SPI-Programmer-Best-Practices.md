---
layout: default
title: SPI Programmer Best Practices
permalink: /SPI-Programmer-Best-Practices/
nav_order: 0
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
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

This is the authoritative reference for safely flashing SPI chips with external programmers. Every board-specific flashing guide links here -- read it before you touch any hardware.

## TL;DR

| Programmer | Best for | Voltage | Speed (16MB read) | Cost |
|------------|----------|---------|-------------------:|------:|
| Tigard | Frequent flashing, debugging | 1.8V / 3.3V / 5V | ~42s | $67-89 |
| CH347F | Budget but fast; adjustable VIO | 1.8V-3.3V (VIO) | ~60s (or ~4s at 60MHz) | $5-15 |
| CH341A rev1.6+ | Occasional flashing, budget | 1.8V / 3.3V (selector) | ~240s | $5-15 |
| [Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/) | DIY / low-cost | 3.3V only | ~30s | $4-10 |

**Before you start:** always make at least two verified full-chip backups to separate locations. Without them, recovery is impossible and we cannot help you.

**Quick start (safe minimal steps):**
1. **Power off** -- remove all batteries (including CMOS), disconnect AC, wait 30s.
2. **Confirm clip orientation** -- match pin 1 (dot) on chip to red wire on [Pomona 5250](https://www.pomonaelectronics.com/products/test-clips/soic-clip-8-pin) clip.
3. **Backup and verify** -- read the full chip and verify the dump before any writes.
4. **Flash** -- only after verified backups. See [Backup & Verify](#backup--verify) and [Flashing](#flashing-new-firmware) below.

> **If unsure, STOP. Ask for help or measure the chip VCC with a multimeter.**

## ⚠️ Critical Safety

### Power Disconnection

**Always disconnect all power sources before attaching the clip:**
- Remove batteries (internal, external, and CMOS)
- Disconnect AC adapter
- Wait 30 seconds for capacitors to discharge
- **Never connect or disconnect the clip while the programmer is powered on** -- this can damage your motherboard or SPI chip.

### Voltage

**Guessing voltage can destroy chips.** If you cannot identify the chip or measure its VCC with a multimeter, do not attempt in-system programming -- remove the chip or ask for help.

- Prefer a programmer with reliable voltage control (Tigard, CH347F with VIO, or CH341A rev1.6+ with physical selector).
- Match voltage to the chip's datasheet spec, not to "what seems to work."
- 1.8V chips require a programmer with explicit 1.8V support -- Tigard or CH347F with VIO. Do not use 3.3V-only programmers (Pico, CH347T) for 1.8V targets.

### Workspace

- Use an anti-static wrist strap and mat.
- Work under good lighting with magnification available.
- Have isopropyl alcohol and flux handy for cleaning contacts.

### EC Firmware

If your board requires vendor-specific EC customizations (keyboard remaps, battery whitelists, power/thermal behaviour), apply those from the vendor BIOS before your first Heads flash. See [Prerequisites]({{ site.baseurl }}/Prerequisites).

## Tool Interchangeability

Board-specific guides use these placeholders in command examples:

- **`[flasher]`** -- `flashrom` or [`flashprog`](https://flashprog.org/). These tools are interchangeable; most commands work identically in both. Install whichever your distro packages (`apt install flashrom` or equivalent).
- **`[programmer]`** -- your SPI programmer type. Common values: `ch341a_spi`, `ch347_spi`, `ft2232_spi:type=2232H,port=B,divisor=4`, `serprog:/dev/ttyACM0`. See [Connection Examples](#connection-examples) for the full programmer strings.

Replace placeholders with your actual values. Your board's specific flashing guide has board-specific commands (chip model, ROM path, dual-chip procedures) that use these placeholders.

## Recommended Programmers

Community-tested, ordered by preference. See [issue #120](https://github.com/linuxboot/heads-wiki/issues/120) for detailed benchmarks.

### Tigard (recommended)

- **Cost:** $67-89 | **Voltage:** 1.8V, 3.3V, 5V, external target voltage
- **Protocols:** SPI, I2C, JTAG, SWD, UART
- **Speed:** ~21s (8MB), ~42s (16MB) at default SPI settings; tunable via `divisor`
- **Reliability:** Excellent (4/4 success in community testing)
- **Best for:** Development, debugging, frequent flashing, new users

The Tigard's SPI header matches standard SOIC8 chip pinout, making it easy to attach a [Pomona 5250](https://www.pomonaelectronics.com/products/test-clips/soic-clip-8-pin) clip. Also includes logic analyzer and USB debugging.

**Buy:** [Crowd Supply](https://www.crowdsupply.com/securinghw/tigard)

### CH347F (preferred budget)

- **Cost:** $5-15 | **Voltage:** Adjustable VIO ~1.8-3.3V (physical selector or exposed VIO pin); typically 5V-tolerant inputs
- **Protocols:** SPI, I2C, JTAG (module-dependent)
- **Speed:** Default ~15 MHz SPI clock; up to 60 MHz on capable modules. At 60 MHz a 16MB chip reads in ~4s.
- **Best for:** Budget users who need speed and adjustable VIO

**Critical:** Verify the module is a **CH347F** variant with VIO support. CH347T variants are 3.3V-only. Check seller documentation for a voltage selector switch or VIO pin before purchasing. If unclear, use Tigard or CH341A rev1.6+ instead.

**References:** [CH347 datasheet](https://www.wch-ic.com/downloads/CH347DS1_PDF.html) | [OneTransistor review](https://www.onetransistor.eu/2024/09/ch347t-programmer-schematic.html)

### CH341A Rev 1.6+ (budget, occasional use)

- **Cost:** $5-15 | **Voltage:** 1.8V, 2.5V, 3.3V, 5V (physical selector switch)
- **Protocols:** SPI, I2C
- **Speed:** ~120s (8MB), ~240s (16MB); no user-adjustable SPI clock
- **Best for:** Occasional flashing, budget users who verify voltage

**⚠️ Only use rev 1.6+ modules with a physical voltage selector or regulator circuit.** Older generic black CH341 modules output 5V and will destroy 3.3V chips.

![CH341A v1.6+ identification](https://github.com/linuxboot/heads-wiki/assets/827570/1b2621f6-978e-4b75-9c32-30647f741b09)

**Quick checklist:** Look for a voltage selector switch (1.8V/3.3V/5V) or regulator near power pins. If it looks like a generic black module without these, **do not buy it.**

**Buy:** [3mdeb Kit](https://shop.3mdeb.com/shop/modules/ch341a-flash-bios-usb-programmer-kit-soic8-sop8/) | [Novacustom Kit](https://novacustom.com/product/modded-ch341a-bios-firmware-programmer-3v/)

### [Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/) (DIY)

- **Cost:** $4-10 | **Voltage:** 3.3V only
- **Protocols:** SPI (via [serprog](https://www.flashrom.org/serprog) firmware)
- **Speed:** ~30s (16MB)
- **Reliability:** High
- **Best for:** DIY enthusiasts, 3.3V-only workflows

Requires [serprog](https://www.flashrom.org/serprog)-capable firmware flashed to the Pico. Confirm device node (e.g., `/dev/ttyACM0`). **Do not use for 1.8V chips** -- prefer Tigard or CH347F.

![SOIC8 clip orientation](https://gist.githubusercontent.com/Thrilleratplay/60dff7cf849de5bdb84d91cabbd4fcfd/raw/6dcfa783230cdbfa7548033ed4144546492a26cd/soic8_pinout.svg)

**Wiring:** VCC (3.3V) to target VCC, GND to GND, MOSI/MISO/SCLK to corresponding flash chip pins. Use short jumper wires, confirm orientation with magnification, and verify voltage with a multimeter before writing.

## Physical Connection

### SPI Chip Pin Mapping (SOIC8)

```
Chip Pin Layout (top view, dot / notch = pin 1):
┌─────────────┐
│ 1 ●       8 │
│ 2         7 │
│ 3         6 │
│ 4         5 │
└─────────────┘

1 - CS / CS#    5 - SI / MOSI
2 - SO / MISO   6 - SCLK / CLK
3 - WP#         7 - HOLD#
4 - GND         8 - VCC
```

### Wiring Color Reference

Tested mapping for [Pomona 5250](https://www.pomonaelectronics.com/products/test-clips/soic-clip-8-pin) clip and CH341A-style programmers:

| Chip Pin | Pomona Clip | Function | Wire Color | Programmer Header |
|:--------:|:-----------:|:--------:|:----------:|:-----------------:|
| Pin 1 (dot) | Pin 1 (red dot) | CS | Red | 25XX / CS |
| Pin 2 | Pin 2 | SO/MISO | Blue | DO / MISO |
| Pin 3 | Pin 3 | WP# | Brown | NC |
| Pin 4 | Pin 4 | GND | Black | GND |
| Pin 5 | Pin 5 | SI/MOSI | Purple | DI / MOSI |
| Pin 6 | Pin 6 | SCLK | Orange | CLK |
| Pin 7 | Pin 7 | HOLD# | White | NC |
| Pin 8 | Pin 8 | VCC | Red | VCC |

**Always verify pin 1 orientation** by locating the dot or notch on the chip. Red wire to pin 1.

## Connection Examples

Test your programmer connection before attempting reads or writes. Run these from a Linux machine with `flashrom` installed.

### Tigard

```bash
sudo flashrom --programmer ft2232_spi:type=2232H,port=B,divisor=4
```

### CH347F

```bash
sudo flashrom --programmer ch347_spi
# With higher clock (example):
sudo flashrom --programmer ch347_spi --spispeed 60000
```

### CH341A Rev 1.6+

```bash
sudo flashrom --programmer ch341a_spi
```

### [Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/) ([serprog](https://www.flashrom.org/serprog))

```bash
sudo flashrom --programmer serprog:/dev/ttyACM0
```
**Note:** 3.3V only. Verify the Pico is running [serprog](https://www.flashrom.org/serprog) firmware and the device node matches your system.

## Backup & Verify

**This is the most important step.** Without verified backups, bricked hardware may be unrecoverable.

```bash
# 1. Detect the chip
sudo [flasher] --programmer [programmer]

# 2. Read full chip backup
sudo [flasher] --programmer [programmer] --read backup1.bin --chip [chip]

# 3. Inspect the dump (sanity check)
hexdump -C backup1.bin | head -20

# 4. Verify the backup reads back correctly
sudo [flasher] --programmer [programmer] --verify backup1.bin --chip [chip]

# 5. Read a second backup to a different file (paranoia is good)
sudo [flasher] --programmer [programmer] --read backup2.bin --chip [chip]
sudo [flasher] --programmer [programmer] --verify backup2.bin --chip [chip]
```

**Store backups in a safe, separate location.** If anything goes wrong or you need to restore stock firmware, these are your only recovery path. If they are lost or corrupted, recovery may be impossible and maintainers cannot help you.

> **Board-specific regions (GBE/IFD):** Some boards store MAC addresses or manufacturing data in chip regions. These are preserved by Heads during normal operation. For advanced selective-region workflows, consult your board's `boards/<boardname>` build configuration and [issue #120](https://github.com/linuxboot/heads-wiki/issues/120).

## Flashing New Firmware

**Only proceed after verified backups.**

```bash
sudo [flasher] --programmer [programmer] --chip [chip] --write firmware.rom --progress
```

**Before writing, confirm:**
- Verified full-chip backup saved to a safe location
- Chip VCC confirmed with multimeter (or programmer voltage set correctly)
- Clip seated firmly with clean contacts
- Correct firmware file for your board (check path and filename)

If verification fails after writing: **do not retry**. Troubleshoot the connection first (clip seating, voltage, clock speed). See [Troubleshooting](#troubleshooting).

## Troubleshooting

### Chip Not Detected

- Check clip connection and orientation (pin 1 alignment)
- **Do not guess voltage.** Measure chip VCC with a multimeter.
- Verify all power sources are disconnected (including CMOS battery).
- Increase clip contact pressure; clean chip pins with isopropyl alcohol.

### Verification Fails

- Increase clip pressure or re-seat the clip.
- Clean chip pins and clip contacts with isopropyl alcohol.
- Reduce SPI clock speed:
  - **CH347 / Pico:** `--spispeed 15000` (or lower)
  - **Tigard:** increase divisor, e.g. `divisor=6` (10 MHz) or `divisor=8` (7.5 MHz)
  - **CH341A:** no clock control; try a shorter/better USB cable or different USB port
- Check for oxidation on contacts; use flux if needed.

### Partial Writes / Intermittent Errors

- Usually a physical connection issue. Re-seat the clip.
- Consider a spring-loaded WSON probe or adapter jig for better contact (see below).
- Use a short, high-quality USB cable.

### WSON8 Package Tips

For WSON8 surface-mount packages (no leads):

- Prefer a **spring-loaded WSON8 probe** or purpose-built adapter jig with alignment guide. These ensure correct pad alignment and consistent contact pressure.
- **Do not** use a standard SOIC/Pomona clip directly on WSON pads -- clips slip and fail to make reliable contact. If you must use a clip, use a PCB adapter designed for WSON.
- Use a spring-assisted guide or 3D-printed jig to hold the probe steady during long operations.
- Start by reading a backup and verifying it to confirm reliable contact before committing to a write.

![WSON8 Probe](https://github.com/user-attachments/assets/ebcd780b-c7db-466a-91ea-a0d9d546b3ec)

See [issue #120](https://github.com/linuxboot/heads-wiki/issues/120) for community photos of spring-guided probes and jigs.

### SPI Clock & Performance

SPI clock directly affects read/write speed. If you increase the clock, throughput scales roughly linearly -- until the connection becomes unstable.

**Clock control by programmer:**

| Programmer | Parameter | Notes |
|------------|-----------|-------|
| CH347 | `--spispeed <kHz>` (e.g. `--spispeed 60000`) | Default ~15 MHz, max 60 MHz. Use powers-of-two steps: 60M, 30M, 15M, 7.5M... |
| Pico (serprog) | `--spispeed <kHz>` | 3.3V only |
| Tigard (FT2232H) | `divisor=<n>` | Clock = 60 MHz / n. `n` must be even (2, 4, 6...). Default `divisor=2` = 30 MHz. |
| CH341A | N/A | No adjustable SPI clock |

**If verification fails, reduce the clock** (lower `--spispeed` or larger `divisor`) until reads/verifies become consistent.

**Example (community measurement):** CH347F reading GD25LQ128E (16MB) at 60 MHz: read ~4s, erase ~12s, write ~110s, verify ~4s = ~126s total.

**References:** [CH347 datasheet](https://www.wch-ic.com/downloads/CH347DS1_PDF.html) | [FT2232H datasheet](https://ftdichip.com/wp-content/uploads/2020/08/DS_FT2232H.pdf) | [Flashrom docs](https://www.flashrom.org/) | [Community benchmarks: issue #120](https://github.com/linuxboot/heads-wiki/issues/120)

## Summary

**Tigard** is the recommended choice: fast, multi-voltage, excellent reliability, debugging features. Worth the cost for anyone flashing more than once.

**CH347F** is the best budget option when you need speed and adjustable VIO. Verify the module exposes VIO before buying.

**CH341A rev 1.6+** is acceptable for occasional use with a physical voltage selector. Expect long read/write times and basic SPI-only operation. **Never use old CH341 modules without voltage selection** -- they can destroy your hardware.

**[Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/)** is a capable DIY option for 3.3V-only workflows, requiring more setup.

## Component Sources

- **Tigard:** [Crowd Supply](https://www.crowdsupply.com/securinghw/tigard)
- **CH341A kits:** [3mdeb](https://shop.3mdeb.com/shop/modules/ch341a-flash-bios-usb-programmer-kit-soic8-sop8/) | [Novacustom](https://novacustom.com/product/modded-ch341a-bios-firmware-programmer-3v/)
- **WSON8 probe:** [Amazon](https://www.amazon.ca/dp/B0DJ8XTCKD)
