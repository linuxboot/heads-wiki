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

Historical: Migrating from Very Old Firmware
---

**Note**: If you are upgrading from very old Heads firmware that predates the 2024 standardization, please refer to the [Historical Legacy Migration](/Historical-Legacy-Migration) page for detailed information and migration steps.

All current Heads boards use modern architecture, and new installations should use current board configurations.

---

Re-Owning the States
===
After upgrading or migrating, follow these steps to re-own the security components:
- Inject your public key or perform a Factory Reset/Re-ownership if asked upon external upgrade. Internal upgrade will migrate trustdb and keyring automatically. 
- Generate new TOTP/HOTP tokens when asked, and reset the TPM if necessary. 
- Generate`/boot` hashes and sign content from Options menu and set a new boot default and optionally set a TPM Disk Unlock Key (DUK) passphrase. 

For more details on re-owning, refer to the relevant sections above.

---

Verify Upgradeability Paths of the Firmware
====
First, verify the [supported platforms]({{ site.baseurl }}/Prerequisites#supported-devices).  

**For ThinkPad users (xx30/xx20 families)**: If you are upgrading from very old firmware that predates 2024, proceed with extra caution and ensure you have recovery equipment available.

*Select the HOTP variant if you own a [Heads-supported USB Security dongle]({{ site.baseurl }}/Prerequisites#usb-security-dongles-aka-security-token-aka-smartcard) for visual remote attestation.*

Review whether the Intel Firmware Descriptor (IFD) and Intel Management Engine (ME) regions are unlocked from the [Recovery Shell]({{ site.baseurl }}/RecoveryShell):

```shell
```shell
flashprog -p internal
```

**Note**: On newer Heads firmware prefer `flashprog -p internal`; on older firmware use `flashrom --programmer internal`.

```

**Note**: On older Heads firmware (pre-2025), use `flashrom -p internal` if `flashprog` is not available. Newer firmware versions use `flashprog` as the flash tool.

The output will show whether you can upgrade internally or need external flashing.

Unlocked IFD and ME (Modern Firmware)
----
This is the expected output for modern Heads firmware with unlocked IFD and ME regions:
![CanBeFlashedToMaximizedRom](https://user-images.githubusercontent.com/827570/167728631-85a5ca9e-48f6-4d4f-8544-532fa75bf5d3.jpeg)
- If unlocked, you can internally migrate to a Maximized ROM from the recovery shell. If you are already running a Maximized board ROM, you can upgrade through the Heads GUI while keeping existing settings.

Example internal upgrade (Recovery Console):
- mount-usb
- flashrom --programmer internal --write /media/heads-hotp-maximized-version-gcommit.rom



- **Current firmware**: You can safely upgrade through the Heads GUI using `.zip` files
- **Very old firmware**: If upgrading from pre-2024 firmware, you may need to manually flash:
  - `mount-usb`
  - `flashprog -p internal -w /media/heads-current-version-gcommit.rom` (or `flashrom` on older firmware)
  - ![InternalUpgradeToMaximizedROM](https://user-images.githubusercontent.com/827570/167729694-6ff8da60-986a-4ec3-9b2d-4fa94e42d3fa.jpeg)

Locked IFD and ME (Very Old Firmware)
----
If you see this output, your firmware has locked IFD/ME regions:
![CantBeInternallyUpgradeToMaximizedROM](https://user-images.githubusercontent.com/827570/167728658-731362da-a676-4610-becb-ff94f2ff48b1.jpeg)

- This indicates very old firmware that requires external flashing
- You will need to externally reflash using current board ROM files:
  - xx30 family: Use the appropriate `.rom` files (typically split into 4MB + 8MB images)
  - xx20 family: Use the appropriate single `.rom` file
- **Warning**: External flashing is required - internal flashing will not work

**Historical Note**: Legacy board variants were [officially deprecated in October 2024](https://github.com/linuxboot/heads/pull/1803). All current boards use modern architecture.

---

Notes:
- Since commit [6873df60](https://github.com/linuxboot/heads/commit/6873df60c1c965ac812a49d9d245f338d8a3b128): Users with this or newer firmware can upgrade with `.zip` files. These automatically verify ROM integrity and flash the contained firmware if valid.
- Since commit [133da0e4](https://github.com/linuxboot/heads/commit/133da0e48e2996674f60f186c520cfad0d4848d0): Users with a TPM Disk Unlock Key (DUK) defined previously will be guided to reseal a new passphrase when resealing TOTP/HOTP automatically.
