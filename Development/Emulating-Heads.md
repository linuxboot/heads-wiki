---
layout: default
title: Emulating Heads
permalink: /Emulating-Heads/
nav_order: 2
parent: Development
---

Available Targets
===

Multiple qemu targets are provided for Heads.

| Target | Interface | Features |
|--|--|--|--|
| `qemu-coreboot` | Text | Basic build/boot test |
| `qemu-coreboot-fbwhiptail` | Graphical | Basic build/boot test |
| `qemu-coreboot-fbwhiptail-tpm1-hotp` | Graphical | TPM, HOTP with USB token, /boot signing and OS booting. |

Basic build/boot tests
===

Generate the `qemu.rom` image:

```Makefile
make BOARD=qemu-coreboot
```

Boot it in qemu:

```Shell
build/make-4.2/make BOARD=qemu-coreboot run
```

Use `qemu-coreboot-fbwhiptail` as the board instead for the graphical interface.

Issues with emulation:

* TPM is not available
* Xen won't start dom0 correctly, but it is sufficient to test that the
 `initrd.cpio` file was correctly generated
* This also lets us test Xen patches for legacy-free systems
* SATA controller sometimes takes minutes to timeout?

Comprehensive test
===

The `qemu-coreboot-fbwhiptail-tpm1-hotp` configuration permits testing of most features of Heads.

For more information and setup instructions, refer to the [qemu-coreboot-fbwhiptail-tpm1-hotp documentation](https://github.com/osresearch/heads/blob/master/boards/qemu-coreboot-fbwhiptail-tpm1-hotp/qemu-coreboot-fbwhiptail-tpm1-hotp).
