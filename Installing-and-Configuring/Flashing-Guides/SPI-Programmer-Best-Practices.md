---
layout: default
title: SPI Programmer Best Practices
permalink: /SPI-Programmer-Best-Practices/
nav_order: 0
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

# SPI Programmer Best Practices

This guide provides essential information for safely flashing SPI chips using external programmers, with recommendations based on extensive community testing and feedback.

## ⚠️ Critical Safety Warning

**ALWAYS disconnect all power sources before flashing:**
- Remove all batteries (internal and external), including CMOS
- Disconnect AC adapter
- Wait 30 seconds for capacitors to discharge

**Never connect or disconnect the clip while the programmer is powered on.** This can damage your motherboard or SPI chip.

**EC note:** If your board requires EC updates or vendor-specific EC customizations (keyboard remaps, battery whitelists, power/thermal behaviour), apply those changes from the vendor BIOS prior to an initial Heads flash. See the **EC firmware & customizations** section in the [Prerequisites]({{ site.baseurl }}/Prerequisites) for details and workflow guidance.

## Recommended Programmers (in order of preference)

Based on extensive community testing and feedback, here are the recommended SPI programmers:

**Note on timings:** The speed figures below are community measurements. For the most accurate per-programmer read/write/verify breakdown, see the benchmarking performed by tlaurion in https://github.com/linuxboot/heads-wiki/issues/120. The numbers shown in the table and in the programmer sections are representative observed **read** times (unless otherwise noted); write and verify operations are typically longer and are documented in the linked issue. Actual times vary by host, cable, chip model, and flashrom parameters.

**Testing methodology:** Timings in issue #120 were measured using controlled read and write operations; consult that issue for the read vs write results and full test setup details.

### 1. **Tigard (Recommended)**
- **Cost**: $67-89 USD
- **Typical read time**: ~21 seconds for 8MB chip, ~42 seconds for 16MB chip
- **Typical read time (32MB observed)**: ~2m37s (community measurement)
- **Voltage**: 1.8V, 3.3V, 5V, external target voltage
- **Protocols**: SPI, I2C, JTAG, SWD, UART
- **Additional features**: Logic analyzer, USB debugging support
- **Reliability**: Excellent (4/4 success rate in community testing)
- **Best for**: Development, debugging, frequent flashing, new users

**Where to buy**: [Crowd Supply](https://www.crowdsupply.com/securinghw/tigard)

### 2. **CH341A Rev 1.6+ (Budget option with voltage selector)**
- **Cost**: $5-15 USD
- **Typical read time**: ~120 seconds for 8MB chip (~240s for 16MB) (community measurement — see issue #120 for write/verify timings)
- **Voltage**: 1.8V, 2.5V, 3.3V, 5V (selectable via switch)
- **Protocols**: SPI, I2C
- **Reliability**: Good when used correctly
- **Best for**: Occasional flashing, budget-conscious users

**⚠️ Important**: Only use CH341A rev 1.6+ with voltage selector. Older versions output dangerous 5V.

**Visual identification**: Look for the voltage selector switch and additional components:
![CH341A v1.6+ identification](https://github.com/linuxboot/heads-wiki/assets/827570/1b2621f6-978e-4b75-9c32-30647f741b09)

**Where to buy**: 
- [3mdeb Kit](https://shop.3mdeb.com/shop/modules/ch341a-flash-bios-usb-programmer-kit-soic8-sop8/)
- [Novacustom Kit](https://novacustom.com/product/modded-ch341a-bios-firmware-programmer-3v/)

### 3. **Raspberry Pi Pico (DIY option)**
- **Cost**: $4-10 USD
- **Typical read time**: ~30 seconds for 16MB chip (see issue #120 for write/verify timings)
- **Voltage**: 3.3V (configurable via firmware)
- **Protocols**: SPI (via serprog firmware)
- **Reliability**: High
- **Best for**: DIY enthusiasts, custom setups

![Raspberry Pi Pico wiring example](https://private-user-images.githubusercontent.com/197665737/409598181-f5df8c82-f9fb-4302-b823-4951781e4e9b.jpg)
*Raspberry Pi Pico wiring example for SPI programming. Ensure correct wiring and 3.3V operation before powering the target board.*

**Wiring tips**:
- Connect VCC (3.3V) from the Pico to the target's VCC/VIO, and GND to GND.
- Match MOSI/MISO/SCLK to the corresponding flash chip pins; use the Pomona clip orientation or board silks for pin-1.
- Use short jumper wires and confirm orientation with magnification; always verify voltage with a multimeter before writing.
- For boards needing 1.8V operation, use a programmer with proper VIO support (Tigard or CH347F) rather than a Pico (3.3V only).

### 4. **CH347 (Newer alternative to CH341A)**
- **Cost**: $5-15 USD
- **Typical read time**: ~60 seconds for 16MB chip (community measurement; see issue #120 for write/verify timings)
- **Voltage**: Varies by module. Some CH347 modules expose a VIO pin (allowing ~1.2–3.3V I/O); others only provide 3.3V and will require an external level shifter for 1.8V chips.
- **Protocols**: SPI, I2C
- **Safety**: Prefer modules that expose a VIO pin or explicit voltage selection; if your module lacks VIO, use an external level shifter for 1.8V chips or choose a programmer with built-in VIO support (e.g., Tigard or CH341A rev1.6+ with selector).
- **Best for**: Budget option when a VIO/exposed voltage control is available; otherwise prefer CH341A rev1.6+ or Tigard.

WCH module documentation: https://www.wch-ic.com/download/file?id=348

## Programmer Comparison Table

| Programmer | Cost | Typical read time (16MB) | Voltage Selection | Protocols | Debugging Features | Reliability |
|------------|------|-----------------------|-------------------|-----------|-------------------|-------------|
| **Tigard** | $67-89 | ~42 seconds (16MB read); ~2m37s (32MB observed read) | 1.8V, 3.3V, 5V, external | SPI, I2C, JTAG, SWD, UART | Logic analyzer, USB debugging | Excellent |
| **Raspberry Pi Pico** | $4-10 | ~30 seconds (16MB read) | 3.3V (configurable) | SPI | None | High |
| **CH347 (prefer CH347F)** | $5-15 | ~60 seconds (16MB read) | VIO or 3.3V (CH347F has VIO; CH347T may only be 3.3V) | SPI, I2C | None | Good |

*Note: numbers above reflect typical read times; see https://github.com/linuxboot/heads-wiki/issues/120 for a detailed read/write/verify benchmark and test methodology.*

### CH341 visual identification — do not buy generic black modules

CH341-family devices vary a lot in quality and safety. Before buying a CH341 module, inspect it carefully. The safe CH341A rev 1.6+ models have a voltage selector or a dedicated regulator circuit; older generic black CH341 modules often drive 5V and can destroy 3.3V flash chips. Prefer **Pomona 5250** style SOIC clips for SOIC packages; do not rely on cheap generic clips that may slip or have poor contacts.

![Generic black CH341 (unsafe)](https://private-user-images.githubusercontent.com/827570/291431796-1749cef0-3446-4c85-b5c8-0d902fb1915c.jpeg)

*Caption: Generic black CH341 modules; often output 5V and can damage chips. Do not buy or use.*

**Side-by-side: black CH341 vs CH341A 1.7 (with regulator):**

![CH341 black vs CH341A 1.7](https://private-user-images.githubusercontent.com/827570/296805149-f29c0092-add9-4a78-9e6c-40d1407e0e19.jpeg)

**CH341A 1.6+ differentiators (what to look for):**

![CH341A 1.6+ differentiators](https://private-user-images.githubusercontent.com/827570/296798854-1b2621f6-978e-4b75-9c32-30647f741b09.jpeg)

**Quick checklist**
- Look for a physical voltage selector (1.8V/3.3V/5V) or a regulator circuit near power pins.
- If it looks like the generic black unit above (no selector/regulator), **do not buy it**.
- Prefer Tigard or CH341A rev 1.6+ with selector for safety.

For additional photos and community timing measurements, see issue #120: https://github.com/linuxboot/heads-wiki/issues/120

For a simple clip-orientation SVG used by Skulls, see: https://gist.github.com/Thrilleratplay/60dff7cf849de5bdb84d91cabbd4fcfd

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

### Beginner equipment checklist
- Programmer (recommended: **Tigard**; budget: **CH341A rev1.6+** or **CH347F** with VIO support)
- SOIC clip (recommended: **Pomona 5250**) for SOIC chips
- WSON8 probe or spring-loaded adapter for WSON packages
- Jumper leads and short USB cable
- Second computer with `flashrom` installed
- `hexdump`, `diff`, and basic Unix tools for sanity checks
- Magnifier/microscope, isopropyl alcohol, flux, and basic hand tools (screwdrivers, spudger)

### Voltage Selection Guidelines (beginner-friendly)
- **Stop if unsure**: If you cannot identify the chip or measure its VCC, do **not** perform in‑place programming — ask for help or remove the chip for external programming. ⚠️
- **Use a safe programmer**: Prefer a programmer with reliable voltage control that explicitly supports 3.3V and does not drive 5V by default (examples: **Tigard**, **CH341A rev1.6+**, **CH347**). 🔧
- **Power checklist (always)**: Remove batteries, disconnect AC adapter, and (if comfortable) disconnect the CMOS battery before attaching the clip. 🔌
- **If you can identify the chip**: Follow the chip datasheet for the correct VCC. If you can safely measure the chip’s VCC with a multimeter, use that voltage.

> Tip: Guessing voltages (e.g., "try 1.8V then 3.3V") can damage some chips. If you are not comfortable identifying/measuring VCC, stop and ask for help.

### Advanced (for experienced users)
If you are experienced and must proceed, follow these steps:
1. **Identify the chip by marking or part number** and consult the datasheet for VCC.
2. **Measure board VCC (VCC = chip voltage)** at the chip's VCC pin with a multimeter if possible to confirm the operating voltage. If you're not comfortable measuring, follow the chip datasheet or ask for help — do not guess.
3. **Use the datasheet-specified VCC** when setting your programmer.
4. **If in doubt, do not perform in‑place programming** — remove the chip or use a powered socket/adapter that isolates the chip from the motherboard.

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

If your goal is to preserve a board's GBE/MAC value, the reliable and recommended approach is to create a custom GBE during the Heads build (see the board's `boards/<boardname>` configuration and the linuxboot/heads build documentation for details). Using `--include`/`--ifd` to selectively write regions is an advanced, internal-only workflow and **should not** be presented as a general recommendation for external or first-time flashing. Those flags are applicable for internal upgrade workflows (for example, when using an internal programmer such as `flashprog --programmer internal`) where the operator has precise knowledge of the region layout and the risks involved.

For most users: back up the full chip, follow the per-board build instructions to create or preserve a GBE if required, and follow the per-board flashing steps. Advanced, internal upgrades that need selective region updates may use `--ifd`/`--include` as part of an internal-only process—consult the `boards/<boardname>` docs and the SPI Programmer Best Practices guide for details.


### 1. Chip Detection and Backup
```bash
# Test connection and detect chip
sudo flashrom --programmer [programmer]

# Create backup and verify
sudo flashrom --programmer [programmer] --read backup.bin --chip "[chip_model]"
# Quick sanity check: inspect the start of the dump for obvious garbage
hexdump -C backup.bin | head -20
sudo flashrom --programmer [programmer] --verify backup.bin --chip "[chip_model]"
```

### 3. Flash New Firmware
```bash
# Best practice: verify backups before writing and use the generic flash_tool syntax:
#   flash_tool [read|write|verify] romname-for-platform
# Note: 'flashrom' and 'flashprog' are commonly used tools and can be used interchangeably.
# Example (read): sudo flashrom --programmer [programmer] --read backup.bin --chip "[chip_model]"
# Example (write): sudo flashrom --programmer [programmer] --chip "[chip_model]" --write firmware.rom
```

## Troubleshooting

### Common Issues and Solutions

**Chip not detected**:
- Check clip connection and orientation
- Try different voltage (1.8V → 3.3V)
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
- Verify electrical contact before committing to a write: take a `flashrom --programmer [programmer] --read backup.bin`, inspect the start (`hexdump -C backup.bin | head -20`) and verify with `--verify` if possible.
- Ensure you have good magnification and lighting when positioning the probe, and consider using flux or contact cleaners if multiple attempts fail.

![WSON8 Probe](https://github.com/user-attachments/assets/ebcd780b-c7db-466a-91ea-a0d9d546b3ec)

For community photos and examples of spring-guided probes and jigs, see issue #120: https://github.com/linuxboot/heads-wiki/issues/120

## Performance Optimization

### Tigard Speed Optimization
- Use `divisor=4` for reliable operation
- Use `divisor=2` for maximum speed (may be less stable)
- Enable progress indicator: `--progress`

### CH341A Optimization
- Ensure clean power supply
- Use shorter USB cable
- Keep programmer cool during long operations

## Advanced Features

### Tigard Logic Analysis
Tigard can function as a logic analyzer for debugging SPI communication issues.

## Recommended Kits

### Complete Kits (Assembled and Ready)
- **Novacustom**: [Modified CH341A kit](https://novacustom.com/product/modded-ch341a-bios-firmware-programmer-3v/)
- **3mdeb**: [CH341A programmer kit](https://shop.3mdeb.com/shop/modules/ch341a-flash-bios-usb-programmer-kit-soic8-sop8/)

### Individual Components
- **Tigard**: [Crowd Supply](https://www.crowdsupply.com/securinghw/tigard)
- **WSON8 probe**: [Amazon](https://www.amazon.ca/dp/B0DJ8XTCKD)

## Summary

For most users, **Tigard is the recommended choice** due to its:
- Fast operation (saves time during development)
- Multiple voltage options (safe for all chips)
- Additional debugging capabilities
- Excellent reliability record

For budget-conscious users, **CH341A rev 1.6+ is acceptable** but:
- Verify it has voltage selector switch
- Expect longer flash times
- Limited to basic SPI operations

**Never use old CH341A programmers** without voltage selection - they can damage your hardware.

---

*This guide is based on extensive community testing and feedback. For specific platform flashing instructions, refer to the individual device guides.*
