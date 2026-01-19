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

This guide provides essential information for safely flashing SPI chips using external programmers, with recommendations based on extensive community testing and feedback.

> **Note:** This page is organised for both **beginners** and **advanced users**. Beginners should follow the **Quick Start** steps below and the *Critical Safety Warning*; advanced content is collapsed in **Advanced** sections.

## TL;DR
A short comparison of recommended programmers for quick decision-making.

| Programmer | Best for | Voltage | Cost |
|------------|----------|---------:|------:|
| Tigard | Frequent flashing, debugging | 1.8V/3.3V/5V | $67-89 |
| CH347F | Budget but fast; adjustable VIO | 1.8V–3.3V (VIO) | $5-15 |
| CH341A rev1.6+ | Occasional flashing, budget | 1.8V/3.3V (selector) | $5-15 |
| Raspberry Pi Pico | DIY / low-cost | 3.3V only | $4-10 |

**Quick reminder:** Always make at least two verified full-chip backups before any write operation.

> **Quick Start — Safe minimal steps (Beginners):**
>
> 1. **Power off & remove all power sources** (batteries, AC). Wait 30s. If you cannot, stop.
> 2. **Confirm clip orientation & pin 1** (see SOIC8 clip orientation image) and ensure a Pomona‑style clip.
> 3. **Take a full chip backup and verify it** before doing any writes.
> 4. **Only after backups verify**, write firmware and verify the write.
>
```bash
# Detect and read a backup (replace placeholders)
sudo flashrom --programmer [programmer] --read backup.bin --chip "[chip_model]"
hexdump -C backup.bin | head -20
sudo flashrom --programmer [programmer] --verify backup.bin --chip "[chip_model]"
# Write (ONLY after you have verified backups):
sudo flashrom --programmer [programmer] --chip "[chip_model]" --write firmware.rom
```

> **If unsure, STOP and ask for help or measure the chip VCC with a multimeter.**



## ⚠️ Critical Safety Warning

**ALWAYS disconnect all power sources before flashing:**
- Remove all batteries (internal and external), including CMOS
- Disconnect AC adapter
- Wait 30 seconds for capacitors to discharge

**Never connect or disconnect the clip while the programmer is powered on.** This can damage your motherboard or SPI chip.

**EC note:** If your board requires EC updates or vendor-specific EC customizations (keyboard remaps, battery whitelists, power/thermal behaviour), apply those changes from the vendor BIOS prior to an initial Heads flash. See the **EC firmware & customizations** section in the [Prerequisites]({{ site.baseurl }}/Prerequisites) for details and workflow guidance.

## Recommended Programmers (in order of preference)

Based on extensive community testing and feedback, here are the recommended SPI programmers:

A short comparative read-time summary (observed community read times):

| Programmer | Typical read time (8MB / 16MB) |
|------------|-------------------------------:|
| Tigard | ~21s / ~42s |
| CH347F | ~30s / ~60s |
| CH341A rev1.6+ | ~120s / ~240s |
| Raspberry Pi Pico | ~15s / ~30s |

Benchmarks and detailed per-programmer timing data are available in issue #120 for those who want to dig deeper.

### Tigard (Recommended)
- **Cost**: $67-89 USD
- **Typical read time**: ~21 seconds for 8MB chip, ~42 seconds for 16MB chip (observed at Tigard's recommended SPI settings)
- **SPI clock notes**: Tigard supports VIO and user-selectable SPI clocks; speed and stability depend on chosen SPI clock.
- **Voltage**: 1.8V, 3.3V, 5V, external target voltage
- **Protocols**: SPI, I2C, JTAG, SWD, UART
- **Additional features**: Logic analyzer, USB debugging support
- **Reliability**: Excellent (4/4 success rate in community testing)
- **Best for**: Development, debugging, frequent flashing, new users

**Where to buy**: [Crowd Supply](https://www.crowdsupply.com/securinghw/tigard)

### CH347F (Preferred CH347 variant)
- **Cost**: $5-15 USD
- **Typical read time**: *depends on SPI clock* — e.g., ~30s for 16MB at modest clocks; **example:** CH347F reading a GD25LQ128E (16 MiB) at **60 MHz** observed **~4 s** read time (community measurement).
- **Voltage / VIO (I/O voltage reference / selector)**: **CH347F** modules commonly provide adjustable I/O voltage (via a physical selector switch or exposed VIO pin), supporting ≈1.8–3.3 V and typically exposing 5V-tolerant inputs. **CH347T** variants are commonly 3.3V-only. Always verify the specific module’s documentation or seller listing and look for an explicit VIO pin or voltage selector switch before using it with 1.8V chips.
- **Protocols**: SPI, I2C, JTAG (depending on module)
- **SPI clock**: Default with `flashrom` is commonly **15 MHz** (CH347); the clock can be increased (e.g., `--spispeed` in `flashrom`) up to **60 MHz** on capable modules. Higher stable SPI clocks yield near-linear speed improvements; unstable clocks require reducing the clock for reliability.
- **Best for**: Budget-conscious users who need higher speed and **adjustable VIO**; verify the seller/module exposes the CH347**F** variant or VIO setting and read the module documentation before purchasing.

Notes & sources:
- [CH347 datasheet / download](https://www.wch-ic.com/downloads/CH347DS1_PDF.html) 🔗
- [Review & schematic analysis: OneTransistor CH347 review](https://www.onetransistor.eu/2024/09/ch347t-programmer-schematic.html) 🔗
- [Community measurements and discussion: issue #120](https://github.com/linuxboot/heads-wiki/issues/120) 🔗

**Safety:** Always confirm the module's VIO (I/O voltage reference / selector) capability for 1.8V chips. **CH347F** modules that provide an adjustable I/O voltage via a selector switch or exposed VIO pin can be configured for 1.8V or 3.3V operation and—when the module documentation indicates 5V-tolerant inputs—do **not** require an external level shifter. If a module lacks an explicit selector/VIO pin or the documentation is unclear, use an external level shifter or prefer a programmer with explicit voltage control (for example, **Tigard** or **CH341A rev1.6+** with a physical selector).

### CH341A Rev 1.6+ (Budget option with voltage selector)
- **Cost**: $5-15 USD
- **Typical read time**: ~120 seconds for 8MB chip (~240s for 16MB) (community measurement — see issue #120 for write/verify timings)
- **Voltage**: 1.8V, 2.5V, 3.3V, 5V (selectable via switch)
- **Protocols**: SPI, I2C
- **Reliability**: Good when used correctly
- **Best for**: Occasional flashing, budget-conscious users

**⚠️ Important:** **Only use CH341A rev 1.6+ that explicitly provides a physical voltage selector or a regulator.** Older CH341A/CH341-style modules frequently output 5V and can permanently damage 3.3V flash chips.

**Visual identification (CH341A v1.6+ identification):** Look for a physical voltage selector switch or a regulator circuit; also inspect for extra components near the power pins. ![CH341A v1.6+ identification](https://github.com/linuxboot/heads-wiki/assets/827570/1b2621f6-978e-4b75-9c32-30647f741b09)


**Where to buy**: 
- [3mdeb Kit](https://shop.3mdeb.com/shop/modules/ch341a-flash-bios-usb-programmer-kit-soic8-sop8/)
- [Novacustom Kit](https://novacustom.com/product/modded-ch341a-bios-firmware-programmer-3v/)

#### CH341A visual identification — do not buy generic black modules

CH341-family devices vary a lot in quality and safety. Before buying a CH341 module, inspect it carefully. The safe CH341A rev 1.6+ models have a voltage selector or a dedicated regulator circuit; older generic black CH341 modules often drive 5V and can destroy 3.3V flash chips. Prefer **Pomona 5250** style SOIC clips for SOIC packages; do not rely on cheap generic clips that may slip or have poor contacts.

> Quick reference: See the CH341A section above for recommended CH341A models (rev 1.6+).

**Quick checklist**
- Look for a physical voltage selector (1.8V/3.3V/5V) or a regulator circuit near power pins.
- If it looks like the generic black unit above (no selector/regulator), **do not buy it**.
- Prefer Tigard or CH347F for safety; CH341A rev 1.6+ with selector is acceptable if you verify the VIO.

**Optimization tips (CH341A)**
- Ensure a clean power supply and a short, good-quality USB cable.
- Keep the programmer cool during long operations to prevent voltage drift or instability.
- If verification fails, try a different USB cable or power source before retrying.

### Raspberry Pi Pico (DIY option)
- **Cost**: $4-10 USD
- **Typical read time**: ~30 seconds for 16MB chip (see issue #120 for write/verify timings)
- **Voltage**: 3.3V (configurable via firmware)
- **Protocols**: SPI (via serprog firmware)
- **Reliability**: High
- **Best for**: DIY enthusiasts, custom setups

*Raspberry Pi Pico wiring example (Thanks @Thrilleratplay) for SPI programming. Ensure correct wiring and 3.3V operation before powering the target board.*

![SOIC8 clip orientation](https://gist.githubusercontent.com/Thrilleratplay/60dff7cf849de5bdb84d91cabbd4fcfd/raw/6dcfa783230cdbfa7548033ed4144546492a26cd/soic8_pinout.svg)

**Wiring tips**:
- Connect VCC (3.3V) from the Pico to the target's VCC/VIO, and GND to GND.
- Match MOSI/MISO/SCLK to the corresponding flash chip pins; use the Pomona clip orientation or board silks for pin-1.
- Use short jumper wires and confirm orientation with magnification; always verify voltage with a multimeter before writing.
- For boards needing 1.8V operation, use a programmer with proper VIO support (Tigard or CH347F) rather than a Pico (3.3V only).



## Essential Safety Procedures

### Before Starting
1. **Power disconnection checklist**:
   - [ ] Remove all batteries (internal and external), including CMOS
   - [ ] Disconnect AC adapter
   - [ ] Wait 30 seconds for capacitors to discharge

2. **Workspace preparation**:
   - Use anti-static wrist strap
   - Work on anti-static mat
   - Ensure good lighting
   - Have magnifying glass available

### Equipment checklist (for flashing)
- **Programmer**: Tigard (recommended). Budget: **CH347F** (preferred budget option with adjustable VIO) or **CH341A rev1.6+** (with physical selector) for basic flashing — always verify the module's VIO/voltage capability.
- **Clip**: Pomona 5250 (recommended) for SOIC chips — high quality, better contact.
- **WSON probe / spring-loaded adapter** for WSON packages (use with an alignment guider or jig).
- **Jumper leads** and a short USB cable (use short, good-quality wires).
- **Second computer** with `flashrom` installed (Linux recommended) to run the programmer.
- **Small tools & cleaning**: magnifier/microscope, isopropyl alcohol, flux, screwdrivers, spudger.
- **Sanity tools**: `hexdump`, `diff`, and basic Unix tools to inspect and compare dumps.

**Extremely important — backups:**
Make and verify multiple full‑chip backups, and store copies in a secure, separate location. If anything goes wrong or you need to restore stock firmware, these backups are required; if they are lost or corrupted recovery may be impossible and maintainers cannot help you.

### Voltage Selection Guidelines (beginner-friendly)
- **Stop if unsure**: If you cannot identify the chip or measure its VCC, do **not** perform in‑place programming — ask for help or remove the chip for external programming. ⚠️
- **Use a safe programmer**: Prefer a programmer with reliable voltage control. Examples: **Tigard** (recommended), **CH347F** (preferred budget option with a physical voltage selector or exposed VIO pin), **CH341A rev1.6+** (budget with 1.8V selector). 🔧
- **Power checklist (always)**: Remove batteries, disconnect AC adapter, and (if comfortable) disconnect the CMOS battery before attaching the clip. 🔌
- **If you can identify the chip**: Follow the chip datasheet for the correct VCC. If you can safely measure the chip’s VCC with a multimeter, use that voltage.

> Tip: Guessing voltages (e.g., "try 1.8V then 3.3V") can damage some chips. If you are not comfortable identifying/measuring VCC, stop and ask for help.



### Chip detection (overview)
## SPI Chip Pin Mapping (SOIC8)

```
Chip Pin Layout (top view, dot indicates pin 1):
┌─────────────┐
│ 1 ●       8 │
│ 2         7 │
│ 3         6 │
│ 4         5 │
└─────────────┘

Pin Functions:
1 - CS / CS#     (Chip Select)
2 - SO / MISO    (Serial Data Output)
3 - WP# / N/C    (Write Protect)
4 - GND          (Ground)
5 - SI / MOSI    (Serial Data Input)
6 - SCLK / CLK   (Clock)
7 - HOLD# / N/C  (Hold)
8 - VCC          (Power)
```

### Wiring Color Code Reference (from community testing)

When using custom wiring with color-coded cables, this mapping has been tested:

| SPI Chip Pin | Pomona Clip | Function | Wire Color | CH341A/Programmer |
|:------------:|:-----------:|:--------:|:----------:|:-----------------:|
| Pin 1 (dot)  | Pin 1 (red dot) | VCC | Red | 25XX (red stripe) |
| Pin 2        | Pin 2       | SO/MISO | Blue | DO (blue) |
| Pin 3        | Pin 3       | WP# | Brown | NC |
| Pin 4        | Pin 4       | GND | Black | GND (black) |
| Pin 5        | Pin 5       | SI/MOSI | Purple | DI (purple) |
| Pin 6        | Pin 6       | SCLK | Orange | CLK (orange) |
| Pin 7        | Pin 7       | HOLD# | White | NC |
| Pin 8        | Pin 8       | VCC | Red | VCC |

**Note**: Always verify pin 1 orientation by looking for the dot or notch on the chip. The red wire should connect to pin 1.

## Connection Procedures

### Using Tigard
```bash
# Basic connection test
sudo flashrom --programmer ft2232_spi:type=2232H,port=B,divisor=4

# With chip specification
sudo flashrom --programmer ft2232_spi:type=2232H,port=B,divisor=4 --chip "MX25L6405D"
```

### Using CH347F
```bash
# Basic connection test
sudo flashrom --programmer ch347_spi

# With chip specification and higher clock (example)
sudo flashrom --programmer ch347_spi --spispeed 60000 --chip "MX25L6405D"
```

### Using Raspberry Pi Pico (serprog)
```bash
# Basic connection test
sudo flashrom --programmer serprog:/dev/ttyACM0

# With chip specification and example clock
sudo flashrom --programmer serprog:/dev/ttyACM0 --spispeed 60000 --chip "MX25L6405D" --read backup.bin
```

- **Note:** Raspberry Pi Pico operates at **3.3V only**; do **not** use it for 1.8V chips. Ensure you have a Pico running a serprog-capable firmware and confirm its device node (e.g., `/dev/ttyACM0`). For 1.8V targets prefer Tigard or a CH347F module with VIO or use an external level shifter.

### Using CH341A rev 1.6+
```bash
# Basic connection test
sudo flashrom --programmer ch341a_spi

# With chip specification
sudo flashrom --programmer ch341a_spi --chip "MX25L6405D"
```

## Flashing Workflow

**Tool notation:** This section uses `flash_tool` as a generic placeholder for flashing utilities (for example, `flashrom` or `flashprog`). Use the syntax: `flash_tool [read|write|verify] romname-for-platform`. Examples in this guide use `flashrom` — most commands are interchangeable with `flashprog`.

**Protecting board-specific regions (GBE/IFD):** Many boards include small, board-specific regions (IFD / FMAP regions) that contain MAC addresses or other manufacturing data. **Always back up the full chip before the first flash** and inspect the dump (for example, `hexdump -C backup.bin | head -20`).

**Extremely important — keep multiple copies of your backups in a safe place.** If anything goes wrong or you need to restore stock firmware, these backups are required; if a backup is lost or corrupted, recovery may be impossible and we cannot help you.

If your goal is to preserve a board's GBE/MAC value, consult the board's `boards/<boardname>` build configuration and the community discussion in issue #120 for recommended workflows.

For most users: back up the full chip and follow the per-board flashing instructions; advanced selective-region updates are an internal workflow—consult the `boards/<boardname>` docs for details.


### Chip Detection and Backup
```bash
# Test connection and detect chip
sudo flashrom --programmer [programmer]

# Create backup and verify
sudo flashrom --programmer [programmer] --read backup.bin --chip "[chip_model]"
# Quick sanity check: inspect the start of the dump for obvious garbage
hexdump -C backup.bin | head -20
sudo flashrom --programmer [programmer] --verify backup.bin --chip "[chip_model]"
```

### Flash New Firmware

**Before writing:** ensure you have a verified full-chip backup, confirmed chip VCC with a multimeter, and validated clip/contact stability.

**Step-by-step (safe & copyable):**

1. Re-check the backup and inspect the start of the dump:

```bash
hexdump -C backup.bin | head -20
sudo flashrom --programmer [programmer] --verify backup.bin --chip "[chip_model]"
```

2. Write the new firmware (only after backups verified):

```bash
sudo flashrom --programmer [programmer] --chip "[chip_model]" --write firmware.rom --progress
```

Notes:
- Always put `--programmer` before other `flashrom` options.
- Use `--progress` for long operations and monitor output.
- If verification fails, stop and troubleshoot (re-check clip, wiring, voltages, and your backup); do **not** retry writes until the issue is resolved.

## Troubleshooting

### Common Issues and Solutions

**Chip not detected**:
- Check clip connection and orientation
- **Do not** guess voltages. Measure the chip VCC with a multimeter to determine the correct operating voltage before attempting further reads.
- Verify all power sources are disconnected
- Check clip contact pressure

**Verification fails**:
- Increase clip pressure
- Clean chip pins with isopropyl alcohol
- Try lower clock speed (for Tigard: increase divisor)
- Check for oxidation on clip contacts

**Partial writes**:
- Usually indicates poor connection
- Re-seat clip and try again
- Consider using probe-style adapter for better contact

### WSON8 Package Tips
For WSON8 packages (surface mount without leads):
- Prefer a dedicated spring-loaded WSON8 probe or a purpose-built adapter jig with an alignment guide. These ensure correct pad alignment and consistent contact pressure for reliable reads/writes.
- Avoid using a standard SOIC/Pomona clip directly on WSON pads — clips can slip or fail to make reliable contact. If you must use a clip, use a PCB adapter or a probe designed for WSON packages.
- Use an assisted guider (spring or fixture) or 3D-printed jig to hold the probe steady during long operations to prevent intermittent connections.
- **Verify electrical contact before committing to a write:** read a full backup (`sudo flashrom --programmer [programmer] --read backup.bin`), inspect the start (`hexdump -C backup.bin | head -20`), and **always** confirm with `--verify` to ensure the dump is readable and stable (for example: `sudo flashrom --programmer [programmer] --verify backup.bin --chip "[chip_model]"`).
- Ensure you have good magnification and lighting when positioning the probe, and consider using flux or contact cleaners if multiple attempts fail.

![WSON8 Probe](https://github.com/user-attachments/assets/ebcd780b-c7db-466a-91ea-a0d9d546b3ec)

> **Note:** For additional community photos and examples of spring-guided probes and jigs, see [issue #120](https://github.com/linuxboot/heads-wiki/issues/120).

## Performance Optimization

### SPI clock & performance (stability note)
- The observed read/write speeds are tied to the SPI clock used during the operation. If you increase the SPI clock (and your hardware remains stable), throughput increases roughly linearly; if you see instability or verification failures, reduce the SPI clock.
- Common parameters to change clock with `flashrom`:
  - **CH347 / Pico (serprog)**: use `--spispeed <kHz>` (e.g. `--spispeed 60000` for 60 MHz). Default CH347 clock with `flashrom` is commonly **15 MHz** but can go up to **60 MHz** on capable hardware.
  - **FT2232H** (used by Tigard-like FT2232 programmers): control clock via `divisor=<n>` (smaller divisor = higher clock); default behavior corresponds to ~30 MHz on many setups.
  - **CH341A**: many CH341A modules do **not** support changing SPI clock via a `flashrom` parameter; treat CH341A as limited in clock control unless your specific module exposes a setting.
- If you experience verification failures: try **reducing** the clock (e.g., `--spispeed` or larger divisor) until reads/verifies become consistent.

**Example (community measurements):**
- **CH347F reading GD25LQ128E (16 MiB) at 60 MHz**: read ≈ **4 s**. Example write costs: erase ≈ **12 s** + write ≈ **110 s** + verify ≈ **4 s** = **126 s** total (community measurement — see issue #120).

**Links & sources:**
- [CH347 datasheet](https://www.wch-ic.com/downloads/CH347DS1_PDF.html) 🔗
- [CH347 review & schematic (OneTransistor)](https://www.onetransistor.eu/2024/09/ch347t-programmer-schematic.html) 🔗
- [FT2232H datasheet](https://ftdichip.com/wp-content/uploads/2020/08/DS_FT2232H.pdf) 🔗
- [Flashrom documentation](https://www.flashrom.org/Flashrom) 🔗
- [Community timings/discussion: issue #120](https://github.com/linuxboot/heads-wiki/issues/120) 🔗

### Tigard Speed Optimization
- Use `divisor=4` for reliable operation
- Use `divisor=2` for maximum speed (may be less stable)
- Enable progress indicator: `--progress`

## Recommended Kits

### Complete Kits (Assembled and Ready)
See the **CH341A** subsection for CH341-specific kits and safety checks; for Tigard-based kits see the Tigard project's pages linked in the Tigard subsection.

### Individual Components
- **Tigard**: [Crowd Supply](https://www.crowdsupply.com/securinghw/tigard)
- **WSON8 probe**: [Amazon](https://www.amazon.ca/dp/B0DJ8XTCKD)

## Summary

For most users, **Tigard is the recommended choice** due to its:
- Fast operation (saves time during development)
- Multiple voltage options (safe for all chips)
- Additional debugging capabilities
- Excellent reliability record

**Budget & fast option:** **CH347F** is a good budget choice when you need higher speed and adjustable VIO; verify the specific module exposes VIO and how to configure it.

**Budget & basic option:** **CH341A rev 1.6+** is acceptable for occasional flashing but:
- Verify it has a physical voltage selector or regulator
- Expect longer read/write times
- Limited to basic SPI operations

**DIY option:** **Raspberry Pi Pico** is a capable DIY option for 3.3V-only workflows (requires more setup).

**Never use old CH341A programmers** without voltage selection — they can damage your hardware.
