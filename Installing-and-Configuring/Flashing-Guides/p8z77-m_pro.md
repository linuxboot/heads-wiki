---
layout: default
title: Asus P8Z77-M Pro
permalink: /Asus-p8z77-m_pro-flashing/
nav_order: 1
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Asus P8Z77-M Pro
====

This board uses a DIP8 socketed W25q64 Winbond rom. The chip can be removed from the motherboard to flash, be careful when removing the chip to not bend or snap pins.

Use a suitable SPI programmer with an appropriate compatible tool (in this example we use a ch341a_spi programmer with flashroom utility in linux).

It is advised to back up your factory rom incase you wish to revert later

```
flashrom --programmer ch341a_spi -r factory.rom
```

Then flash the heads rom
```
flashrom --programmer ch341a_spi -w heads-circleci-artefect.rom
```
(replace heads-circleci-artefect.rom with the name of the artefact downloaded frmo CircleCi

Re-insert the w25q64 into the motherboard, ensuring that the notch on the side ofg the chip matches the notch on the DIP8 socket. 

It may take a little longer on the first boot after first inserting the chip, as the RAM training happens. 
