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
- Remove battery
- Disconnect AC adapter
- Disconnect CMOS battery (recommended for extra safety)

**Never connect or disconnect the clip while the programmer is powered on.** This can damage your motherboard or SPI chip.

## Recommended Programmers (in order of preference)

Based on extensive community testing and feedback, here are the recommended SPI programmers:

### 1. **Tigard (Recommended)**
- **Cost**: $67-89 USD
- **Speed**: ~21 seconds for 8MB chip, ~42 seconds for 16MB chip
- **Voltage**: 1.8V, 3.3V, 5V, external target voltage
- **Protocols**: SPI, I2C, JTAG, SWD, UART
- **Additional features**: Logic analyzer, USB debugging support
- **Reliability**: Excellent (4/4 success rate in community testing)
- **Best for**: Development, debugging, frequent flashing, new users

**Where to buy**: [Crowd Supply](https://www.crowdsupply.com/securinghw/tigard)

### 2. **CH341A Rev 1.6+ (Budget option with voltage selector)**
- **Cost**: $5-15 USD
- **Speed**: ~120 seconds for 8MB chip
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
- **Speed**: ~30 seconds for 16MB chip
- **Voltage**: 3.3V (configurable via firmware)
- **Protocols**: SPI (via serprog firmware)
- **Reliability**: High
- **Best for**: DIY enthusiasts, custom setups

### 4. **CH347 (Newer alternative to CH341A)**
- **Cost**: $5-15 USD
- **Speed**: ~60 seconds for 16MB chip (significantly faster than CH341A)
- **Voltage**: 3.3V only (5V tolerant inputs)
- **Protocols**: SPI, I2C
- **Safety**: Safer than old CH341A (no dangerous 5V output)
- **Best for**: Budget option with better performance than CH341A

**Note**: The CH347 is a newer alternative to CH341A with proper 3.3V operation and faster speeds. However, it lacks voltage selection for 1.8V chips.

## Programmer Comparison Table

| Programmer | Cost | Speed (16MB) | Voltage Selection | Protocols | Debugging Features | Reliability |
|------------|------|--------------|-------------------|-----------|-------------------|-------------|
| **Tigard** | $67-89 | ~42 seconds | 1.8V, 3.3V, 5V, external | SPI, I2C, JTAG, SWD, UART | Logic analyzer, USB debugging | Excellent |
| **CH341A rev 1.6+** | $5-15 | ~120 seconds | 1.8V, 2.5V, 3.3V, 5V | SPI, I2C | None | Good |
| **Raspberry Pi Pico** | $4-10 | ~30 seconds | 3.3V (configurable) | SPI | None | High |
| **CH347** | $5-15 | ~60 seconds | 3.3V only | SPI, I2C | None | Good |
| **CH341A old** | $2-10 | ~480 seconds | 5V (dangerous) | SPI, I2C | None | **Not recommended** |

## Essential Safety Procedures

### Before Starting
1. **Power disconnection checklist**:
   - [ ] Remove laptop battery
   - [ ] Disconnect AC adapter
   - [ ] Disconnect CMOS battery (recommended)
   - [ ] Wait 30 seconds for capacitors to discharge

2. **Workspace preparation**:
   - Use anti-static wrist strap
   - Work on anti-static mat
   - Ensure good lighting
   - Have magnifying glass available

### Voltage Selection Guidelines
1. **Start with 1.8V**: Most modern SPI chips prefer 1.8V
2. **Try 3.3V**: If chip detection fails at 1.8V, try 3.3V
3. **Never use 5V**: Unless specifically required by chip datasheet
4. **Test detection**: Run `flashrom -p [programmer]` to verify chip detection

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
sudo flashrom -p ft2232_spi:type=2232H,port=B,divisor=4

# With chip specification
sudo flashrom -p ft2232_spi:type=2232H,port=B,divisor=4 -c "MX25L6405D"
```

### Using CH341A rev 1.6+
```bash
# Basic connection test
sudo flashrom -p ch341a_spi

# With chip specification
sudo flashrom -p ch341a_spi -c "MX25L6405D"
```

## Flashing Workflow

### 1. Chip Detection and Backup
```bash
# Test connection and detect chip
sudo flashrom -p [programmer]

# Create backup (run twice to verify)
sudo flashrom -r backup1.bin -p [programmer] -c "[chip_model]"
sudo flashrom -r backup2.bin -p [programmer] -c "[chip_model]"

# Verify backups match
diff backup1.bin backup2.bin
```

### 2. Verify Backup Content
```bash
# Check backup contains data (not all 0x00 or 0xFF)
hexdump -C backup1.bin | head -20
```

### 3. Flash New Firmware
```bash
# Flash with retry loop (recommended)
false; while [ $? != 0 ]; do 
    sudo flashrom -p [programmer] -c "[chip_model]" -w firmware.rom
done
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
- Use specialized WSON8 probe with alignment guide
- Apply gentle downward pressure during operation
- Consider using flux for better electrical contact

![WSON8 Probe](https://github.com/user-attachments/assets/ebcd780b-c7db-466a-91ea-a0d9d546b3ec)

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