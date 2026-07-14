---
layout: default
title: Lenovo T480s Maximized
permalink: /T480s-maximized-flashing/
nav_order: 1
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Lenovo T480s (Maximized)
===

[T480s Hardware Maintenance Manual](https://download.lenovo.com/pccbbs/mobiles_pdf/t480s-hmm_en.pdf)  

Accessing and flashing the BIOS chip on a T-series machine has never been easier. The process is straightforward and takes approximately 10 minutes.

The ThinkPad T480s has two SPI flash chips important for the port. The first chip holds the BIOS, ME, etc., while the second holds the Thunderbolt firmware. To access these chips, you only need to remove the back panel, which is secured by six screws. 

For whole procedure you will need: 
- A Phillips screwdriver +1 (PH1), which is standard for most laptop screws.
- A SPI programmer (recommended: Tigard; budget: CH347F or CH341A rev1.6+). For programmer-specific safety and example commands, see the [SPI Programmer Best Practices]({{ site.baseurl }}/SPI-Programmer-Best-Practices/). Using Raspberry Pi Pico as an example is described in the [Libreboot flash guide](https://libreboot.org/docs/install/spi.html).
- Other laptop/PC with a Linux-based OS installed.  
- Optional: A plastic guitar pick or an old credit card to help detach the bottom case from the clips holding it in place. Otherwise, it can be difficult to remove, increasing the risk of breaking the tabs or the top part of the bottom case above the battery connector.

There is still debate over which programmer and software should be used (flashprog vs. flashrom), however we will use flashrom to keep all the steps consistent. 
Before following this guide, make sure you read [README.md](https://github.com/linuxboot/heads/tree/master/blobs/xx80/README.md) and the related information.

Some ThinkPad T480s units on the used market are affected by an Intel bug in the Thunderbolt firmware. In short, the flash chip becomes full, causing Thunderbolt fast charging to stop working, though slow charging still functions. This issue can also affect the USB-C port. For convenience, Heads provides a fixed and padded Thunderbolt firmware that resolves the "charging problem" if your laptop is affected. Board testers did not encounter this issue, and it is unlikely to occur if your laptop was in use for more than 12 months before flashing. If you do experience the "charging bug," it is possible to fix it with external flashing. Also, the update is possible prior to flashing Heads by using [fwupd from a Linux distribution](https://www.reddit.com/r/thinkpad/comments/12tf6xv/psa_t480_thunderbolt_controller_v23_is_now_on/)

Please note that as of March 2025, Thunderbolt data transfer is not supported upstream by [coreboot](https://review.coreboot.org/c/coreboot/+/83274). However, video output through Thunderbolt and charging still work. This means only the USB-C charging port can be used for data transfer.

## (Optional) Updating EC Firmware

Before flashing heads, it is advisable to update the EC Firmware on the laptop. Some old EC firmware may be vulnerable to some serious CVEs. See [Heads-threat-model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-microcode-updates-and-transient-execution-vulnerabilities) for additional information.

The only way to update the EC Firmware, called ECP (Embedded Controller Program) by Lenovo, is to use their proprietary BIOS Update utility which typically requires a Windows system. However, you can use the BIOS Update .iso file to create a bootable USB to boot directly into the BIOS Update Utility by using geteltorito.

Locate the latest Bootable CD (.iso) file from lenovo: [BIOS Update (Bootable CD)](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/thinkpad-t-series-laptops/thinkpad-t480s-type-20l7-20l8/downloads/ds502226)

As of writting, the latest T480s BIOS Update is n22ur40w.iso (v1.60) published on April 16th, 2025 (so the following wget command could be oudated).

```shell
wget https://download.lenovo.com/pccbbs/mobiles/n22ur40w.iso
```

Acquire the geteltorito script. On debian, the genisoimage package contains geteltorito:
```shell
sudo apt install genisoimage
```

Generate a bootable image from the .iso:
```shell
geteltorito -o t480s_bootable_update_utility.img n22ur40w.iso
```

Write the resulting image to a USB Flash drive:
```shell
sudo dd if=t480s_bootable_update_utility.img of=/dev/sdX status=progress
```

Boot from the USB on the laptop and follow the update prompts.

## Flashing Heads

First, disconnect the power cable from your device then begin removing the back panel by removing the screws. A guitar pick or an old credit card can be helpful for detaching the panel.

![Back view]({{ site.baseurl }}/images/T480s/1_screws.jpg)

The back panel and the battery are removed. Important, ensure that all batteries, including the CMOS battery, are disconnected. Arrows indicate the direction you should pull the connectors. Pull the plastic part, not the wires, as the wires are thin and can be damaged.

![Battery]({{ site.baseurl }}/images/T480s/2_battery.jpg)

The top-right chip corresponds to the Thunderbolt SPI flash chip (1 MB). The chip located in the middle of the board corresponds to the BIOS (16 MB) chip, respectively. 

The chip located in the middle of the board contains the [Intel Management Engine (ME)](https://www.flashrom.org/ME) firmware.

![Chips]({{ site.baseurl }}/images/T480s/3_chips.jpg)

First [download]({{ site.baseurl }}/Downloading)  or build (please see [general building]({{ site.baseurl }}/general-building/) / [building x230]({{ site.baseurl }}/x230-maximized-building/)) the board rom for this board and verify its hash value.

Try to read the name of the SPI flash chip. The dot on the chip helps to identify the correct clip orientation. 

![SPI BIOS flash chip closed view]({{ site.baseurl }}/images/T480s/4_bios_chip_orientation.jpg)

First, connect the clip of your chosen SPI programmer (CH341a Programmer was used here) to the chip. Next, connect the programmer to the USB port of your other Linux-based computer with flashrom installed. In my setup, the red wire should be where the dot is (the dot indicates pin 1). Here, please also see the flashing guide for the T480. 

Use flashrom to check the chip you are connected to:

```shell
sudo flashrom --programmer [programmer]
```

Here is my output.

```
flashrom v1.6.0 on Linux 6.19.14+kali-amd64 (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Found Winbond flash chip "W25Q128.V" (16384 kB, SPI) on ch341a_spi.
No operations were specified.
```

Read from the chip twice to make sure our original BIOS file is consistent (where the name of the flash chip is `YYY`):

```shell
sudo flashrom --programmer [programmer] --read ~/t480s_original_bios.bin --chip YYY
```

It will then dump our original BIOS prior to flashing HEADS to:
```
~/t480s_original_bios.bin
```

And do it once again just to make sure (notice a different file name is used below):
```shell
sudo flashrom --programmer [progarmmer] --read ~/t480s_original_bios_1.bin --chip YYY
```

It will then dump our original BIOS (second read attempt) prior to flashing HEADS to:
```
~/t480s_original_bios_1.bin
```

Make sure that files do not differ. We can verify them using `sha256sum` as follows:

```shell
sha256sum t480s_original_bios.bin
sha256sum t480s_original_bios_1.bin
```

My dumps were the same. 

```
[user@h4ckb0x ~]$ sha256sum t480s_original_bios.bin 
54d58fdf217f41d4385576bb74f45f20de233d1a9d9ea2d6f1543b86111e1062  t480s_original_bios.bin
[user@h4ckb0x ~]$ sha256sum t480s_original_bios_1.bin 
54d58fdf217f41d4385576bb74f45f20de233d1a9d9ea2d6f1543b86111e1062  t480s_original_bios_1.bin
```

Alternative comparison is bit-by-bit. If the files are the same, there should be no output of this command. Otherwise, you will see a bit-by-bit difference between the files.

```shell
diff <(hexdump -C t480s_original_bios.bin) <(hexdump -C t480s_original_bios_1.bin)
```

Now make sure that the dump matches the chip content. If this is the case, the output of the following command will state `Verifying flash... VERIFIED`
```shell
sudo flashrom --programmer [programmer] --verify ~/t480s_original_bios.bin --chip YYY
```

In our case the command output is:
```
flashrom v1.6.0 on Linux 6.19.14+kali-amd64 (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Found Winbond flash chip "W25Q128.V" (16384 kB, SPI) on ch341a_spi.
Verifying flash... VERIFIED.
```

If the files differ or the chip content does not match the dump, try reconnecting your programmer to the SPI flash chip and make sure your flashrom software is up-to-date.


If they are the same, then write `t480s-maximized.rom` to the SPI flash chip:

```shell
sudo flashrom --programmer [programmer] --chip W25Q128.V --write heads-EOL_t480s-maximized-202607091420-v0.2.1-3090-g8d0064f.rom 
```

Here is a successful attempt. Be patient, it may take a while.

```
flashrom v1.6.0 on Linux 6.19.14+kali-amd64 (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Found Winbond flash chip "W25Q128.V" (16384 kB, SPI) on ch341a_spi.
Reading old flash chip contents... done.
Updating flash chip contents... Erase/write done from 0 to ffffff
Verifying flash... VERIFIED.
```

If all goes well you can connect the CMOS and internal battery, press the power button and you should see the keyboard LED flash. After that, Heads will boot into its GUI. 

Two reboots are sometimes needed after flashing. Force a power off by holding the power button for 10 seconds. Since the memory training data was wiped by the content of the fully flashed ROM, this is normal.

You should then follow through with [configuring keys]({{ site.baseurl }}/Configuring-Keys/).

## Flash Thunderbolt firmware
Important, ensure that power supply and all batteries, including the CMOS battery, are disconnected. After connecting the clip to the Thunderbolt chip as shown in the figure above read from the chip, making sure the connection is stable. The procedure is similar to the flashing Heads on the SPI chip. Therefore, comments are skipped.

Check the chip you are connected to:
```shell
sudo flashrom --programmer [programmer]
```
Output:
```
flashrom v1.6.0 on Linux 6.19.14+kali-amd64 (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Found Winbond flash chip "W25Q80.V" (1024 kB, SPI) on ch341a_spi.
===
This flash part has status UNTESTED for operations: WP
The test status of this chip may have been updated in the latest development
version of flashrom. If you are running the latest development version,
please email a report to flashrom@flashrom.org if any of the above operations
work correctly for you with this flash chip. Please include the flashrom log
file for all operations you tested (see the man page for details), and mention
which mainboard or programmer you tested in the subject line.
You can also try to follow the instructions here:
https://www.flashrom.org/contrib_howtos/how_to_mark_chip_tested.html
Thanks for your help!
No operations were specified.
```

Dump the Thunderbolt firmware (again, twice, and compare):
```shell
sudo flashrom --programmer [programmer] --read ~/t480s_original_tb.bin --chip YYY
```

```shell
sudo flashrom --programmer [programmer] --read ~/t480s_original_tb_1.bin --chip YYY
```

Compare our Thunderbolt firmware dumps with `sha256sum`:
```shell
sha256sum t480s_original_tb.bin
sha256sum t480s_original_tb_1.bin
```

Or, bit-by-bit with `diff`
```shell
diff <(hexdump -C t480_original_tb.bin) <(hexdump -C t480_original_tb_1.bin)
```

And if they match, then proceed to the next step to Verify:

```shell
sudo flashrom --programmer [programmer] --verify ~/t480s_original_tb.bin --chip YYY
```

Flash the padded Thunderbolt firmware. The firmware file tb.bin is located in the blobs folder after you build the Heads locally, or in the CircleCI artifacts.

```shell
sudo flashrom --programmer [programmer] --chip YYY --write ~/t480s_tb.bin
```
Done.
