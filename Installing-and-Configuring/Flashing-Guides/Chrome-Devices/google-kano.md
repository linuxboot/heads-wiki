---
layout: default
title: Acer Chromebook Spin 714 (KANO)
permalink: /google-kano-flashing/
nav_order: 12
parent: Chrome Devices
grand_parent: Step 2 - Flashing Guides
---

Acer Chromebook Spin 714 (KANO)
===

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## ✅ Active: CPU generation still receiving microcode updates (12th Gen Alder Lake)
{: .note }

Still receiving microcode updates.
See [BOARDS_AND_TESTERS.md](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status),
[Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-me-and-peripheral-firmware).

## ✅ PROTECTED: TPM GPIO Reset
{: .warning }

GPIO PLTRST# assertion does not apply to the TPM on this platform. Not vulnerable.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).

## ⚡ Safety First

**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

### Disable Software Write Protection
A good guide on how to do this is [here](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html#disabling-software-write-protection)

We will be using the SuzyQ device to disable Software write protection. 
If you don't already have that plugged in follow the steps above to get
the SuzyQ cable plugged in correctly.

1. Check if write protection is enabled:
   ```
   flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True \
     --chip "W25Q256JV_M" \
     --wp-status
   ```
2. Disable software write protection:
   ```
   flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True \
     --chip "W25Q256JV_M" \
     --wp-disable
   ```
3. Changes software write protection addresses range:
   ```
   flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True \
     --chip "W25Q256JV_M" \
     --wp-range 0,0
   ```
   (`--wp-range 0 0` on older devices)

## Extra step
KANO requires an extra step before we flash with flashrom:

```
# Get the baud rate
sudo stty -F /dev/ttyUSB2
speed 9600 baud; line = 0;

sudo minicom -D /dev/ttyUSB2 -b 9600

run: apshutdown
wait 5s
run: gpioset en_S5_rails 1
```

[source](https://forum.chrultrabook.com/t/flashrom-error-when-trying-to-unbrick-with-suzyq/8789/2?u=stonework5729)

## Backup
Before you flash Heads you should take a backup of the bootloader.

```
# 1. Read the full chip backup
flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True \
  --chip "W25Q256JV_M" \
  --read kano_backup.rom

# 2. Inspect the dump (sanity check)
hexdump -C backup1.bin | head -20

# 3. Verify the backup reads back correctly
flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True \
  --chip "W25Q256JV_M" \
  --verify kano_backup.rom

# 4. Read a second backup to a different file (paranoia is good)
flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True \
  --chip "W25Q256JV_M" \
  --read kano_backup2.rom

flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True \
  --chip "W25Q256JV_M" \
  --verify kano_backup2.rom

```

More info in the [SPI Programmer Best Practices](https://osresearch.net/SPI-Programmer-Best-Practices/#backup--verify)

## Flashing

```
flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True \
  --chip "W25Q256JV_M" \
  --write heads-kano-202607081551-v0.2.1-3112-gb9dfd39.rom
```
