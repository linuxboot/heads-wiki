---
layout: default
title: Acer Chromebook Spin 714 (KANO)
permalink: /google-kano-flashing/
nav_order: 12
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
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

## Disassembly

Full disassembly instructions, SPI chip details, programmer recommendations, and step-by-step flashing procedures for the Suzy-Q CCD cable:

### CCD with Suzyq cable
With Chromebooks there is an option called "Closed Case Debugging".  A special usb cable called a SuzyQ cable
is required.  A good guide on using the SuzyQ cable is 
[MrChromebox](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html).

Here are 2 places where you can find info on purchasing a SuzyQ board:

1. [ChocalateLoverRaj Github](https://github.com/ChocolateLoverRaj/gsc-debug-board)
2. [FyraLabs Github](https://github.com/fyraLabs/suzyqboard)

### Enable Enabling Closed Case Debugging (CCD)
You will be using `gsctool` in the ChromeOS Developer mode console.

1. Power on the device to the login screen (booted into Developer Mode).
2. Open VT-2 terminal: press [CTRL+ALT+F2] (F2 is the right arrow).
3. Login as `root`
4. Open the CCD: run `gsctool -a -o`
5. You will be prompted to press the PP (physical presence) button several times. On almost all devices, this means to press the power button. Opening the CCD requires you to press the PP button several times over a 2-3 minute period.
6. When the open CCD process is complete, you will see a message showing `PP Done!` and the device will reboot in Normal/Verified Boot Mode.
7. Re-enable developer mode and continue with the instructions below.

[MrChromebox source](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html#step-1-enabling-closed-case-debugging-ccd)

### Disable Hardware Write Protection
For this section we will be using a laptop running linux (host) and the Kano device (source).

1. Connect the USB-C end of the Suzy-Q cable to the CCD port on your ChromeOS device
   (usually left USB-C port) and the USB-A end to your Linux device
   Verify the cable is properly connected:

   ```
   $ ls /dev/ttyUSB*
   /dev/ttyUSB0  /dev/ttyUSB1  /dev/ttyUSB2
   ```

   This command should return 3 items: ttyUSB0, ttyUSB1, and ttyUSB2.
   If not, then your cable is connected to the wrong port or is upside down.
   Adjust and repeat command until output is as expected.
2. Run the following commands to disable hardware write protection:
   ```
   echo "wp false" > /dev/ttyUSB0
   echo "wp false atboot" > /dev/ttyUSB0
   echo "ccd reset factory" > /dev/ttyUSB0
   ```

3. Power on the Kano device to the login screen (booted into Developer Mode).
4. Open VT-2 terminal: press [CTRL+ALT+F2] (F2 is the right arrow).
5. Login as `root`
6. Run gsctool -a -I to verify the CCD is opened, and that the factory values are set. The current value for all CCD flags should be set to Y/Always.
7. Run crossystem wpsw_cur and verify it returns 0.
8. Reboot.

[MrChromebox source](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html#step-2-disabling-write-protection)

### Disable Software Write Protection
A good guide on how to do this is [here](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html#disabling-software-write-protection)

We will be using the SuzyQ device to disable Software write protection. 
If you don't already have that plugged in follow the steps above to get
the SuzyQ cable plugged in correctly.

1. Check if write protection is enabled:
   `flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True --chip "W25Q256JV_M" --wp-status`
2. Disable software write protection:
   `flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True --chip "W25Q256JV_M" --wp-disable`
3. Changes software write protection addresses range:
   `flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True --chip "W25Q256JV_M" --wp-range 0,0`
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