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

Generating your PGP key
===

If you're using a new Yubikey, you'll need to generate your key files. If you
already have the public key stubs for your Yubikey, please proceed
to the next section.  There is some more info in the [GPG guide](/GPG))

Insert your Yubikey into the x230, then invoke GPG's the "Card Edit"
function with it targetting the local directory:

```shell
gpg --homedir=/media/gnupg/ --card-edit
```

Go into "Admin" mode and generate a new key inside the Yubikey:

```shell
admin
generate
```

Since this key can be replaced by replacing the ROM, it is not necessary
to make a backup unless you want to.
This will prompt you for the admin pin (`12345678` by default) and then
the existing pin (`123456`).  Follow the other prompts and eventually
you should have a key in `/media/gnupg/`.

Create a single file containing the public key for this Yubikey (the secret key
 lives only in the Yubikey).

```shell
gpg --homedir=/media/gnupg/ --export -a > /media/gnupg/public.key
```

Adding your PGP key
===

Heads uses your own GPG key to sign updates and as a result it needs the
key stored in the ROM image before flashing the full Heads ROM.

Add your key to the Heads ROM using the following command:

```shell
cbfs -o /media/x230.rom -a "heads/initrd/.gnupg/keys/public.key" -f /media/gnupg/public.key
```

Any name can be used as long as the it is preceded by
 `heads/initrd/.gnupg/keys/`.

After these files are added to the `/media/x230.rom`, you should flash the full
 ROM:

```shell
flashrom-x230.sh /media/x230.com
```

Once `flashrom` is complete, reboot (using the `reboot` command)
and now you should now be back in the Heads runtime. It should
display a message that is is unable to unseal TOTP.

Because the reproducible flash has an empty MRC cache, you need to
reboot one more time so that the PCR values as they would be going
forward.




Configuring the TPM
===

There aren't very many good details on how to setup TPMs, so this section could
 use some work.

Taking ownership
---

If you've acquired the machine from elsewhere, you'll need to establish physical
 presence, perform a force clear and take ownership with your own password.
 Should the storage root key (SRK) be set to something other than the well-known
 password?

```shell
tpm-reset
```

There is something weird with enabling, presence and disabling. Sometimes reboot
 fixes the state.

tpmtotp
---

![TPMTOTP QR code]({{ site.url }}/images/TPMTOTP_QR_code.jpg)

Once you own the TPM, run `seal-totp` to generate a random secret, seal it with
 the current TPM PCR values and store the sealed value in the TPM's NVRAM. This
 will generate a QR code that you can scan with your google authenticator
 application and use to validate that the boot block, rom stage and Linux
 payload are un-altered.

![TPMTOTP output]({{ site.url }}/images/TPMTOTP_output.jpg)

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
3. Insert your GPG card
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
