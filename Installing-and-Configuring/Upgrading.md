---
layout: default
title: Upgrading Heads
permalink: /Updating
nav_order: 99
parent: Installing and configuring
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


![Flash_gui-init](https://user-images.githubusercontent.com/827570/204049689-ec4af8b7-5cc3-46f5-b4fe-bcbc47d61d34.jpeg)


Upgrading Heads
===

The first time you install Heads, you'll need [some SPI programmer]({{ site.baseurl }}/Prerequisites#required-equipment)
 to replace the existing vendor firmware. This process involves **initial external flashing** using `.rom` files.

For subsequent upgrades, you can use the Heads GUI to perform **internal upgrading** using `.zip` files. This method is supported for firmware versions built after [this November 2023 commit](https://github.com/linuxboot/heads/commit/6873df60c1c965ac812a49d9d245f338d8a3b128).

---

Global Warning: No Hardware Programmer? Be Cautious!
---
Heads is a rolling release. If you do not have an external programmer to recover from potential issues, exercise caution when upgrading. Follow these guidelines:

1. **Wait Before Upgrading**:
   - Wait a day or two after a new commit is merged, especially if it involves coreboot or kernel updates. This allows time for potential regressions to be identified and resolved.

2. **Check Reported Issues**:
   - Review the [most recently reported issues](https://github.com/linuxboot/heads/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc) to ensure no critical problems have been reported for your platform.

3. **Verify Community Testing**:
   - Confirm that other users with the same platform have successfully tested the changes. Heads relies on community testing to validate updates across supported devices.

4. **Understand the Risks**:
   - Major updates, such as coreboot or kernel version bumps, require extensive testing. Untested platforms may experience regressions or even bricking.

5. **Pay Attention to Board Labels**:
   - Boards labeled as `UNTESTED` or `UNMAINTAINED` in their names indicate limited or no support. These labels mean exactly what they state:
     - **UNTESTED**: The board has not been tested and may not function as expected.
     - **UNMAINTAINED**: The board is no longer actively supported or maintained.
   - For additional details, refer to the [UNTESTED and UNMAINTAINED boards documentation](https://github.com/linuxboot/heads/blob/master/unmaintained_boards/README.md).

By following these precautions, you can minimize the risk of encountering issues during an upgrade. If in doubt, wait for additional feedback from the community or consider using an external programmer for recovery.

---

Initial External Flashing
---
If you are installing Heads for the first time, you will need to perform an external flash using `.rom` files. Refer to the [Downloading documentation]({{ site.baseurl }}/Downloading) for detailed instructions on:
- Downloading `.rom` files and `hashes.txt`.
- Verifying file integrity.
- Preparing for external flashing.

**Note**: This process is only required for the initial installation of Heads.

---

Internal Upgrading Using ZIP Files
---
If Heads is already installed on your device, you can upgrade the firmware internally using a `.zip` file. This method is simpler and does not require an external programmer.

1. **Prepare the ZIP File**:
   - Download the `.zip` file for your board from the CircleCI "Artifacts" tab (refer to the [Downloading documentation]({{ site.baseurl }}/Downloading)).

2. **Transfer to Target System**:
   - Save the `.zip` file to a USB drive and insert it into the target system.

3. **Upgrade via Heads GUI**:
   - Boot into the Heads GUI.
   - Navigate to `Options -> Flash/Update BIOS`.
   - Select the `.zip` file from the USB drive. The system will automatically verify the ROM integrity and flash the firmware.

**Note**: Internal upgrading via `.zip` files is supported for firmware versions built after [this November 2023 commit](https://github.com/linuxboot/heads/commit/6873df60c1c965ac812a49d9d245f338d8a3b128).

---

Caution for Rolling Releases
---
Heads is a rolling release. If you do not have an external programmer, wait a few days after a new commit before upgrading to ensure no regressions are reported. Check the [most recently reported issues](https://github.com/linuxboot/heads/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc) and confirm that other users have tested the changes on your platform.

---

Reflashing the Same Firmware
---
If you need to validate the current firmware integrity against the last flashed firmware image stored on a USB thumb drive, follow these steps to back up the current ROM and ensure no changes occur during the process:

1. **Back Up the Current ROM**:
   - Insert a USB thumb drive and mount it in read-write mode:
     ```shell
     mount-usb --mode rw
     ```
   - Back up the current ROM to the USB drive:
     ```shell
     ${CONFIG_FLASH_OPTIONS} -r /media/backup.rom
     ```
   - Remount the USB drive in read-only mode for safety:
     ```shell
     mount -o remount,ro /media
     ```

2. **Reflash the Same Firmware**:
   - Use the Heads Flashing GUI (for zip file downloaded) or the following command to reflash the same firmware image from Heads Recovery shell:
     ```shell
     ${CONFIG_FLASH_OPTIONS} -w /media/same_rom_image_previously_flashed.rom
     ```
   - The process should report **no changes** to the SPI flash.

**Note on `CONFIG_FLASH_OPTIONS`**:
- The `CONFIG_FLASH_OPTIONS` variable specifies the board-specific flash options to ensure proper handling of SPI regions during flashing. These options are defined in the board's configuration file.
- Boards may specify different SPI regions to flash. For example:
  - The `novacustom-v540tu` board preserves the `GBE` (Gigabit Ethernet) region, ensuring the manufacturing MAC address remains intact.
  - The `x230-hotp-maximized` board overwrites the entire SPI flash, including the `GBE` region, replacing it with a generic configuration.
- To inspect the flash options for your board, use the `env` command in the recovery shell:
  ```shell
  env
  ```
  This will display all environment variables, including `CONFIG_FLASH_OPTIONS`, as defined for your board.

**Tip**: Advanced users can review the board-specific configuration files under the Heads repository (`boards/<boardname>/<boardname>.config`) to understand what their board's flash options entail.

---

Migrating from Legacy to Maximized Firmware
---
**Warning**: Migration from Legacy to Maximized builds is now considered a legacy process. Legacy builds were officially deprecated in [October 2024](https://github.com/linuxboot/heads/pull/1803), while Maximized builds have been promoted since [December 2020](https://github.com/linuxboot/heads/pull/703). Use maximum caution when upgrading internally to Maximized builds, as improper steps may result in a bricked device.

### How to Identify Legacy vs Maximized Builds
To determine whether you are using a Legacy or Maximized build:
1. Boot into the Heads main menu.
2. Check the main screen title:
   - If the build is **Maximized**, the label will explicitly include the word "Maximized."
     - Example: `x230-hotp-maximized`
   - If the label does not include "Maximized," you are using a Legacy build.
     - Example: `x220-legacy`, `x220`, `t430`, `x230-hotp` (no "Maximized" in the board name).

For migration instructions, refer to the [Downloading documentation]({{ site.baseurl }}/Downloading) and ensure you follow both the upgradability validation and integrity validation steps before flashing.

**Important Note for Nitrokey Customers**:  
If you are using a NitroPad X230 or T430 with firmware version **1.2 or earlier**, you should contact [Nitrokey support](https://support.nitrokey.com/) to get their flashing service. For more details, refer to this [Nitrokey support thread](https://support.nitrokey.com/t/nitropad-t430-firmware-update-brick/3777/2).

---

Re-Owning the States
===
After upgrading or migrating, follow these steps to re-own the security components:
- Inject your public key or perform a Factory Reset/Re-Ownership.
- Generate new TOTP/HOTP tokens and reset the TPM if necessary.
- Sign `/boot` content and set a new boot default.

For more details on re-owning, refer to the relevant sections above.

---

Verify Upgradeability Paths of the Firmware
====
First, verify the [supported platforms]({{ site.baseurl }}/Prerequisites#supported-devices).  
If you have a *ThinkPad (xx30/xx20 flavors), proceed with caution.*  
*Also select the HOTP variant if you own a [Heads-supported USB Security dongle]({{ site.baseurl }}/Prerequisites#usb-security-dongles-aka-security-token-aka-smartcard) for visual remote attestation.*

Review whether the Intel Firmware Descriptor (IFD) and Intel Management Engine (ME) were unlocked from the [Recovery Shell]({{ site.baseurl }}/RecoveryShell) before proceeding.

```shell
flashrom -p internal
```

The two following situations must apply, which will define what to do next.

Unlocked IFD and ME
----
This is the expected output if the initial external flashing of the firmware unlocked IFD and ME regions:
![CanBeFlashedToMaximizedRom](https://user-images.githubusercontent.com/827570/167728631-85a5ca9e-48f6-4d4f-8544-532fa75bf5d3.jpeg)
- This means you can internally migrate from Legacy boards (xxxx-hotp/xxxx boards ROM) to their Maximized boards counterpart (xxx-hotp-maximized/xxxx-maximized boards ROM).
  - If you are presently on a Legacy board (If Heads boot screen is not showing Maximized):
    - You will have to manually call flashrom from the Recovery Console: 
      - `mount-usb`
      - `flashrom -p internal -w /media/heads-hotp-maximized-version-gcommit.rom`
    - ![InternalUpgradeToMaximizedROM](https://user-images.githubusercontent.com/827570/167729694-6ff8da60-986a-4ec3-9b2d-4fa94e42d3fa.jpeg)
- If you are already running a Maximized board ROM, you can safely upgrade through Heads GUI keeping your current settings. 

Locked IFD and ME
----
Otherwise, initial external flashing of the firmware didn't unlock the IFD/ME regions:
![CantBeInternallyUpgradeToMaximizedROM](https://user-images.githubusercontent.com/827570/167728658-731362da-a676-4610-becb-ff94f2ff48b1.jpeg)
- This means you will have to:
  - Externally reflash Maximized board ROM 
    - xx30: xx30-*maximized-top.rom(4Mb) and xx30-*maximized-bottom.rom(8Mb) ROMs 
    - xx20: xx20-*-maximized.rom (8Mb ROM)

Legacy board variants were [officially deprecated in October 2024 per this PR](https://github.com/linuxboot/heads/pull/1803).

---

Notes:
- Since commit [6873df60](https://github.com/linuxboot/heads/commit/6873df60c1c965ac812a49d9d245f338d8a3b128): Users with this or newer firmware can upgrade with `.zip` files. These automatically verify ROM integrity and flash the contained firmware if valid.
- Since commit [133da0e4](https://github.com/linuxboot/heads/commit/133da0e48e2996674f60f186c520cfad0d4848d0): Users with a TPM Disk Unlock Key (DUK) defined previously will be guided to reseal a new passphrase when resealing TOTP/HOTP automatically.
