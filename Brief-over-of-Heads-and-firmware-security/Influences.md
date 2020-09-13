---
layout: default
title: Influences and prior works
permalink: /Influences/
nav_order: 3
parent: Brief overview of Heads and firmware security
---

Heads is influenced by several years of firmware vulnerability
 research by Trammell Hudson including [Thunderstrike](https://trmm.net/Thunderstrike)
 and [Thunderstrike 2](https://trmm.net/Thunderstrike_2)

As well as many other researchers' work

* "[Hardening hardware and choosing a #goodBIOS](https://media.ccc.de/v/30C3_-_5529_-_en_-_saal_2_-_201312271830_-_hardening_hardware_and_choosing_a_goodbios_-_peter_stuge#t=2372)
 " by Peter Stuge,
* "[Beyond anti evil maid](https://media.ccc.de/v/32c3-7343-beyond_anti_evil_maid)
 " by Matthew Garret,
* "[Towards (reasonably) trustworthy x86 laptops](http://www.theregister.co.uk/2015/12/31/rutkowska_talks_on_intel_x86_security_issues/)
 " by Joanna Rutkowska,
* "[LightEater malware seek GPG keys in Tails](http://www.theregister.co.uk/2015/03/19/cansecwest_talk_bioses_hack/)
 " by Kallenberg and Kovah, etc..

Prior works
===

tpmtotp
----

![CoreBoot + Linux + tpmtotp]({{ site.url }}/images/TPMTOTP_in_use.jpg)

Trammell Hudson ported [mjg59's tpmtotp](https://mjg59.dreamwidth.org/35742.html)
 to run from inside the boot ROM of a Thinkpad x230 using CoreBoot with a Linux
 payload. This provides attestation that the firmware hasn't been tampered with,
 since the TPM won't unseal the secret to used in the TOTP HMAC unless the PCR
 values match those expected for the ROM image.

![mjg59 presenting tpmtotp at 32c3]({{ site.url }}/images/mjg59_presenting_tpmtotp_at_32c3.jpg)

 Garrett presented tpmtotp at 32c3 and I really liked the idea, but felt that it
  ran "too late" -- the system has already fetched the kernel and initrd from the
  disk and potentially had the chain of trust compromised. Since my version of
  code is executed from the difficult-to-write SPI flash ROM and the read-only
  boot block initializes the root of trust with measurements of itself as well as
  the rest of the ROM, it is much harder to compromise.

 Additionally, since my ROM image is very size constrained, I didn't want to use
  OpenSSL and liboath and all of the other dependencies. My branch replaces them
  with mbedtls and my own TOTP code, which reduces the size of the executables
  from 5MB to 180KB.

 The source is available from [github.com/osresearch/tpmtotp](https://github.com/osresearch/tpmtotp)
  , although it is currently not usable outside of my build environment. As
  Heads is more developed it will be merged into that project (and likely
  re-written as a shell script).
