---
layout: default
title: Lenovo T480 Maximized
permalink: /T480-maximized-flashing/
nav_order: 1
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Lenovo T480 (Maximized)
===

[T480 Hardware Maintenance Manual](https://download.lenovo.com/pccbbs/mobiles_pdf/t480_hmm_en.pdf)  

It has never been so easy to access and flash the BIOS chip on a T series machine. It is a straightforward process and takes approximately 10 minutes. 

The thinkpad T480 has two SPI flash chips important for the port. First chip holds the BIOS, ME, etc. Second holds the thunderbolt firmware. To access these chips, you only need to remove the back panel.  For whole procedure you will need: a screwdriver, an assembled raspberry pi pico or ch341a SPI programmer 3.3V (3.3V is important!) (e.g. [Modified ch341a SPI programmer](https://novacustom.com/product/modded-ch341a-bios-firmware-programmer-3v/) by Novacustom) and an other laptop/PC with Ubuntu installed. Other linux based OS (in this example Qubes OS, fedora-41-xfce) should be fine too. 

Before you follow the guide make sure you read the [README.md](https://github.com/linuxboot/heads/tree/master/blobs/xx80/README.md) and following information. Some T480 on used marked are affected by an intel bug in the thunderbolt firmware. In short, the flash chip gets full, so the thunderbolt fast charging stops working, but slow charging still works. It can affect the usb-c port too. You may want to resolve the problem before flashing Heads using [Lenovo guide ](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-t-series-laptops/thinkpad-t480-type-20l5-20l6/solutions/ht508988-critical-intel-thunderbolt-software-and-firmware-updates-thinkpad) or [libreboot guide](https://libreboot.org/docs/install/t480.html#thunderbolt-issue-read-this-before-flashing).
Please note, at the moment (Feb 2025) the thunderbolt data tranfser is not supported upstream by coreboot. However, the video output through the thunderbolt and charging works. So, only the usb-c charging port can be used for the data transfer. Heads provides fixed and padded thunderbolt firmware for your convinience that will only fix "charging problem" if the bug affected your laptop. The flashing steps are based on libreboot strategy and described in the second part of the guide. Board testers did not have the problem. It is unlikely that you get it if your laptop was in use longer than 12 months before. If you get "charging bug" it is possible to fix with external flashing. However, please note that this neither was tested after Heads installation nor the bug appeared after extensive Heads testing of the board.

## Flashing Heads

First remove the battery and the cable powering your device. Removing these screws will allow you to remove the back panel.

![Back view]({{ site.baseurl }}/images/T480/1_screws.jpg)

The back panel and the battery are removed. Important, detach the internal battery and CMOS. Arrows point to the direction you should pull the connectors. Pull the plastic part and not the wires, since they are thin and can be damaged.

![Battery]({{ site.baseurl }}/images/T480/2_battery.jpg)

Top right chip corresponds to the Thunderbolt SPI flash chip (1 mb). Chip located in the middle of the board corresponds to the BIOS (16 mb) chip, respectively. 

The chip located in the middle of the board contains the [Intel Management Engine (ME)](https://www.flashrom.org/ME) firmware.

![Chips]({{ site.baseurl }}/images/T480/3_chips.jpg)

First [download]({{ site.baseurl }}/Downloading)  or build (please see [general building]({{ site.baseurl }}/{{ site.baseurl }}/x230-maximized-building/) / [building x230]({{ site.baseurl }}/x230-maximized-building/)) the board rom for this board and verify its hash value.


Try to read the name of the SPI flash chip. The dot on the chip helps to identify the correct clip orientation. 

![SPI BIOS flash chip closed view]({{ site.baseurl }}/images/T480/4_bios_chip_orientation.jpg)

 First, connect the clip of the ch341a programmer to the chip. Next, connect the programmer to the usb port of your other Linux-based computer with flashrom installed. In my set up, the red wire should be where the dot is (dot indicates pin 1). Here, please see flashing guide for the t430. Using raspberry pi pico is nicely described in the libreboot guide [Libreboot flash ](https://libreboot.org/docs/install/spi.html).

 Use flashrom to check the chip that you are connected to:

```shell
sudo flashrom -p ch341a_spi
```

 Here is my output.

![output bios chip]({{ site.baseurl }}/images/T480/5_chip_name.jpg)

Read from the chip twice (where the name of the flash chip is `YYY`):

```shell
sudo flashrom -r ~/t480_original_bios.bin --programmer ch341a_spi - c YYY
```

First output can be seen here. 
![1-st read]({{ site.baseurl }}/images/T480/6_lenovo_bios.jpg)

```shell
sudo flashrom -r ~/t480_original_bios_1.bin --programmer ch341a_spi - c YYY
```
Second output can be seen here. 
![2-nd read]({{ site.baseurl }}/images/T480/7_lenovo_bios_1.jpg)

Make sure that files do not differ.

```shell
sha256sum t480_original_bios.bin
sha256sum t480_original_bios_1.bin
```

My dumps were same. 
![Comparison]({{ site.baseurl }}/images/T480/8_sha256.jpg)

If the files differ then try reconnecting your programmer to the SPI flash chip and make sure your flashrom software is up to date.


If they are the same then write `T480-hotp-maximized.rom` to the SPI flash chip:

```shell
sudo flashrom -p ch341a_spi -c YYY -w ~/heads/build/x86/T480-hotp-maximized/T480-hotp-maximized.rom
```

Here is a successful attempt. Be patient, it can take a while.
![erase/write done]({{ site.baseurl }}/images/T480/9_flash.jpg)

If all goes well you can connect the battery, press the power button and you should see the keyboard LED flash, and after that Heads will boot in its GUI. 

Two reboots are sometimes needed after flash. Force power off by holding the power button for 10 seconds. Since the memory training data was wiped by the content of the full flashed ROM, this is normal.

You should then follow through with [configuring keys]({{ site.baseurl }}/Configuring-Keys/).

## Flash Thundebolt firmware
This guide was adopted from libreboot. The original instructions were provided by [Adam McNutt](https://gitlab.com/MobileAZN/lenovo-t480-thunderbolt-firmware-fixes).

After connecting the clip to the thunderbolt chip (the figure is shown above) read from the chip, making sure the connection is stable. 

```shell
sudo flashrom -r ~/t480_original_tb.bin --programmer ch341a_spi - c YYY
```

```shell
sudo flashrom -r ~/t480_original_tb_1.bin --programmer ch341a_spi - c YYY
```

Make sure that files do not differ.

```shell
sha256sum t480_original_tb.bin
sha256sum t480_original_tb_1.bin
```
First erase the thunderbolt chip.
```shell
sudo flashrom -E --programmer ch341a_spi - c YYY
```
Create a file null.bin containing all zeros.
```shell
dd if=/dev/zero of=null.bin bs=1M count=1
```
Ensure that everyting went smoothly.
```shell
hexdump -v -e '1/1 "%02X\n"' null.bin | grep -vqE '^00$' && echo "null.bin contain non-zero bytes" || echo "null.bin contains only zero bytes"
```
If yes, flash the null.bin on the thunderbolt chip.

```shell
sudo flashrom -p ch341a_spi -c YYY -w ~/null.bin
```
Remove the clip, connect the battery and boot the machine. It should boot. Next, shutdown, remove the battery and reconnect the clip again in order to flash the padded thunderbolt firmware. The firwmare is located in the blobs folder after you build the Heads locally, or in the CircleCI artificats. Alternatively, it can be taken from libreboot. 

```shell
sudo flashrom -p ch341a_spi -c YYY -w ~/tb.bin
```
Done.