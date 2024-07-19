---
layout: default
title: Lenovo X230 Maximized
permalink: /x230-maximized-building/
nav_order: 1
parent: Step 1 - Building Heads
grand_parent: Installing and configuring
---

Lenovo X230 Maximized
====

Please determine the [version]({{ site.baseurl }}/Prerequisites#supported-devices) you want to build (HOTP or not).

For the Thinkpad x230 there are multiple maximized boards under `./boards`, 
 x230-maximized, x230-hotp-maximized and usb-kb or edp-fhd variations.

 All those roms are externally flashable through their top and bottom rom images.

As opposed to Legacy boards produced ROM images, Maximized boards produced ROMs 
 are totally valid ROMs, including a valid Intel Flash Descriptor (IFD), Ethernet
 Intel Gigabit configuration flash space fiaxating MAC to DE:AD:C0:FF:EE,
 a valid neutered Intel ME containing only BUP+ROMP modules which liberated
 space is given back to the BIOS (coreboot+payload) region. 

x230 maximized boards need blobs to be downloaded and extracted from Lenovo prior 
 of building the board configuration but those are managed by the board build, so see general builds instructions.

Please refer to the [xx30 blobs documentation](https://github.com/osresearch/heads/blob/master/blobs/xx30/README),

You can also [download the ROMs directly from CircleCI]({{ site.baseurl }}/Downloading)


Please continue with the [flashing guide]({{ site.baseurl }}/x230-maximized-flashing/)

More options and detail about Heads modules under [Makefile]({{ site.baseurl }}/Makefile/)
