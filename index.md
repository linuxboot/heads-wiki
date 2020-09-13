---
layout: default
title: About
permalink: /
nav_order: 1
---

![Heads booting on an x230]({{ site.url }}/images/Heads_booting_on_an_x230.jpg)

Overview
===

Heads is an open source custom firmware and OS configuration for laptops
and servers that aims to provide slightly better physical security and
protection for data on the system. Unlike Tails, which aims to be a
stateless OS that leaves no trace on the computer of its presence, Heads
is intended for the case where you need to store data and state on the
computer.

![Flashing an x230 bootrom]({{ site.url }}/images/Flashing_an_x230_bootrom.jpg)

Heads is not just another Linux distribution -- it combines physical
hardening of specific hardware platforms and flash security features with
custom coreboot firmware and a Linux boot loader in ROM.  This moves
the root of trust into the write-protected region of the SPI flash and
prevents further software modifications to the bootup code (and on
platforms that support it, [Bootguard](https://trmm.net/Bootguard) can
protect against many hardware attacks as well).  Controlling the
first instruction the CPU executes allows Heads to measure every step of
the boot firmware and configuration into the TPM, which makes it possible
to attest to the user or a remote system that the machine has not been
tampered with.
While modern Intel CPUs require binary blobs to boot, these non-Free
components are included in the measurements and are at least guaranteed
to be unchanging.  Once the system is in a known good state, the TPM is
used as a hardware key storage to decrypt the drive.

![TPM TOTP in action]({{ site.url }}/images/TPM_TOTP_in_action.jpg)

Additionally, the hypervisor, kernel and initrd images are signed by
keys controlled by the user, and the OS uses a signed, immutable root
filesystem so that any software exploits that attempt to gain persistence
will be detected.  While all of these firmware and software changes don't
secure the system against every possible attack vector, they address
several classes of attacks against the boot process and physical hardware
that have been neglected in traditional installations, hopefully raising
the difficulty beyond what most attackers are willing to spend.

Further reading
---

![33c3]({{ site.url }}/images/33c3.jpg)

* [Presentation at 33c3](https://trmm.net/Heads_33c3)
* [Heads threat model](/Heads-threat-model) goes into more
 detail about what classes of threats Heads attempts to counter.
