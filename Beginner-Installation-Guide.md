---
layout: default
title: Beginner Installation Guide
permalink: /Beginner-Installation-Guide/
nav_order: 2
---

# Heads Installation Guide

![Heads booting on an x230](images/Heads_booting_on_an_x230.jpg)

## Why Heads

[Heads README](https://github.com/osresearch/heads/blob/master/README.md#heads-the-other-side-of-tails)

[Heads FAQ](https://trmm.net/Heads_FAQ)

## The Purpose of this Guide

This guide explains how to flash Heads on the Thinkpad x230 with a ch341a
 programmer. However, much of the advice given here can be generalized for
 different boards and programmers.

As you read, remember that while the Thinkpad x230 has two SPI flash chips,
 many boards do not.

## What You Will Need

* Supported motherboard or laptop (List of supported boards in `./boards`)
* [Spi Programmer](https://trmm.net/SPI_flash): ch341a programmer or raspberry
 pi or bus pirate (ch341a is recommended for new users and can be found almost
 [anywhere](https://www.amazon.com/s?k=ch341a+programmer))
* Wires and a clip to connect your programmer of choice to the board’s SPI flash
 chip(s)
* If you plan to use a ch341a programmer you will need another computer to flash
 from (Try to use a recommended operating system: Qubes or Debian 9 or Fedora 30)
* USB flash drive
* Librem Key/Nitrokey for HOTP authentication (It is possible to use other
  security keys (Yubikey, etc.), but only TOTP will work)

## Building Heads

Install the necessary support tools for your distribution.

* [Debian or Ubuntu](https://github.com/osresearch/heads/blob/master/.circleci/config.yml#L11)
* [Fedora](https://github.com/osresearch/heads/blob/master/.gitlab-ci.yml#L19)

Install `qemu` and `qt5-devel` (for emulation and analysis with UEFITool
  respectively):

```shell
dnf install -y \
    qemu qt5-devel \
```

Clone the Heads repo:

```shell
git clone https://github.com/osresearch/heads.git
```

Move into the cloned repository:

```shell
cd heads
```

Builds of Heads are reproducible, which means that Heads will build in the exact
 same way on different computers. Because of this, as a user, you can guarantee
 that Heads has built correctly and has not been tampered with.

However, this also means that the first time you build Heads it must first build
 the compilers that it will use to build itself. If that seems complicated,
 don’t worry. The result is that the first build of Heads will take about an
 hour to complete. After the first build, building Heads will take less than a
 minute.

For the Thinkpad x230 there are two boards in `./boards`, x230-flash and x230.
 x230-flash is externally flashable and contains a smaller package that will
 let us boot to the Heads recovery shell. x230 is only internally flashable and
 contains all of Heads. Since we will end up using both x230-flash and x230, it
 makes the most sense to build both now.

Make both boards:

```Makefile
make BOARD=x230-flash
make BOARD=x230
```

Make Heads for another board (`XXX` should be the name of your board in ./boards):

```Makefile
make BOARD=XXX
```

The resulting rom file will be either `./build/XXX/XXX.rom` or
 `./build/XXX/coreboot.rom` (`XXX` should be the name of your board in
 `./boards`).

## External Flashing

First remove the battery or cable powering your device. The Thinkpad x230 has
 two SPI flash chips that hold the BIOS, ME, etc. and are located under the
 palm rest. To access these chips, first remove the indicated screws on the back
 of the laptop.

Removing these screws will allow you to remove the keyboard and palm rest.

The keyboard is connected to the motherboard by a ribbon cable which easily
 detaches from the motherboard. (The keyboard only needs to be removed so that
 the palm rest can be removed. After removing the palm rest, you can put the
 keyboard back.)

The palm rest is also connected to the motherboard, but there is a little latch
 holding its ribbon cable. After undoing that latch, the palm rest should be
fairly easy to remove.

Pull up the black plastic on the bottom right of the palm rest to reveal the two
 SPI flash chips.

The top chip is 4MB and contains the BIOS and reset vector. The bottom chip is
 8MB and has the [Intel Management Engine (ME)](https://www.flashrom.org/ME)
  firmware, plus the flash descriptor.

Try to read the name on the top SPI flash chip. Then, connect the clip and
 ch341a programmer to the top SPI flash chip. Use flashrom to check the chip
  that you are connected to:

```shell
sudo flashrom -p ch341a_spi
```

Find the chip and read from it twice (For me the SPI flash chip is `YYY`):

```shell
sudo flashrom -r ~/top.bin --programmer ch341a_spi -c YYY && \
    sudo flashrom -v ~/top.bin --programmer ch341a_spi -c YYY
```

If the files differ then try reconnecting your programmer to the SPI flash chip
 and make sure your flashrom software is up to date.

If they are the same then write `x230-flash.rom` to the SPI flash chip:

```shell
sudo flashrom -p ch341a_spi -c “YYY” -w ~/heads/build/x230-flash/x230-flash.rom
```

Try to read the name on the bottom SPI flash chip. Then, connect the clip and
 ch341a programmer to the bottom SPI flash chip. Use flashrom to check the chip
  that you are connected to:

```shell
sudo flashrom -p ch341a_spi
```

Find the chip and read from the chip twice (For me the SPI flash chip is `ZZZ`):

```shell
sudo flashrom -r ~/bottom.bin --programmer ch341a_spi -c ZZZ && \
    sudo flashrom -v ~/bottom.bin --programmer ch341a_spi -c ZZZ
```

### Cleaning the ME

If your reads match then you are good to move foreward with cleaning the ME
 firmware.

Clone the me_cleaner repo:

```shell
git clone https://github.com/corna/me_cleaner.git
```

Use me_cleaner to verify that the read from the SPI flash chip contains ME
 firmware:

```shell
python me_cleaner.py -c ~/bottom.bin
```

The output should be something like:

```text
Full image detected
The ME/TXE region goes from 0x3000 to 0x4ff000
Found FPT header at 0x3010
Found 23 partition(s)
Found FTPR header: FTPR partition spans from 0x183000 to 0x24d000
ME/TXE firmware version 8.1.30.1350
Checking FTPR RSA signature... VALID
```

If the output is good then unlock the descriptor and ME regions with ifdtool:

```shell
~/heads/build/coreboot-4.8.1/util/ifdtool/ifdtool -u ~/bottom.bin
```

The resulting file will be called `bottom.bin.new` and should be located in the
 same directory as `bottom.bin`.

Remove all of the ME firmware that is not necessary to boot from the unlocked
 rom file:

```shell
python me_cleaner.py -r -t -d -S -O clean_flash.bin bottom.bin.new --extract-me extracted_me.rom
```

Flash `clean_flash.bin` to the SPI flash chip:

```shell
flashrom -p ch341a_spi -c “ZZZ” -w clean_flash.bin
```

## Internal Flashing

Reconnect power to the laptop and you should be able to boot into the Heads
 recovery shell.

Plug your USB flash drive into the laptop that you used to build Heads. If your
 USB drive is already formatted as ext4 or you are confident you can format it
 then just move the coreboot.rom file to the usb drive. Otherwise, find your usb
 drive using fdisk:

```shell
sudo fdisk -l
```

Format your usb drive as ext4 (My usb drive is /dev/sdb):

```shell
sudo mkfs.ext4 /dev/sdb1
```

These are the commands I used to create a directory ~/usb/ and mount my usb
 drive there, but you can mount it wherever you want:

```shell
mkdir ~/usb
sudo mount /dev/sdb1 ~/usb/
```

Move the full Heads rom file to the usb drive:

```shell
sudo cp ~/heads/build/x230/coreboot.rom ~/usb/
```

Insert the usb drive into the Thinkpad x230 and mount it:

```shell
mount-usb
```

You should now see the file coreboot.rom in /media:

```shell
ls /media/
```

Internally flash coreboot.rom (This command will write to both SPI flash chips
  as if they are one 12Mb chip):

```shell
flash.sh -c /media/coreboot.rom
```

Wait for the flashing to finish and you should be able to reboot into Heads!
