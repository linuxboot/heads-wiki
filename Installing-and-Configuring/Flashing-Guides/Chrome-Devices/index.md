---
layout: default
title: Chrome Devices
permalink: /flashing-guides-chrome-devices/
nav_order: 3
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
has_children: yes
---

Chrome Devices
===
There are a lot of Chrome devices (Chromebooks and ChromeBoxes) that have MrChromebox's Coreboot support.
Most of these devices can support Heads as well.  This guide is a general guide with common steps that
apply to all Chrome devices.

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## ⚡ Safety First

**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

## Summary
At a high level we will be doing the following:

1. Enable closed case debugging (CCD)
2. Disable hardware write protection
3. Disable software write protection

Then you can follow the individual device guides on how to backup, and then flash Heads.

## Closed Case Debugging
With Chrome devices there is an option called "Closed Case Debugging" or CCD.  With CCD
You can flash the rom without having to open the device.  A special usb cable called a SuzyQ cable
is required.  A good guide on using the SuzyQ cable is 
[MrChromebox](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html).

Here are 2 places where you can find info on purchasing a SuzyQ board:

1. [ChocalateLoverRaj Github](https://github.com/ChocolateLoverRaj/gsc-debug-board)
2. [FyraLabs Github](https://github.com/fyraLabs/suzyqboard)

### Enabling Closed Case Debugging (CCD)
There are 2 ways to enable CCD.  One is by using the ChromeOS Developer mode console, this method
requires the stock ChromeOS bootloader, but no SuzyQ board. The 2nd option is using a SuzyQ board
to send commands to the GSC/CR50 console

#### ChromeOS Developer Mode Method

You will be using `gsctool` in the ChromeOS Developer mode console.

1. Power on the device to the login screen (booted into Developer Mode).
2. Open VT-2 terminal: press [CTRL+ALT+F2] (F2 is the right arrow).
3. Login as `root`
4. Open the CCD: run `gsctool -a -o`
5. You will be prompted to press the PP (physical presence) button several times. On almost all devices, this means to press the power button. Opening the CCD requires you to press the PP button several times over a 2-3 minute period.
6. When the open CCD process is complete, you will see a message showing `PP Done!` and the device will reboot in Normal/Verified Boot Mode.
7. Re-enable developer mode and continue with the instructions below.

[MrChromebox source](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html#step-1-enabling-closed-case-debugging-ccd)

#### SuzyQ and CR50 Console Method
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

2. Check if CCD is already Unlocked.
  ```
  # Check the baud rate
  sudo stty -F /dev/ttyUSB0

  sudo minicom -D /dev/ttyUSB0 -b <baud_rate from above>
  < Press Enter >
  ccd
  ```

  If CCD is locked it will show:

  ```
  State: Locked
  Password: none
  Flags: 0x000000
  Capabilities: 0000000000000000
    UartGscRxAPTx   Y 0=Default (Always)
    UartGscTxAPRx   Y 0=Default (Always)
    UartGscRxECTx   Y 0=Default (Always)
    UartGscTxECRx   - 0=Default (IfOpened)
    FlashAP         - 0=Default (IfOpened)
    FlashEC         - 0=Default (IfOpened)
    OverrideWP      - 0=Default (IfOpened)
    RebootECAP      - 0=Default (IfOpened)
    GscFullConsole  - 0=Default (IfOpened)
    UnlockNoReboot  Y 0=Default (Always)
    UnlockNoShortPP Y 0=Default (Always)
    OpenNoTPMWipe   - 0=Default (IfOpened)
    OpenNoLongPP    - 0=Default (IfOpened)
    BatteryBypassPP Y 0=Default (Always)
    Unused          Y 0=Default (Always)
    I2C             - 0=Default (IfOpened)
    FlashRead       Y 0=Default (Always)
    OpenNoDevMode   - 0=Default (IfOpened)
    OpenFromUSB     - 0=Default (IfOpened)
    OverrideBatt    - 0=Default (IfOpened)
    APROCheckVC     - 0=Default (IfOpened)
  read_tpm_nvmem: object at 0x100a not found
  [906.873298 Console unlock allowed]
  read_tpm_nvmem: object at 0x1007 not found
  TPM:
  Capabilities are default.
  Use 'ccd help' to print subcommands
  ```

3. Unlock CCD
  ```
  ccd open
  ```

## Disable Hardware Write Protection
We now need to disable hardware write protection so we can flash the rom.

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
6. Run `gsctool -a -I` to verify the CCD is opened, and that the factory values are set. The current value for all CCD flags should be set to Y/Always.
7. Run `crossystem wpsw_cur` and verify it returns 0.
8. Reboot.

[MrChromebox source](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html#step-2-disabling-write-protection)

### Disable Software Write Protection
A good guide on how to do this is [here](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html#disabling-software-write-protection)

We will be using the SuzyQ device to disable Software write protection. 
If you don't already have that plugged in follow the steps above to get
the SuzyQ cable plugged in correctly.

Note: Some devices have special programmer parameters, if the below commands
do not work for you, go to the specific device page to see if your device has
different options.

1. Check if write protection is enabled:
   ```
   flashrom --programmer raiden_debug_spi:target=AP \
     --wp-status
   ```
2. Disable software write protection:
   ```
   flashrom --programmer raiden_debug_spi:target=AP \
     --wp-disable
   ```
3. Changes software write protection addresses range:
   ```
   flashrom --programmer raiden_debug_spi:target=AP \
     --wp-range 0,0
   ```
   (`--wp-range 0 0` on older devices)
