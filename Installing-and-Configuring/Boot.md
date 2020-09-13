---
layout: default
title: Boot config files
permalink: /Boot/
nav_order: 6
parent: Installing and configuring
---

Boot config files
===

A user has the option to make persistent modifications to the non-Qubes boot
 process by creating one or more of the following files:

| file | description |
| ---- | ---- |
| kexec_menu.txt | contains multiple options for parameters to the kexec command|
| kexec_hashes.txt | a sha256sum file from within the respective boot directory |
| kexec_iso_add.txt | a sh variable to override the standard ISO kernel argument
 additions |
| kexec_iso_remove.txt | a sh variable to override the standard ISO kernel
 argument removals |
| kexec_default.$N.txt | specifies the default kexec parameters corresponding to
 the Nth menu option |
| kexec_default_hashes.txt | a sha256sum file for the default entry kexec file
 parameters |
| kexec_rollback.txt | a sha256sum of the TPM counter contents in the tmp
 directory |
| kexec_key_devices.txt | contains a list of "device uuid" combos for all LUKS
 devices to unlock |
| kexec_key_lvm.txt | contains the name of an LVM group to activate on boot |

These can be placed in any of the following locations:

| location | description |
| ---- | ---- |
| /boot/ | used during internal HD boot |
| /media/ | used during standard USB boot |
| /media/kexec_iso/$ISO_FILENAME/ | used during USB boot from a particular ISO
 file |

These files are only used if there is an appropriate signature for them in
 `kexec.sig` covering all `kexec*.txt` in that location.  This can be generated
  by running `kexec-sign-config -p /boot/`, etc.  These files are copied by
 `kexec-check-config` to `/tmp/kexec/` only there's a valid signature.  From
  there the boot routines reference only the configs in `/tmp/kexec`.

If there is no persistent `kexec_menu.txt`, the boot directory will be searched
 for grub/syslinux-like configurations and it will be generated on-the-fly (for
 any of the HD/USB/USB-ISO locations).  Creating a persistent
 `kexec_menu.txt` can be useful to limit the options displayed or to make custom
 persistent alterations to xen or kernel params.

`kexec_menu.txt` has a simple layout with a single line per boot option:

```text
description 1|elf|kernel /vmlinuz... |initrd /initramfs... |append ...
description 2|multiboot|kernel ... |module ... |module ...
description 3|xen|kernel /xen... |module /vmlinuz... | module /initramfs...
```

This is a sample `kexec_menu.txt` covering the expected options (derived from
 grub.cfg):

```text
Ubuntu|elf|kernel /vmlinuz-4.8.0-58-generic|initrd /initrd.img-4.8.0-58-generic|append root=/dev/mapper/ubuntu--vg-root ro quiet splash crashkernel=384M-:128M crashkernel=384M-:128M
Memory test (memtest86+, serial console 115200)|elf|kernel /memtest86+.bin|append console=ttyS0,115200n8
Qubes, with Xen hypervisor|multiboot|kernel /xen-4.6.5.gz placeholder |module /vmlinuz-4.4.67-13.pvops.qubes.x86_64 placeholder root=/dev/mapper/luks-UUID ro rd.qubes.hide_all_usb|module /initramfs-4.4.67-13.pvops.qubes.x86_64.img
```

If there is a persistent `kexec_hashes.txt`, a non-default boot will fail when
 the file hashes don't match the expected values.  By default, no such checks
 are made.

When booting from an ISO file on a USB drive, it must be signed by a valid key
 in the Heads ROM and the boot process will fail if invalid. The
 `kexec_iso_add.txt` and `kexec_iso_remove.txt` are useful to inject the
 appropriate kernel arguments to allow it to load properly. ISOs for Debian
 require that `kexec_iso_add.txt` contains to load properly:

```text
findiso=${ISO_PATH}
```

Take a look at [http://mbusb.aguslr.com/howto.html](http://mbusb.aguslr.com/howto.html)
 for more variations on the distro-specific ISO mounting command lines
 requirements.  By default Heads uses two variants of this when booting from
 ISO where a `kexec_iso_add.txt` is not specified:

```text
fromiso=/dev/disk/by-uuid/$DEV_UUID/$ISO_PATH iso-scan/filename=/${ISO_PATH}
```

Note that currently, any multiboot entry is interpreted as a Xen-variant and
 `kexec-boot` overrides the arguments to the multiboot kernel with custom
 arguments.  A user can manually specify `multiboot` entries to override the
 default behavior by creating a custom `kexec_menu.txt`.

If a user wishes to require that file hashes be checked for a succesful
 non-recovery boot, they may set the `CONFIG_BOOT_REQ_HASH=y` in their
 respective Heads config file.

As as convenience mechanism, a user may select a boot option to always be used
 in the future, assuming that the boot parameters and file hashes have not
 changed.  This can be done by running `kexec-save-default` manually or directly
 from the boot menu.  This works for any boot location (HD/USB/USB ISO) but does
 modify the respective `/boot/` or `/media/` filesystems.  An entry index is
 maintained so that if the options are being derived from the live `grub.cfg`
 (i.e. no persistent `kexec_menu.txt`) and when there is a change to the
 underlying grub parameters, the boot will fail and require the user to
 resign/revalidate the settings.  This is useful to detect changes to the
 primary kernel/initramfs (for example in the Qubes case when the primary entry
 is first).

If a user wishes to require that a TPM counter be set for rollback prevention,
 they may set the `CONFIG_BOOT_REQ_ROLLBACK=y` in their respective Heads config
 file.  When this is true, standard boot will only succeed if:

1) Booting from an verified ISO
2) Booting from a mount point that has a valid `kexec_rollback.txt` in its
 parameter directory

The simplest way to achieve this is to set a default boot option as this update
 the rollback counter by default.
