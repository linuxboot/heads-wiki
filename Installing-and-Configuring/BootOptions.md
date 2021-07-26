---
layout: default
title: Configuring Boot Options
permalink: /BootOptions/
parent: Installing and configuring
nav_order: 85
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


Boot config files
===

A user has the option to make persistent modifications to the non-Qubes boot
 process by creating one or more of the following files:

| file | description |
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
| /boot/ | used during internal HD boot |
| /media/ | used during standard USB boot |
| /media/kexec_iso/$ISO_FILENAME/ | used during USB boot from a particular ISO
 file |

These files are only used if there is an appropriate signature for them in `kexec.sig` covering all `kexec*.txt` in that location. This can be generated from the user interface from the `Update checksums and sign all files` in /boot menu option, or manually from the recovery shell by running `kexec-sign-config -p /boot/`, etc. These files are only copied by `kexec-check-config` to `/tmp/kexec/` if there is a valid signature. From there the boot routines reference only the configs in `/tmp/kexec`.

Dynamic vs Persistent Boot Options
====

There are two ways for heads to boot Operating systems from /boot.

* Dynamic (no kexec_menu.txt)
* Persistent

`kexec_menu.txt` is generated from GUI menu option `Default boot` while /boot contents detached signed digest is verified as valid.  If there is no persistent `kexec_menu.txt` the boot directory will be searched for grub/syslinux-like configurations and it will be generated dynamically (for any of the HD/USB/USB-ISO locations). Creating a persistent `kexec_menu.txt` can be useful to limit the options displayed or to make persistent alterations to xen or kernel params.


Persistent Boot Options
----

To customize the boot options and ignore the default OS boot configurations you may create a
`kexec_menu.txt` which has a simple layout of a single line per boot option:

```text
description 1|elf|kernel /vmlinuz... |initrd /initramfs... |append ...
description 2|multiboot|kernel ... |module ... |module ...
description 3|xen|kernel /xen... |module /vmlinuz... | module /initramfs...
```

Here is a sample `kexec_menu.txt` derived from grub.cfg:

<!-- markdownlint-disable MD013 -->

```text
Ubuntu|elf|kernel /vmlinuz-4.8.0-58-generic|initrd /initrd.img-4.8.0-58-generic|append root=/dev/mapper/ubuntu--vg-root ro quiet splash crashkernel=384M-:128M crashkernel=384M-:128M
Memory test (memtest86+, serial console 115200)|elf|kernel /memtest86+.bin|append console=ttyS0,115200n8
Qubes, with Xen hypervisor|multiboot|kernel /xen-4.6.5.gz placeholder |module /vmlinuz-4.4.67-13.pvops.qubes.x86_64 placeholder root=/dev/mapper/luks-UUID ro rd.qubes.hide_all_usb|module /initramfs-4.4.67-13.pvops.qubes.x86_64.img
```

<!-- markdownlint-enable MD013 -->

### Securing Persistent Boot Options

By default, no file hash checks are made for default boot since this was done during configuration.  A non-default boot will fail when the file hashes don't match the expected values.  

### require hash checks

If a user wishes to require that file hashes be checked for a succesful
 non-recovery boot, they may set the `CONFIG_BOOT_REQ_HASH=y` in their
 respective Heads config file (/etc/config.user).

### default boot

As as convenience mechanism, a user may select a boot option to always be used
 in the future, assuming that the boot parameters and file hashes have not
 changed.  This can be done by running `kexec-save-default` manually or directly
 from the boot menu.  This works for any boot location (HD/USB/USB ISO) but does
 modify the respective `/boot/` or `/media/` filesystems.  
 
 In the case of dynamicly derived boot options from `grub.cfg` (i.e. no persistent kexec_menu.txt) an entry index is cached so that the boot will fail when there is a change to the underlying grub parameters.  This will require the user to resign/revalidate the settings.  This is useful to detect changes to the primary kernel/initramfs (for example in the Qubes case when the primary entry  is first).

### multiboot

Note that currently, any multiboot entry is interpreted as a Xen-variant and
 `kexec-boot` overrides the arguments to the multiboot kernel with custom
 arguments.  A user can manually specify `multiboot` entries to override the
 default behavior by creating a custom `kexec_menu.txt`.

### rollback counter

If a user wishes to require that a TPM counter be set for rollback prevention,
 they may set the `CONFIG_BOOT_REQ_ROLLBACK=y` in their respective Heads config
 file.  When this is true, standard boot will only succeed in these two cases:

 * Booting from an verified ISO
 * Booting from a mount point that has a valid `kexec_rollback.txt` in its
 parameter directory

The simplest way to achieve this is to set a default boot option as this updates
 the rollback counter by default.

