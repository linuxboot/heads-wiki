---
layout: default
title: Cleaning Intel Management Engine
permalink: /Clean-the-ME-firmware/
nav_order: 4
nav_exclude: true
---

What is the Intel Management Engine
===
The Intel ME is a coprocessor, running inside your Intel CPU, which is supposed to function as a out-of-band management system for your computer.  
It's based on a ARC/i386 core depending of the version, can be powered even when the main CPU is off and has more privileges than the CPU itself.  
More info: [Intel Active Management Technology](https://en.wikipedia.org/wiki/Intel_Active_Management_Technology)

How to disable/deactive most of it
===
The ME firmware sits on the second SPI flash chip of the x230 (the 8MB one). We cannot remove it completely, otherwise the machine will shut itself off after 30 minutes. We can, however, reduce it to the bare minimum necessary to keep it running, but without any malicious code in it (or so we hope, depending of what the ROMP and BUP modules really do...).

The initial step is to upgrade the proprietary BIOS to the last upgradeable version one for each platform.
As an example, for the x230, the latest upgradeable version would be [version 2.76](https://download.lenovo.com/pccbbs/mobiles/g2uj32us.iso) without [EC signature verification](https://support.lenovo.com/us/en/solutions/len-27764). Newer firmware version [won't permit to swap a x220 keyboard on the x230](https://github.com/hamishcoleman/thinkpad-ec/pull/130).  

Prepare a USB bootable disk by following [el torito instructions](https://askubuntu.com/questions/651281/write-bootable-bios-update-iso-to-usb-stick), then boot that prepared USB disk and upgrade the prioprietary firmware to latest available version following on screen instructions. Be sure to have a fully charged battery, be connected to power source prior of attempting to upgrade, else you will have to wait for the battery to be changed.

Once the proprietary firmware is updated to the latest available user ownable version, take a flash dump of the bottom SPI chip and verify that its backup is valid.

Flashrom can be downloaded on most linux distribution on the external laptop that will be used to flash the cleaned rom. (`sudo dnf install flashrom` or `sudo apt-get install flashrom`.

With a ch341a programmer, the command would look like the following:
`sudo flashrom -r ~/down.rom --programmer ch341a_spi && sudo flashrom -v ~/down.rom --programmer ch341a_spi`

If they match, clone this repo:  
`https://github.com/corna/me_cleaner.git`  
and then run:  
`python ~/me_cleaner/me_cleaner.py -c flash.bin`  
to check if it's a valid ME firmware.  
The output should be like:  

```
Full image detected
The ME/TXE region goes from 0x3000 to 0x4ff000
Found FPT header at 0x3010
Found 23 partition(s)
Found FTPR header: FTPR partition spans from 0x183000 to 0x24d000
ME/TXE firmware version 8.1.30.1350
Checking FTPR RSA signature... VALID
```

If it's not, replace the reprogrammer clip and try again :)  

Next, unlock the descriptor and ME regions with ifdtool. We consider here that you already build Heads through `make BOARD=x230`:
`~/heads/build/coreboot-4.8.1/util/ifdtool/ifdtool -u down.rom`
This produced a new unlocked rom under `down.rom.new`

Next, let's strip all the nasty bits:  

`python ~/me_cleaner/me_cleaner.py -r -t -d -S -O clean_flash.bin down.rom.new --extract-me extracted_me.rom`

Output:  

```
Full image detected
The ME/TXE region goes from 0x3000 to 0x4ff000
Found FPT header at 0x3010
Found 23 partition(s)
Found FTPR header: FTPR partition spans from 0x183000 to 0x24d000
ME/TXE firmware version 8.1.30.1350
Removing extra partitions...
Removing extra partition entries in FPT...
Removing EFFS presence flag...
Removing ME/TXE R/W access to the other flash regions...
Correcting checksum (0x7b)...
Reading FTPR modules list...
 UPDATE           (LZMA   , 0x1cf4f2 - 0x1cf6b0): removed
 ROMP             (Huffman, fragmented data    ): NOT removed, essential
 BUP              (Huffman, fragmented data    ): NOT removed, essential
 KERNEL           (Huffman, fragmented data    ): removed
 POLICY           (Huffman, fragmented data    ): removed
 HOSTCOMM         (LZMA   , 0x1cf6b0 - 0x1d648b): removed
 RSA              (LZMA   , 0x1d648b - 0x1db6e0): removed
 CLS              (LZMA   , 0x1db6e0 - 0x1e0e71): removed
 TDT              (LZMA   , 0x1e0e71 - 0x1e7556): removed
 FTCS             (Huffman, fragmented data    ): removed
 ClsPriv          (LZMA   , 0x1e7556 - 0x1e7937): removed
 SESSMGR          (LZMA   , 0x1e7937 - 0x1f6240): removed
Relocating FTPR from 0xd00 - 0xcad00 to 0xd00 - 0xcad00...
 Adjusting FPT entry...
 Adjusting LUT start offset...
 Adjusting Huffman start offset...
 Adjusting chunks offsets...
 Moving data...
The ME minimum size should be 98304 bytes (0x18000 bytes)
The ME region can be reduced up to:
 00003000:0001afff me
Setting the AltMeDisable bit in PCHSTRP10 to disable Intel ME...
Removing ME/TXE R/W access to the other flash regions...
Extracting and truncating the ME image to "extracted_me.rom"...
Checking the FTPR RSA signature of the extracted ME image... VALID
Checking the FTPR RSA signature... VALID
Done! Good luck!
```

After that, you got your new, cleaned up version of the ME firmware inside clean_flash.bin  
Flash it back on the SPI:

`sudo flashrom -w clean_flash.bin --programmer ch341a_spi`

You're now good to go :)
