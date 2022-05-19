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
 x230-maximized, x230-hotp-maximized and x230-hotp-maximized_usb-kb.

 All those roms are externally flashable through their top and bottom rom images.

As opposed to Legacy boards produced ROM images, Maximized boards produced ROMs 
 are totally valid ROMs, including a valid Intel Flash Descriptor (IFD), Ethernet
 Intel Gigabit configuration flash space fiaxating MAC to DE:AD:C0:FF:EE,
 a valid neutered Intel ME containing only BUP+ROMP modules which liberated
 space is given back to the BIOS (coreboot+payload) region. 

x230 maximized boards need blobs to be downloaded and extracted from Lenovo prior 
 of building the board configuration.

Please refer to the [xx30 blobs documentation](https://github.com/osresearch/heads/blob/master/blobs/xx30/README),
debian-11 [dependencies tested working per CircleCI](https://github.com/osresearch/heads/blob/7327f3524eadfa9fe77d0477d63d8e42c692c83a/.circleci/config.yml#L16)
and [how CircleCI call the blobs download script](https://github.com/osresearch/heads/blob/7327f3524eadfa9fe77d0477d63d8e42c692c83a/.circleci/config.yml#L96) to produce ROMs.

You can also [download the ROMs directly from CircleCI]({{ site.baseurl }}/Downloading)

```Makefile
make BOARD=x230-maximized bootstrap
make BOARD=x230-maximized
```

or

```Makefile
make BOARD=x230-hotp-maximized bootstrap
make BOARD=x230-hotp-maximized
```

If running a supported OS (Debian-11/Ubuntu 20.04), have installed proper dependencies, 
 ran the xx30 blobs download script prior of calling the above make commands for your chosen 
 board flavor, 3 roms should have been created: 
- A fully valid 12Mb rom to be flashed internally, also splitted into:
- A top(4Mb) ROM, to be flashed externally on the top SPI flash chip.
- A bottom(8Mb) ROM, to be flashed externally on the bottom SPI flash chip

Please continue with the [flashing guide]({{ site.baseurl }}/x230-maximized-flashing/)

More options and detail about Heads modules under [Makefile]({{ site.baseurl }}/Makefile/)
