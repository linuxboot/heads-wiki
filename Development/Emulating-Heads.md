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

| Target | Interface | Configuration |
|--|--|--|--|
| `qemu-coreboot-whiptail-tpm1` | Text | TPM1 |
| `qemu-coreboot-whiptail-tpm1-hotp` | Text | TPM1, with HOTP |
| `qemu-coreboot-fbwhiptail-tpm1-hotp` | Graphical | TPM1, with HOTP |
| `qemu-coreboot-fbwhiptail-tpm2-hotp` | Graphical | TPM2, with HOTP |
| ... | ... | Other permutations |

Basic build/boot tests
===

Generate the ROM:

```Makefile
make BOARD=qemu-coreboot-fbwhiptail-tpm1-hotp
```

Boot it in qemu:

```Shell
make BOARD=qemu-coreboot-fbwhiptail-tpm1-hotp run
```

Comprehensive test
===

Most functionality of Heads can be tested in these ROMs with some manual steps
in initial setup.

For more information and setup instructions, refer to the [qemu documentation](https://github.com/linuxboot/heads/blob/sample-brand-acmeboot/targets/qemu.md).

Flashing firmware is not currently possible in QEMU - a GPG key must be injected
at build time, config changes / firmware upgrades cannot be tested, etc.
