---
layout: default
title: Upgrading Heads
permalink: /Updating/
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


![Flashing Heads on an x230 at HOPE]({{ site.baseurl }}/images/Flashing_Heads_on_an_x230_at_HOPE.jpg)


Acessing Heads Recovery shell
===

![Recovery shell](/images/Recovery_shell.jpg)

If the flash protection bits are set correctly it is not possible to
 rewrite the firmware from the normal OS.  You'll need to reboot
 to the Heads [recovery shell](/RecoveryShell/).

Repeatedly press 'r' on boot or choose 'Exit to Recovery Shell' from Heads menus.


Reflashing the same firmware
===
If you reflash the same firmware image by selecting the retain settings option from Heads GUI, your 
 TOTP/HOTP/TPM Disk Unlock Key and passphrase will stay valid since measurements will repopulate PCRs
 exactly the same way, resulting in the same sealed secrets.

So if you are in doubt of the current firmware state, you can consequently reflash the firmware image 
 to validate it's current integrity. It is a good idea to keep last flashed firmware image on a USB
 drive.

Bonus, Maximized ROMs reflash the whole SPI flash; not just the BIOS region like Legacy ROMs do.


Upgrading Heads
===

The first time you install Heads, you'll need [some SPI programmer](/Prerequisites#required-equipment) 
 to be able to replace the existing vendor firmware.  

Subsequent upgrades can be performed internally through Heads menus although you'll probably 
 want a hardware programmer since we don't have a fail-safe recovery mechanism in the event of
 a bad flash or buggy firmware.

Additionally, *upgrading the firmware will change the [TPM PCRs](/Keys/#tpm-pcrs)*.
 This will require resealing TOTP/HOTP and to setup a new TPM Disk Unlock Key and passphrase
 by setuping a new boot default.  

Be sure you have your TPM owner's password and your Disk Recovery Key passphrase available 
 since, by design, the TPM Disk Unlock Key will become invalid. Also note that TPM PCRs are 
 extended by going to the recovery shell anyway, which does not permit to have access to
 sealed secrets from there.


Verify upgradeability paths of the firmware
====

It is a good time to review if the Intel Firmware Descriptor (IFD) and
 Intel Management Engine (ME) were unlocked or not. 

```shell
flashrom -p internal
```


Unlocked IFD and ME
----
This is the expected output if the initial external flashing of the firmware unlocked IFD and ME regions:
![CanBeFlashedToMaximizedRom](https://user-images.githubusercontent.com/827570/167728631-85a5ca9e-48f6-4d4f-8544-532fa75bf5d3.jpeg)
- This means you can internally migrate from Legacy boards (xxxx-hotp/xxxx boards ROM) to their Maximized boards counterpart (xxx-hotp-maximized/xxxx-maximized boards ROM).
  - If you are presently on a Legacy board (If Heads boot screen is not showing Maximized):
    - You will have to manually call flashrom from the Recovery Console:
    - ![InternalUpgradeToMaximizedROM](https://user-images.githubusercontent.com/827570/167729694-6ff8da60-986a-4ec3-9b2d-4fa94e42d3fa.jpeg)
- If you are already running a Maximized board ROM, you can safely upgrade through Heads GUI keeping your current settings. 


Locked IFD and ME
----
Otherwise, initial external flashing of the firmware didn't unlock the IFD/ME regions:
![CantBeInternallyUpgradeToMaximizedROM](https://user-images.githubusercontent.com/827570/167728658-731362da-a676-4610-becb-ff94f2ff48b1.jpeg)
- This means you either have to:
  - Externally reflash Maximized board ROM 
    - xx30: xx30-*maximized-top.rom(4Mb) and xx30-*maximized-bottom.rom(8Mb) ROMs 
    - xx20: xx20-*-maximized.rom (8Mb ROM)
  - Stay with Legacy board ROMs. This means you can safely update through Heads GUI, keeping your current settings.


Internal Flashing
===

On the laptop used to build/download Heads 
---
For safety, list the drives available without plugging your USB drive:
```shell
sudo fdisk -l
```


Plug your USB flash drive and redo last command to witness appearance of a new drive:

```shell
sudo fdisk -l
```

Note that Heads currently supports ext3/ext4/fat32 filesystems, where fat32 limits 
 file size to a maximum of 4Gb. So if you intend to use that USB drive to host bigger
 files, you should probably backup current content and format to ext3/ext4.
 Otherwise, fat32 is fine now to put only firmware image.

If you want to reformat your usb drive as ext4 (USB drive is /dev/sdb here):

```shell
sudo mkfs.ext4 /dev/sdb1
```

These are the commands I used to create a directory ~/usb/ and mount my usb
 drive there, but you can mount it wherever you want:

```shell
mkdir ~/usb
sudo mount /dev/sdb1 ~/usb/
```

Move the full Heads rom file to the usb drive and unmount the drive:

```shell
sudo cp ~/heads/build/x230/heads.rom ~/usb/
sudo umount /dev/sdb1
```


Flashing new firmware under Heads
---
As discussed above: 

- If you are not migrating from Legacy boards to Maximized board configurations, 
 you can safely upgrade from Heads GUI through `Options->Flash/Update BIOS`
 and choose the retain settings option. This will copy your GPG keyring and user configuration
 into the new ROM prior of flashing it the whole combined 12Mb SPI flash with the ROM.

- If you are migrating from Legacy to Maximized ROM, you need to manually call flashrom
 from the Recovery Shell, having a copy of your public key on a USB drive to inject it back
 on next Heads boot.


Re-Owning the states
===
Reboot and verify that the new firmware is running. Don't be scared if you have to power off twice
 The trained memory cache in firmware might have been wiped and reconstructed. Hold the power
 button for 10 seconds and power back on.

- If you migrated from Legacy to Maximized builds (no migration of settings), you will
 be prompted on next reboot by the same prompts following an initial flash. That is:
  - To inject your public key or do OEM Factory Reset/Re-Ownership
    - The Factory Reset/Re-Ownership option will guide you into re-owning all security components
     including resetting USB Security dongle, injecting public key in ROM and signing /boot.
  - Then on next reboot, you will be prompted to generate new TOTP/HOTP token. Normal, since none
   of the previous measurements are valid anymore (GPG Admin PIN and TPM Ownership passphrase required)
  - Sign /boot content (GPG User PIN required)
  - Select a new boot default through Boot Options (GPG User PIN required to sign the new default)
    - Optionally set a TPM Disk Unlock Key (Disk Recovery Key passphrase and GPG User PIN required)

- If you upgraded your firmware by choosing the retain settings options for a same board configuration 
  - The same steps above will be required, outside of the public key injection/Re-Ownership.
