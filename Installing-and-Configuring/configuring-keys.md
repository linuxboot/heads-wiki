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

`/boot` missing
===

![Boot missing]({{ site.baseurl }}/images/boot_missing.jpg)

If starting with a blank hard drive, the Heads setup will display an error
 saying `/boot` cannot be found.  Unless there is an nonencrypted boot partition
 already configured on the drive, select "No".  You can either use `fdisk` in
 the recovery shell or using the partition management option during the OS
 installation.

Generating your PGP key
===

![GPG missing]({{ site.baseurl }}/images/gpg_missing.jpg)

Select "Add a GPG key to the running BIOS" to enter the GPG Management menu.  If
 you're using a new USB security dongle, you'll need to generate your key files.
 If you already have the public key stubs for your USB security dongle, please
 proceed to [Adding your PGP key](#adding-your-pgp-key).

![GPG Management menu]({{ site.baseurl }}/images/gpg_management.jpg)

Insert your USB security dongle and USB drive into the device, then chose
 "Generate GPG keys manually on a USB security token".

When at the GPG prompt, go into "Admin" mode and generate a new key inside the
 USB security dongle:

```shell
admin
generate
```

This will prompt you for the admin pin (`12345678` by default for Yubikey) and
 then the existing pin (`123456` default for Yubikey).  Follow the other prompts
 and eventually you should have a key saved onto the USB drive.

When complete, you will be returned to menu.

There is some more info in the [GPG guide](/GPG).

Adding your PGP key
===

Heads uses your own GPG key to sign updates and as a result it needs the
key stored in the ROM image before flashing the full Heads ROM.

Ensure your USB security dongle and the USB drive with your key are still
 inserted. Select "Add a GPG key to the running BIOS" to enter the GPG
 Management menu, then "Add a GPG key to the running BIOS + reflash".  Follow
 the steps and your GPG key will be added to the Heads rom.

Once `flashrom` is complete, reboot and now you should now be back in the Heads
 runtime. It should display a message that is is unable to unseal TOTP.

Because the reproducible flash has an empty MRC cache, you need to
reboot one more time so that the PCR values as they would be going
forward.

Configuring the TPM
===

![TOTP/HOTP missing]({{ site.baseurl }}/images/otp_error.jpg)

There aren't very many good details on how to setup TPMs, so this section could
 use some work.

tpmtotp
---

![TPMTOTP QR code]({{ site.baseurl }}/images/TPMTOTP_QR_code.jpg)

Once you own the TPM, a QR code will generated that you can scan with your
 google authenticator or
 [FreeOTP+](https://f-droid.org/en/packages/org.liberty.android.freeotpplus/)
 application and use to validate that the boot block, rom stage and Linux
 payload are un-altered.

![TPMTOTP output]({{ site.baseurl }}/images/TPMTOTP_output.jpg)

On the next boot, or if you run `unseal-totp`, the script will extract the
 sealed blob from the NVRAM and the TPM will validate that the PCR values are as
 expected before it unseals it. If this works, the current TOTP will be computed
 and you can compare this one-time-password against the value that your phone
 generates.

This does not eliminate all firmware attacks (such as evil maid ones that
 replace the SPI flash chip), but when combined with the WP# pin and BP bits
 should eliminate a software only attack.

TPM Disk encryption keys
---

The keys are currently derived only from the user passphrase, which is expanded
 via the LUKS expansion algorithm to increase the time to brute force it. For
 extra protection it is possible to store the keys in the TPM so that they will
 only be released if the PCRs match.

If you want to use the TPM to seal a secret used to unlock your LUKS volumes:

1. Enter recovery mode
2. Ensure that your the boot devices is mounted: `mount -o ro /dev/sda1 /boot`
 or whatever is appropriate
3. Insert your USB security dongle
4. Run `kexec-save-key -p /boot/ ...` with the followed by options appropriate
 to your OS.  The key will be installed in all devices in the LVM volume group
 as well as any other devices specified after the `-l` option.

Examples for the `kexec-save-key` parameters:

| Installation Type | Command |
| ---- | ---- |
| Previous Heads installation | `kexec-save-key -p /boot/ -l qubes_dom0` |
| Default Qubes / Default Fedora 25 | `kexec-save-key -p /boot/ /dev/sda2` |
| Default Ubuntu 16.04 / Debian 9 (\*) | `kexec-save-key -p /boot/ /dev/sda5` |

Reboot and you will be prompted for your boot password when that device is
 used to boot in the future.

NOTE: should the new LUKS headers be measured and the key re-sealed with those
 parameters? This is what the Qubes AEM setup uses and is probably a good idea
 (although we've already attested to the state of the firmware).

This is where things get messy right now. The key file can not persist on disk
 anywhere, since it would allow an adversary to decrypt the drive. Instead it is
 necessary to unseal/decrypt the key from the TPM and then bundle the key file
 into a RAM copy of Qubes' dom0 initrd on each boot. The initramfs format allows
 concatenated cpio files, so it is easy for the Heads firmware to inject files
 into the Qubes startup script.
