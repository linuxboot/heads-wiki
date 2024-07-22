---
layout: default
title: About
has_children: true
has_toc: false
permalink: /
nav_order: 1
---

![Heads booting on an x230](https://user-images.githubusercontent.com/827570/156627927-7239a936-e7b1-4ffb-9329-1c422dc70266.jpeg)

Overview
===

Heads is an open source custom firmware and OS configuration for laptops
and servers that aims to provide slightly better physical security and
protection for data on the system. Unlike Tails, which aims to be a
stateless OS that leaves no trace on the computer of its presence, Heads
is intended for the case where you need to store data and state on the
computer.

![ch341a_flashing](https://user-images.githubusercontent.com/827570/204050962-66f78acb-4dfa-4465-9cda-b212905b8bb8.png)


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


Additionally, the hypervisor, kernel and initrd images are signed by
 keys controlled by the user.  While all of these firmware and software changes
 don't secure the system against every possible attack vector, they address
 several classes of attacks against the boot process and physical hardware
 that have been neglected in traditional installations, hopefully raising
 the difficulty beyond what most attackers are willing to spend.

Further reading
---

![33c3]({{ site.baseurl }}/images/33c3.jpg)

Conferences
===

* [2016 - 33C3 - Trammel Hudson - Heads Presentation](https://trmm.net/Heads_33c3)
* [2017 - 34C3 - Trammel Hudson - LinuxBoot Presentation](https://trmm.net/LinuxBoot_34c3/)
* [2019 - Platform Security Summit - Thierry Laurion - Accessible Security: An OEM approach to transferring device and secrets ownership](https://www.platformsecuritysummit.com/2019/)
* [2020 - FOSDEM - Thierry Laurion - Heads OEM device initial/trasfer of ownership](https://archive.fosdem.org/2020/schedule/event/firmware_hodorateatria/)
* [2020 - SOCALINUXEXPO - Kyle Rankin - Tamper Evident Firmware with User-controlled keys](https://www.youtube.com/watch?v=NqQI3nr1dqk)
* [2023 - FOSDEM - Thierry Laurion - Heads status update](https://archive.fosdem.org/2023/schedule/event/heads_status_update/)

Articles
===
* [2023 - BLOG - Michael Atfield - Trusted Boot (Anti-Evil-Mail, Heads and Pureboot](https://tech.michaelaltfield.net/2023/02/16/evil-maid-heads-pureboot/)


Learn more about Heads
---

* [Heads threat model]({{ site.baseurl }}/Heads-threat-model/) - goes into more
 detail about what classes of threats Heads attempts to counter.
* [Frequently Asked Questions]({{ site.baseurl }}/FAQ/)
* [Requirements for Heads]({{ site.baseurl }}/Install-and-Configure)
