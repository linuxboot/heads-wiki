---
layout: default
title: Lenovo T430 Maximized
permalink: /T430-maximized-flashing/
nav_order: 5
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Lenovo T430 (Maximized)
===
<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details> 

## ⚠️ EOL: No microcode updates
{: .warning }
This board's CPU generation has reached End of Servicing Updates.
See [per-board EOL/ESU status](https://github.com/linuxboot/heads/blob/master/doc/BOARDS_AND_TESTERS.md#per-board-eolesu-status)
for ESU dates and [Heads threat model]({{ site.baseurl }}/Heads-threat-model/#binary-blobs-me-and-peripheral-firmware)
for security implications.

## ✅ PROTECTED: TPM GPIO Reset
{: .warning }

Pre-Skylake platform — dedicated PLTRST# pin, not GPIO-shared.
See [Per-Board Protection Status]({{ site.baseurl }}/Heads-threat-model/#per-board-protection-status),
[TPM GPIO Reset Vulnerability](https://github.com/linuxboot/heads/blob/master/doc/TPM_GPIO_Reset_Vulnerability.md).



## ⚡ Safety First

**Before starting, please read our [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/) for essential safety information and programmer recommendations.**

[T430 Hardware Maintenance Manual](https://download.lenovo.com/ibmdl/pub/pc/pccbbs/mobiles_pdf/t430_t430i_hmm_en_0b48304_04.pdf)  

→ **[coreboot T430 documentation](https://doc.coreboot.org/mainboard/lenovo/t430.html)** (chip layout, flashing instructions)

## Disassembly

First remove the battery and the cable powering your device.

![Keyboard tilted up]({{ site.baseurl }}/images/t430/1_1_back_view_removed_battery.jpg)

Removing these screws will allow you to remove the keyboard and palm rest.

![Last 3 crews]({{ site.baseurl }}/images/t430/1_2_back_view_remove_last3_screws.jpg)

First, slightly shift the keyboard towards the screen.

 ![Shift the keyboard]({{ site.baseurl }}/images/t430/3_keyboard_shift.jpg)
 The keyboard is connected to the motherboard by a ribbon cable which easily
 detaches from the motherboard. 

![Keyboard disconnected]({{ site.baseurl }}/images/t430/4_keyboard_disconnected.jpg)

Remove these screws in order to remove the palm-rest.

![Palm-rest screws]({{ site.baseurl }}/images/t430/5_palmrest_screws.jpg)

The palm-rest is removed. Removing these screws will allow you to further detach the screen and the CPU cooler.

![Palm-rest removed]({{ site.baseurl }}/images/t430/6_palmrest_removed.jpg)

The screen and CPU with left speaker are removed.

![CPU cooler and screen removed]({{ site.baseurl }}/images/t430/7_remove_cpu_cooler_screen.jpg)

Flip the board and remove these screws too. This should allow you to get rid of  the aluminium part to access the SPI flash chips.

![Flipped board]({{ site.baseurl }}/images/t430/8_back_view.jpg)

Flip the board again. The SPI flash chips are located under this plastic.

![Flipped board again]({{ site.baseurl }}/images/t430/9_flipped_again.jpg)

Left chip corresponds to the "bottom" flash chip (8192 kb) and right corresponds to the "top" (4096 kb) chip, respectively. The top chip is 4MB and contains the BIOS and reset vector. The bottom chip is 8MB and has the [Intel Management Engine (ME)](https://www.flashrom.org/ME) firmware, plus the flash descriptor. To be on the safe side, you may want to disconnect CMOS battery before next steps.

![SPI flash chips]({{ site.baseurl }}/images/t430/10_spi_flash_chips.jpg)


First [download]({{ site.baseurl }}/Downloading)  or build (please see [general building]({{ site.baseurl }}/x230-maximized-building/) / [building x230]({{ site.baseurl }}/x230-maximized-building/))  the maximized board roms (top and bottom) for this board and verify their hashes.


Try to read the name on the top SPI flash chip. I was unable to do that. The dots on the chip help to identify the correct clip orientation. 

![SPI flash chips closed view]({{ site.baseurl }}/images/t430/11_spi_chips_closed_view.jpg)

 Then, connect the clip and ch341a programmer to the "top" (4096 kb) SPI flash chip. In my set up, the red wire should be where the dot is.

![Flashing 4 mb chip]({{ site.baseurl }}/images/t430/12_flash_4mb_spi_chip.jpg)

## Flashing

Use `[flasher]` of your choice (flashrom or flashprog -- see [Tool Interchangeability]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#tool-interchangeability)) with the programmer you selected ([programmer] -- see [Programmer Selection]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#recommended-programmers)):

```shell
sudo [flasher] --programmer [programmer]
```


 Here is my output.

![output top 4 mb chip]({{ site.baseurl }}/images/t430/13_ubuntu_output_4mb.jpg)

Find the chip and create a backup and verify it (For me the SPI flash chip is `YYY`):

```shell
sudo [flasher] --read ~/top.bin --programmer [programmer] --chip YYY && \
    sudo [flasher] --verify ~/top.bin --programmer [programmer] --chip YYY
```

If the files differ then try reconnecting your programmer to the SPI flash chip
 and make sure your flashrom/flashprog software is up to date.


If they are the same then write `t430-maximized-top.rom` to the SPI flash chip:

```shell
sudo [flasher] --programmer [programmer] --chip YYY --write ~/heads/build/x86/t430-maximized/t430-maximized-top.rom
```

 While everything goes well you should see the blue LED on the programmer.

![erase/write done]({{ site.baseurl }}/images/t430/14_programmer_flashing.jpg)


 Here is a successful attempt. 

![erase/write done]({{ site.baseurl }}/images/t430/15_successful_output_top.jpg)


Try to read the name on the bottom SPI flash chip. Then, connect the clip and
 ch341a programmer to the bottom SPI flash chip. 
 
![flashing bottom 8 mb chip]({{ site.baseurl }}/images/t430/16_flash_8mb_chip.jpg)
 
Use `[flasher]` of your choice (flashrom or flashprog -- see [Tool Interchangeability]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#tool-interchangeability)) with the programmer you selected ([programmer] -- see [Programmer Selection]({{ site.baseurl }}/SPI-Programmer-Best-Practices/#recommended-programmers)):

```shell
sudo [flasher] --programmer [programmer]
```

Here is my output.
 
![output bottom 8 mb chip]({{ site.baseurl }}/images/t430/17_ubuntu_output_8mb.jpg)

Find the chip and create a backup and verify it (For me the SPI flash chip is `ZZZ`):

```shell
sudo [flasher] --read ~/bottom.bin --programmer [programmer] --chip ZZZ && \
    sudo [flasher] --verify ~/bottom.bin --programmer [programmer] --chip ZZZ
```

The 8M bottom chip contains the ME firmware.  It is neutralized in maximized version. You can flash it specifying the same chip you found under ZZZ:
```shell
sudo [flasher] --programmer [programmer] --chip ZZZ --write ~/heads/build/x86/t430-maximized/t430-maximized-bottom.rom
```

If all goes well, you should see the keyboard LED flash, and within a second Heads will boot in its GUI. 

Two reboots are sometimes needed after flash. Force power off by holding the power button for 10 seconds. Since the memory training data was wiped by the content of the full flashed ROM, this is normal.

You should then follow through with [configuring keys]({{ site.baseurl }}/Configuring-Keys/).
