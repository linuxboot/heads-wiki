---
layout: default
title: Historical Legacy Migration
permalink: /Historical-Legacy-Migration
nav_order: 99
parent: About
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

Historical Legacy Migration
===

**Note**: This page contains historical reference information and applies only to users upgrading from very old Heads firmware (pre-2024). All new installations should use current board configurations available from the [Prerequisites](/Prerequisites) page.

**Warning**: Migration from pre-2024 firmware requires careful attention. Legacy boards were [officially deprecated in October 2024](https://github.com/linuxboot/heads/pull/1803). Use maximum caution when upgrading from very old firmware, as improper steps may result in a bricked device.

Historical Background
---

Originally, Heads supported two types of board configurations:

- **Legacy boards**: Produced incomplete ROM images that required specific flashing procedures and only worked with locked IFD/ME regions
- **Maximized boards**: Produced complete ROM images with neutered/deactivated ME and unlocked IFD, allowing full internal flashing

Legacy boards were [officially deprecated in October 2024](https://github.com/linuxboot/heads/pull/1803), and all current boards now use the approach that was previously called "maximized."

Identifying Very Old Firmware
---

If you are running very old Heads firmware:

1. Boot into the Heads main menu
2. Check if the main screen title includes "Maximized" in the board name
   - If it does NOT include "Maximized," you may be on very old firmware
   - Examples of old firmware: `x220-legacy`, `x220`, `t430`, `x230-hotp` (without "Maximized")
   - Current firmware: All board names are standardized and use modern architecture

Migration Steps
---

1. Refer to the [Downloading documentation](/Downloading) for current board configurations
2. Follow the [upgrade verification steps](/Updating#verify-upgradeability-paths-of-the-firmware) in the Upgrading documentation
3. Ensure you have proper recovery equipment before proceeding

**Important for Nitrokey Customers**:  
If you are using a NitroPad X230 or T430 with firmware version **1.2 or earlier**, contact [Nitrokey support](https://support.nitrokey.com/) for their flashing service. See this [support thread](https://support.nitrokey.com/t/nitropad-t430-firmware-update-brick/3777/2) for details.

Technical Details (Historical)
---

The historical approach involved:

- **ME neutering/deactivation**: For older Intel platforms (up to Ivy Bridge), reducing the Intel Management Engine to only essential components (BUP and ROMP for xx30 family, BUP only for xx20). For newer platforms, ME is deactivated using HAP bits or other methods
- **IFD unlocking**: Allowing the Intel Flash Descriptor regions to be modified
- **Space optimization**: Reclaiming freed ME space for the BIOS region (expanding from ~7MB to ~11.5MB on x230 family)

This approach is now standard for all Heads boards, adapted to each platform's capabilities.

Legacy Board Details (Historical Reference)
---

### Legacy Board Types (Deprecated)

**xx30-flash**: 4MB ROM images to be flashed internally through 1vyrain or Skulls. Unlocking IFD and cleaning/neutering ME still highly recommended prior of doing initial flash. 1vyrain deactivates ME internally. But if one leaves 1vyrain and chooses another ROM, 1vyrain applied hacks to deactivate ME will not be applied anymore. Note that Skulls permits to unlock IFD as an option prior of initial flash. So if it was not applied at that step, then the end user can only upgrade to Legacy boards produced ROMs in the future, the IFD and ME not being flashable internally and requiring an external flash to go with Maximized boards.

**xx30**: Baked 12MB ROM images to be flashed internally through xxxx-flash flashed ROMs. Those ROMs can only be internally flashed from/to legacy boards configuration. Flashing a legacy ROM from a Maximized ROM will result in a brick, since Maximized boards produced ROMs will flash the whole combined opaque 12MB ROM internally, overwriting IFD, ME and GBE with empty content. Resulting into a brick.

**xx20**: Those ports (t420 and x220 at time of writing) landed on Heads later in time and were historically produced by making required blobs available by applying scripts on SPI backups to extract required blobs. Consequently, those boards do not suffer from feature reduction as of now; they always took for granted that ME was neutered and IFD was unlocked. They still only flash internally the BIOS region, which was maximized to take advantage of 7.5MB available SPI space for BIOS region, while not reflashing the whole SPI.

**xxxx-hotp-verification**: Legacy, reduced versions of their HOTP maximized counterpart. At the time of writing this, those board configuration will normally loose dropbear support, while xx30 versions will not have FBWhiptail anymore. That means that there is no framebuffer enforced graphical interface under Heads with background color cues notifying the end user of warning (yellow) or errors (red) contextual, graphical menus.

Legacy to Maximized Upgrade Path (Historical)
---

It was possible to upgrade from Legacy to Maximized boards under certain conditions:

- **If you come from 1vyrain, this was impossible**. 1vyrain does not unlock neither IFD nor ME regions of the SPI. Consequently, flashing internally anything else then Legacy boards produced ROMs would result in a brick.

- **If coming from Skulls**, if and only optional unlocking step has been followed, you could upgrade internally through a manual flash tool call (`flashprog` on newer firmware or `flashrom` on older firmware), just like if you were coming from Heads Legacy boards while having followed the me_cleaning page instructions prior of initial flash.

- **If coming from Skulls or Heads Legacy board configurations** while having unlocked IFD initially, you could flash from the recovery shell manually.

### Manual Upgrade Process (Historical)

Having a full xxxx-hotp-maximized or xxxx-maximized board config produced ROM available on a USB stick, alongside with your USB Security dongle's matching exported public key, the process was:

```
mount-usb
flashprog -p internal -w /media/PathToMaximizedRom.rom
```

on board with Intel based Ethernet you might want to use: 

```
sudo flashrom -p internal --ifd -i bios -i me -i fd -w /media/PathToMaximizedRom.rom
```

to perserve the orignal mac adresse

**Note**: Use `flashprog` on newer Heads firmware (2025+) or `flashrom` on older firmware versions, depending on what is available in your Heads system.

On next reboot, Heads would guide you into factory resetting your USB Security dongle or import your previously generated public key matching your USB Security dongle's private key.

It would then regenerate a TOTP/HOTP secret and sign /boot content. You would then have to define a new default boot and optionally renew/change your Disk Unlock Key to be released to to OS to unlock your encrypted OS installation to move forward.

In the case nothing was found installed on your disk, Heads would propose you to boot from USB to install a new Operating System, prior of being able to do the above steps prior of booting into your system.
