---
layout: default
title: Step 3 - Configuring-Keys
permalink: /Configuring-Keys/
nav_order: 7
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

No OS installed
===

If starting with a blank hard drive, Heads will detect it and propose the
 to boot from USB alongside with other options:
![IMG_20720216_035536](https://user-images.githubusercontent.com/827570/168883552-58dfb283-52b1-4026-9ae3-ae962dfb0672.JPG)
 
You can also set a different boot device, or go into Recovery shell and partition
 with `fdisk`. 
 
 Installing an OS by booting from USB is probably the best option.
 Note here again that for OSes that provide detached signatures 
 (iso.asc or iso.sig) alongside of their ISO download option, this is the preferred
 Heads installation method, since Heads can validate both integrity and authenticity
 against distribution public key, fused (and measured) in the firmware.
 
If there are other OSes providing detached signatures alongside of their ISOs, please
open an issue so we can add it. 

No public key found in ROM
===
At early Heads boot, you will see the following:
![IMG_20720216_035524](https://user-images.githubusercontent.com/827570/168883785-a94c77dc-0743-4622-83cf-62bbf8024462.JPG)

You can either add a backuped gpg public key matching an already provisioned USB Security dongle (see below on adding public key)
or generate the keys, alongside as setting all security components in one go with the `OEM Factory Reset/Re-Ownership` option.

OEM Factory Reset/Re-Ownership
---
Once you have installed the OS, you will need to take ownership of the different security
components under Heads. The easiest way is through `OEM Factory Reset/Re-Ownership` option.

[EFF Diceware passphrases are recommended](https://www.rempe.us/diceware/#eff)
 
The Secrets involved under Heads are the following (and their recommended lengths):
- Disk Recovery Key passphrase (6 words. *Do not forget this one)*
  - This passphrase is required to setup a TPM Disk Unlock Key passphrase.
  - This passphrase is required to access encrypted data from any computer
  - This passphrase is required to "unsafe boot", where the installed OS will prompt for it.
- TPM Ownership passphrase (2 words.)
  - Used to set ownership on the TPM.
- GPG Admin PIN (2 words. *Locks Admin out after 3 bad attempts in a row. DO NOT FORGET*)
  - This passphrase is requested to do management tasks on the USB Security dongle
   - Under Heads, it is to seal measurements under HOTP
   - It will be needed in case the GPG User PIN was locked
- GPG User PIN (2 words. *Locks user out after 3 bad attempts in a row. DO NOT FORGET*)
  - Used to sign/encrypt content
  - Used to do anything linked to user interaction with the USB Security dongle.
  - GPG prompts for this passphrase when asking "Type Unlock PIN".
- TPM Disk Unlock Key passphrase (3 words, asked to boot default boot option)
  - Requires GPG User PIN and Disk Recovery Key passphrase to setup

It is recommended to write those secrets down and fill them as you go:
- Disk Recovery Key passphrase:
- TPM Ownership passphrase:
- GPG Admin PIN:
- GPG User PIN:
- TPM Disk Unlock Key passphrase:  

Process
---
This will go first briefly over a survey, asking you if you want to: 
- Re-encrypt the LUKS encrypted container (Say yes here if you didn't install the OS yourself)
  - As explained on screen, anyone having a LUKS header backup could restore it and decrypt with
    past corresponding passphrase. Changing passphrase without reencrypting doesn't change the
    encryption key.
- Change the disk encryption key passphrase (Say yes here if you didn't install the OS yourself)
  - You should have also said yes above.
- Define a single shared passphrase across all security components (not recommended)
  - This option is used by some OEMs to provision initial secrets. Passphrases should be different
- Define individual passphrases for each security components (recommended:y )
  - This is the preferred option
- Set custom under information for the GnuPG key (recommended: y)
  - If you desire to use the USB Security dongle to encrypt/sign content linked to a public identity
  That identity needs to be provisioned in a way that it will be searchable if you ever decide to
  upload the resulting public key to gpg key search engines.
  
  Note that the Comment section is used to differenciate the resulting public key from other public
  keys that would be linked with the same Real Name and E-Mail address, and should be distinguishable
  from the Comment. A good Comment example is: "USB Security dongle".


The process then enforces user's selected choices. 

At the end, the wizard outputs on screen the `Provisioned Security Components Secrets` 
This is the last chance you have to note provisioned secrets correctly until you known them by heart. 
That piece of paper's content is precious, and should be safeguarded accordingly. 

Adding your PGP key
===
![IMG_20720216_040452](https://user-images.githubusercontent.com/827570/168885326-67a3b8e6-ba17-483e-b5ea-72fdc8123dbc.JPG)


Heads uses your own GPG key to sign updates and as a result it needs the
key stored in the ROM image before flashing the full Heads ROM.

Ensure your USB security dongle and the USB drive with your key are still
 inserted. Select "Add a GPG key to the running BIOS" to enter the GPG
 Management menu, then "Add a GPG key to the running BIOS + reflash".  Follow
 the steps and your GPG key will be added to the Heads rom.

Once `flashrom` is complete, reboot and now you should now be back in the Heads
 runtime. It should display a message that is is unable to unseal TOTP.


Configuring the TPM
===
![IMG_20720216_043720](https://user-images.githubusercontent.com/827570/168885434-fee94afb-57c0-4046-bb4d-39de028403d7.JPG)


Once you own the TPM, a QR code will generated that you can scan with your
 google authenticator or [FreeOTP+](https://f-droid.org/en/packages/org.liberty.android.freeotpplus/)
 application and use to validate that the firmware (bootblock, ram/rom stages, Linux
 payload and user config injected files are un-altered.
 
 If you have the HOTP version of the firmware, this is also where Heads will
 ask you for your GPG Admin PIN to seal the secret inside of a HOTP compatible 
 USB Security dongle.

On the next boot, the current TOTP will be computed and you can compare 
 this one-time-password against the value that your phone generates.


TPM Disk Encryption Key (TPM Disk Unlock Key)
---

The LUKS Disk Recovery Key stored under LUKS header at OS install is derived from its user passphrase, 
 which is expanded via the LUKS expansion algorithm to increase the time needed to brute force it. 
 For extra protection it is possible to store an additional LUKS key in the TPM so that it will
 only be released to unlock the LUKS container if the PCRs match (firmware measurements, kernel 
 modules loaded, no recovery shell access) from Heads when selecting a boot option.

If you want to use the TPM to seal a secret used to unlock your LUKS volumes:
Go to boot options, show OS boot options and select a new default boot option:
![IMG_20720216_043824](https://user-images.githubusercontent.com/827570/168886309-35bf30e5-5afc-4203-b991-6f832317d4e1.JPG)

Select make default:
![IMG_20720216_043830](https://user-images.githubusercontent.com/827570/168886395-2678e5b0-771c-4a69-a484-6ee0ca9fc016.JPG)

Answer the prompts properly:
![IMG_20720216_043921](https://user-images.githubusercontent.com/827570/168886507-6e8671f1-c553-464c-90dc-28137a5fbf46.JPG)

This will prompt you for your Disk Recovery Key passphrase, a new TPM Disk unlock passphrase and confirm and finally ask you to enter your GPG Unser PIN to sign the new default boot option before rebooting.

Reboot and you will be prompted for your boot password when that device is
 used to boot in the future:
![IMG_20720216_043940](https://user-images.githubusercontent.com/827570/168886785-581e8548-945b-4b06-a2d7-36ceb1707220.JPG)
![IMG_20720216_061726](https://user-images.githubusercontent.com/827570/168889805-4f606591-1a0c-41c2-8c8a-3493a65bba04.JPG)


The key file can not persist on disk anywhere, since it would allow an adversary to decrypt the drive. 
 Instead it is necessary to unseal/decrypt the key from the TPM and then bundle the key file
 into a RAM copy of Qubes' dom0 initrd on each boot. The initramfs format allows
 concatenated cpio files, so it is easy for the Heads firmware to inject files
 into the Qubes startup script.
