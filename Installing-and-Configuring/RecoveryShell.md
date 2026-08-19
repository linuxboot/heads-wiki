---
layout: default
title: Heads Recovery Shell
permalink: /RecoveryShell/
nav_order: 90
has_children: false
has_toc: false
parent: Installing and configuring
---

{: .note }
The authoritative recovery shell reference is maintained in the source repository:
[doc/recovery-shell.md](https://github.com/linuxboot/heads/blob/master/doc/recovery-shell.md).

Heads Recovery Shell
====

The recovery shell is a command line environment where heads and *nix utilities may be accessed (see [modules](https://github.com/linuxboot/heads/tree/master/modules)) This document is a work in progress.

Overview
---

The recovery shell is running a Linux kernel and shell based on busybox.  The environment comes from the initrd image flashed into the BIOS with the Linux kernel.  i.e.  Heads

The shell can be used to troubleshoot Heads.  The security dongle paired with Heads may be needed to effectively use this environment.


Access
----

The recovery shell may be accessed using two different methods.

1.  at power-on repeatedly press 'r' before the GUI loads
2.  select the recovery shell menu in the Heads GUI

These two different methods of access will result in some different settings.


Limitations
----

The recovery shell wipes secrets--normally used for security checks--that were [computed]({{ site.baseurl }}/Keys/#tpm-pcrs) from the BIOS, kernel modules loaded, etc.  This will limit sealing/unsealing functions (Disk Unlock Key creation, TOTP/HOTP sealing) from the recovery shell environment. To seal/unseal secrets, the same measurements needs to be calculated, which would be different depending of the kernel modules loaded and if going in/out of the recovery shell, which invalidates per design the TPM measurements to not release secrets.

To seal/unseal secrets, use the GUI environment.


TPM GPIO Reset Vulnerability Testing
----

Heads includes tools to test for the [TPM GPIO reset vulnerability](https://mkukri.xyz/2024/06/01/tpm-gpio-fail.html) (disclosed by Mate Kukri, 2024). On Intel Skylake+ platforms, the PLTRST# pin may be reprogammable to GPIO mode, allowing an attacker to reset the TPM from userspace.

Two tools are available in the recovery shell (USB stick recommended for log capture):

```bash
# Mount USB stick read-write, audit GPIO lock status (safe, read-only)
mount-usb.sh --mode rw && tpm-gpio-detect 2>&1 | tee /media/tpm-gpio-detect.log

# Assert PLTRST# and check TPM reset (destructive -- platform may reset)
tpm-gpio-assert 2>&1 | tee /media/tpm-gpio-assert.log
```

**detect** identifies your platform and checks whether GPIO pad configuration
registers are locked by firmware. **assert** attempts to assert PLTRST# and
verifies whether the write took effect.

After testing, report results to the [Heads community](https://github.com/linuxboot/heads/issues)
or the [tpm-gpio-fail fork](https://github.com/tlaurion/tpm-gpio-fail). Per-platform
status is tracked in the [Threat Model]({{ site.baseurl }}/Heads-threat-model/).

See the [Threat Model]({{ site.baseurl }}/Heads-threat-model/) for per-platform vulnerability
status and mitigation guidance.


Boot Process
----

To troubleshoot you should understand the processes used to configure and boot Heads after flashing it into the BIOS.

### configuration

* First Boot Checks
* set default boot and flash BIOS (again)
* Reset the TPM
* sign files in /boot
* reset htop/totp

### boot

* mount default boot
* check signatures
* check hotp
* menus and user interaction
* boot kernel


Troubleshoot the boot process
----

### manual boot

If you want or need to manually boot a Linux system you must specify a kernel, initrd, and root file system.  Use the kexec-boot utility.  In this example 'foo' is a description that normally comes from other parts of Heads configurations.  It can be anything.  The root filesystem must be the correct one used in the target Linux installation.  

This example may work for you by changing only the root= setting.  Normally, there are symlinks in the boot partition using these file names.  Of course you need to adjust if your target system uses a different convention for specifying these files.

    kexec-boot -b /boot -e ‘foo|elf|kernel /vmlinuz|initrd /initrd.img|append root=/dev/whatever’


### hotp and totp

Will not work in recovery shell due to missing secrets. See [Limitations](#limitations).


Upgrading Heads
----

The Heads [upgrade process]({{ site.baseurl }}/Updating) may be performed from the recovery shell.