---
layout: default
title: Asus P8Z77-M Pro
permalink: /p8z77-m_pro-building/
nav_order: 1
parent: Step 1 - Building Heads
grand_parent: Installing and configuring
---

ASUS P8Z77-M Pro 
===

This board currently supports TPMv1 modules, please note that TPM v2 module is not currently supported
The following TPM modules have been tested and known to work: Asus branded TPM rev. 1.02H & Foxconn TPM Krypton rev. 1.0

Please determine the [version]({{ site.baseurl }}/Prerequisites#supported-devices) you want to build (HOTP or not).

For the ASUS P8Z77-M Pro there are multiple maximized boards under `./boards`,  p8z77-m_pro-tpm1-maximized and p8z77-m_pro-tpm1-maximized-hotp.

As opposed to Legacy boards produced ROM images, Maximized boards produced ROMs are totally valid ROMs, including a valid Intel Flash Descriptor (IFD), Ethernet, a valid neutered Intel ME containing only BUP+ROMP modules which liberated space is given back to the BIOS (coreboot+payload) region. The IFD also has the VSCC table remove in these boards](https://github.com/corna/me_cleaner/issues/80) which prevents the ME having an instruction of what model of flash chip to write to.

These boards have a script in the 'blobs/p8z77-m_pro' folder which will automatitcally download a factory rom and perform the necassary modifications and extractionas.

You can also [download the ROMs directly from CircleCI]({{ site.baseurl }}/Downloading)

```Makefile
make BOARD=p8z77-m_pro-tpm1-maximized
```

or

```Makefile
make BOARD=p8z77-m_pro-tpm1-hotp-maximized
```

Please continue with the [flashing guide]({{ site.baseurl }}/Asus-p8z77-m_pro-flashing/)

More options and detail about Heads modules under [Makefile]({{ site.baseurl }}/Makefile/)
