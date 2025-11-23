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

Accessing and flashing the BIOS chip on a T-series machine has never been easier. The process is straightforward and takes approximately 10 minutes.

The ThinkPad T480 has two SPI flash chips important for the port. The first chip holds the BIOS, ME, etc., while the second holds the Thunderbolt firmware. To access these chips, you only need to remove the back panel, which is secured by six screws. 

For whole procedure you will need: 
- A Phillips screwdriver +1 (PH1), which is standard for most laptop screws.
- An assembled Raspberry Pi or CH341A SPI programmer. You should use a CH341A revision 1.6 or later (e.g., 1.6, 1.7, etc.) because these versions have a properly implemented and enforced voltage regulator, ensuring stable 3.3V operation (3.3V is important!) (e.g. [Modified CH341A SPI programmer](https://novacustom.com/product/modded-ch341a-bios-firmware-programmer-3v/) by Novacustom) or [open-source Tigard tool](https://github.com/tigard-tools/tigard). Using Raspberry Pi pico is  described in the [Libreboot flash guide](https://libreboot.org/docs/install/spi.html).
- Other laptop/PC with a Linux-based OS installed.  
- Optional: A plastic guitar pick or an old credit card to help detach the bottom case from the clips holding it in place. Otherwise, it can be difficult to remove, increasing the risk of breaking the tabs or the top part of the bottom case above the battery connector.

There is still debate over which programmer and software should be used (flashprog vs. flashrom). Before following this guide, make sure you read [README.md](https://github.com/linuxboot/heads/tree/master/blobs/xx80/README.md) and the related information.

Some ThinkPad T480 units on the used market are affected by an Intel bug in the Thunderbolt firmware. In short, the flash chip becomes full, causing Thunderbolt fast charging to stop working, though slow charging still functions. This issue can also affect the USB-C port. For convenience, Heads provides a fixed and padded Thunderbolt firmware that resolves the "charging problem" if your laptop is affected. Board testers did not encounter this issue, and it is unlikely to occur if your laptop was in use for more than 12 months before flashing. If you do experience the "charging bug," it is possible to fix it with external flashing. Also, the update is possible prior flashing heads using [fwupd from a Linux distribution](https://www.reddit.com/r/thinkpad/comments/12tf6xv/psa_t480_thunderbolt_controller_v23_is_now_on/?rdt=44850)

Please note that as of March 2025, Thunderbolt data transfer is not supported upstream by [coreboot](https://review.coreboot.org/c/coreboot/+/83274). However, video output through Thunderbolt and charging still work. This means only the USB-C charging port can be used for data transfer.

## Flashing Heads

First, remove the battery and disconnect the power cable from your device. Removing the screws will allow you to remove the back panel. A guitar pick or an old credit card can be helpful for detaching the panel.

![Back view]({{ site.baseurl }}/images/T480/1_screws.jpg)

The back panel and the battery are removed. Important, ensure that all batteries, including the CMOS battery, are disconnected. Arrows indicate the direction you should pull the connectors. Pull the plastic part, not the wires, as the wires are thin and can be damaged.

![Battery]({{ site.baseurl }}/images/T480/2_battery.jpg)

The top-right chip corresponds to the Thunderbolt SPI flash chip (1 MB). The chip located in the middle of the board corresponds to the BIOS (16 MB) chip, respectively. 

The chip located in the middle of the board contains the [Intel Management Engine (ME)](https://www.flashrom.org/ME) firmware.

![Chips]({{ site.baseurl }}/images/T480/3_chips.jpg)

First [download]({{ site.baseurl }}/Downloading)  or build (please see [general building]({{ site.baseurl }}/general-building/) / [building x230]({{ site.baseurl }}/x230-maximized-building/)) the board rom for this board and verify its hash value.

Try to read the name of the SPI flash chip. The dot on the chip helps to identify the correct clip orientation. 

![SPI BIOS flash chip closed view]({{ site.baseurl }}/images/T480/4_bios_chip_orientation.jpg)

 First, connect the clip of the CH341A programmer to the chip. Next, connect the programmer to the USB port of your other Linux-based computer with flashrom installed. In my setup, the red wire should be where the dot is (the dot indicates pin 1). Here, please also see the flashing guide for the T430. 

 Use flashrom to check the chip you are connected to:

```shell
sudo flashrom -p ch341a_spi
```

Here is my output.

![output bios chip]({{ site.baseurl }}/images/T480/5_chip_name.jpg)

Read from the chip twice (where the name of the flash chip is `YYY`):

```shell
sudo flashrom -r ~/t480_original_bios.bin --programmer ch341a_spi -c YYY
```

First output can be seen here. 
![1-st read]({{ site.baseurl }}/images/T480/6_lenovo_bios.jpg)

```shell
sudo flashrom -r ~/t480_original_bios_1.bin --programmer ch341a_spi -c YYY
```
Second output can be seen here. 
![2-nd read]({{ site.baseurl }}/images/T480/7_lenovo_bios_1.jpg)

Make sure that the dump matches the chip content. If this is the case, the output of the following command will state `Verifying flash... VERIFIED`

```shell
sudo flashrom -v ~/t480_original_bios.bin --programmer ch341a_spi -c YYY
```
Make sure that files do not differ.

```shell
sha256sum t480_original_bios.bin
sha256sum t480_original_bios_1.bin
```

My dumps were the same. 
![Comparison]({{ site.baseurl }}/images/T480/8_sha256.jpg)

Alternative comparison is bit-by-bit. If the files are the same, there should be no output of this command. Otherwise, you will see a bit-by-bit difference between the files.

```shell
diff <(hexdump -C t480_original_bios.bin) <(hexdump -C t480_original_bios_1.bin)
```

If the files differ or the chip content does not match the dump, try reconnecting your programmer to the SPI flash chip and make sure your flashrom/flashprog software is up-to-date.


If they are the same, you can then write `T480-hotp-maximized.rom` to the SPI flash chip.

```shell
sudo flashrom -p ch341a_spi -c YYY -w ~/heads/build/x86/T480-hotp-maximized/T480-hotp-maximized.rom
```

On boards with Intel-based Ethernet, such as the T480, this will also overwrite the GbE region in the BIOS, which stores the MAC address of the chip, with a forged one (MAC: 00:DE:AD:C0:FF:EE). This has the privacy benefit that the chip uses this shared MAC so it can't be used as a personal identifier for this exact board. The downside is that this can create connectivity problems on local networks if other heads boards with the same MAC address are present. To preserve the original MAC address of the board, use:

```shell
sudo flashrom -p ch341a_spi -c YYY --ifd -i bios -i me -i fd -w ~/heads/build/x86/T480-hotp-maximized/T480-hotp-maximized.rom
```


Here is a successful attempt. Be patient, it may take a while.
![erase/write done]({{ site.baseurl }}/images/T480/9_flash.jpg)

If all goes, well you can connect the battery, press the power button and you should see the keyboard LED flash. After that, Heads will boot into its GUI. 

Two reboots are sometimes needed after flashing. Force a power off by holding the power button for 10 seconds. Since the memory training data was wiped by the content of the fully flashed ROM, this is normal.

You should then follow through with [configuring keys]({{ site.baseurl }}/Configuring-Keys/).

## Flash Thunderbolt firmware
Important, ensure that power supply and all batteries, including the CMOS battery, are disconnected. After connecting the clip to the Thunderbolt chip as shown in the figure above read from the chip, making sure the connection is stable. The procedure is similar to the flashing Heads on the SPI chip. Therefore, comments are skipped.

```shell
sudo flashrom -r ~/t480_original_tb.bin --programmer ch341a_spi -c YYY
```

```shell
sudo flashrom -r ~/t480_original_tb_1.bin --programmer ch341a_spi -c YYY
```

```shell
sudo flashrom -v ~/t480_original_tb.bin --programmer ch341a_spi -c YYY
```

```shell
sha256sum t480_original_tb.bin
sha256sum t480_original_tb_1.bin
```

```shell
diff <(hexdump -C t480_original_tb.bin) <(hexdump -C t480_original_tb_1.bin)
```

Flash the padded Thunderbolt firmware. The firmware file tb.bin is located in the blobs folder after you build the Heads locally, or in the CircleCI artifacts.

```shell
sudo flashrom -p ch341a_spi -c YYY -w ~/t480_tb.bin
```
Done.
