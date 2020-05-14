# Heads Installation Guide

![Heads booting on an x230](images/Heads_booting_on_an_x230.jpg)

## Why Heads?:

[Heads README](https://github.com/osresearch/heads/blob/master/README.md#heads-the-other-side-of-tails)

[Heads FAQ](https://trmm.net/Heads_FAQ)

## The Purpose of this Guide:

This guide explains how to flash Heads on the Thinkpad x230 with a ch341a programmer. However, much of the advice given here can be generalized for different boards and programmers.

As you read, remember that while the Thinkpad x230 has two SPI flash chips, many boards do not.

## What You Will Need:

* Supported motherboard or laptop (List of supported boards in `./boards`) 
* [Spi Programmer](https://trmm.net/SPI_flash): ch341a programmer or raspberry pi or bus pirate (ch341a is recommended for new users and can be found almost [anywhere](https://www.amazon.com/s?k=ch341a+programmer))
* Wires and a clip to connect your programmer of choice to the board’s SPI flash chip(s)
* If you plan to use a ch341a programmer you will need another computer to flash from (Try to use a recommended operating system: Qubes or Debian 9 or Fedora 30)
* USB flash drive
* Librem Key/Nitrokey for HOTP authentication (It is possible to use other security keys (Yubikey, etc.), but only TOTP will work)

## Building Heads:

Install the necessary support tools for your distribution.

On Debian or Ubuntu:

```
apt update
apt install -y \
    build-essential \
    zlib1g-dev uuid-dev libdigest-sha-perl \
    libelf-dev \
    bc \
    bzip2 \
    bison \
    flex \
    git \
    gnupg \
    iasl \
    m4 \
    nasm \
    patch \
    python \
    wget \
    gnat \
    cpio \
    ccache \
    pkg-config \
    cmake \
    libusb-1.0-0-dev \
    pkg-config \
    texinfo \
```

On Fedora:

```
dnf update
dnf install -y \
    @development-tools \
    gcc-c++ gcc-gnat zlib-devel perl-Digest-MD5 perl-Digest-SHA \
    uuid-devel pcsc-tools qemu ncurses-devel lbzip2 \
    libuuid-devel lzma \
    elfutils-libelf-devel \
    bc \
    bzip2 \
    bison \
    flex \
    git \
    gnupg \
    iasl \
    m4 \
    nasm \
    patch \
    python \
    wget \
    libusb-devel \
    cmake \
    pv \
    bsdiff \
    diffutils \ 
    texinfo
```

Install `qemu` and `qt5-devel` (for emulation and analysis with UEFITool respectively):

```
dnf install -y \
    qemu qt5-devel \
```

Clone the Heads repo:

```
git clone https://github.com/osresearch/heads.git
```

Move into the cloned repository:

```
cd heads
```

Builds of Heads are reproducible, which means that Heads will build in the exact same way on different computers. Because of this, as a user, you can guarantee that Heads has built correctly and has not been tampered with.

However, this also means that the first time you build Heads it must first build the compilers that it will use to build itself. If that seems complicated, don’t worry. The result is that the first build of Heads will take about an hour to complete. After the first build, building Heads will take less than a minute.

For the Thinkpad x230 there are two boards in `./boards`, x230-flash and x230. x230-flash is externally flashable and contains a smaller package that will let us boot to the Heads recovery shell. x230 is only internally flashable and contains all of Heads. Since we will end up using both x230-flash and x230, it makes the most sense to build both now.

Make both boards:

```
make BOARD=x230-flash
make BOARD=x230
```

Make Heads for another board (`XXX` should be the name of your board in ./boards):

```
make BOARD=XXX
```

The resulting rom file can be found in `./build/XXX/XXX.rom` (`XXX` should be the name of your board in `./boards`).


## External Flashing:

First remove the battery or cable powering your device. The Thinkpad x230 has two SPI flash chips that hold the BIOS, ME, etc. and are located under the palm rest. To access these chips, first remove the indicated screws on the back of the laptop.

Removing these screws will allow you to remove the keyboard and palm rest. 

The keyboard is connected to the motherboard by a ribbon cable which easily detaches from the motherboard. (The keyboard only needs to be removed so that the palm rest can be removed. After removing the palm rest, you can put the keyboard back.) 

The palm rest is also connected to the motherboard, but there is a little latch holding its ribbon cable. After undoing that latch, the palm rest should be fairly easy to remove.

Pull up the black plastic on the bottom right of the palm rest to reveal the two SPI flash chips.

The top chip is 4MB and contains the BIOS and reset vector. The bottom chip is 8MB and has the [Intel Management Engine (ME)](https://www.flashrom.org/ME) firmware, plus the flash descriptor.

Try to read the name on the top SPI flash chip. Then, connect the clip and ch341a programmer to the top SPI flash chip. Use flashrom to check the chip that you are connected to:

```
sudo flashrom -p ch341a_spi
```

Find the chip and read from it twice (For me the SPI flash chip is `YYY`):

```
sudo flashrom -p ch341a_spi -c “YYY” -r read11.bin
sudo flashrom -p ch341a_spi -c “YYY” -r read12.bin
```

Make sure that your programmer is reading correctly by checking if the files are the same:

```
diff read1.bin read2.bin
```

If the files differ then try reconnecting your programmer to the SPI flash chip and make sure your flashrom software is up to date.

If they are the same then write x230-flash.rom to the SPI flash chip:

```
sudo flashrom -p ch341a_spi -c “YYY” -w ~/heads/build/x230-flash/x230-flash.rom
```

Try to read the name on the bottom SPI flash chip. Then, connect the clip and ch341a programmer to the bottom SPI flash chip. Use flashrom to check the chip that you are connected to:

```
sudo flashrom -p ch341a_spi
```

Find the chip and read from the chip twice (For me the SPI flash chip is `ZZZ`):

```
sudo flashrom -p ch341a_spi -c “ZZZ” -r read21.bin
sudo flashrom -p ch341a_spi -c “ZZZ” -r read22.bin
```

Make sure that your programmer is reading correctly by checking if the files are the same:

```
diff read1.bin read2.bin
```

### Cleaning the ME:

If your reads match then you are good to move foreward with cleaning the ME firmware.

Clone the me_cleaner repo:

```
git clone https://github.com/corna/me_cleaner.git
```

Use me_cleaner to verify that the read from the SPI flash chip contains ME firmware: 

```
python me_cleaner.py -c flash21.bin
```

The output should be something like:

```
Full image detected
The ME/TXE region goes from 0x3000 to 0x4ff000
Found FPT header at 0x3010
Found 23 partition(s)
Found FTPR header: FTPR partition spans from 0x183000 to 0x24d000
ME/TXE firmware version 8.1.30.1350
Checking FTPR RSA signature... VALID
```

If the output is good then unlock the descriptor and ME regions with ifdtool:

```
~/heads/build/coreboot-4.8.1/util/ifdtool/ifdtool -u read21.bin
```

The resulting file will be called `read21.bin.new` and should be located in the same directory as `read21.bin`.

Remove all of the ME firmware that is not necessary to boot from the unlocked rom file:

```
python me_cleaner.py -r -t -d -S -O clean_flash.bin read21.bin.new --extract-me extracted_me.rom
```

Flash `clean_flash.bin` to the SPI flash chip:

```
flashrom -p ch341a_spi -c “ZZZ” -w clean_flash.bin
```

## Internal Flashing:

Reconnect power to the laptop and you should be able to boot into the Heads recovery shell.

Plug your USB flash drive into the laptop that you used to build Heads. If your USB drive is already formatted as ext4 or you are confident you can format it then just move the coreboot.rom file to the usb drive. Otherwise, find your usb drive using fdisk:

```
sudo fdisk -l
```

Format your usb drive as ext4 (My usb drive is /dev/sdb):

```
sudo mkfs.ext4 /dev/sdb1
```

These are the commands I used to create a directory ~/usb/ and mount my usb drive there, but you can mount it wherever you want:

```
mkdir ~/usb
sudo mount /dev/sdb1 ~/usb/
```

Move the full Heads rom file to the usb drive:

```
sudo cp ~/heads/build/x230/coreboot.rom ~/usb/
```

Insert the usb drive into the Thinkpad x230 and mount it:

```
mount-usb
```

You should now see the file coreboot.rom in /media:

```
ls /media/
```

Internally flash coreboot.rom (This command will write to both SPI flash chips as if they are one 12Mb chip):

```
flash.sh -c /media/coreboot.rom
```

Wait for the flashing to finish and you should be able to reboot into Heads!

