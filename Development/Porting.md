---
layout: default
title: Porting Heads
permalink: /Porting/
nav_order: 3
parent: Development
---

To add a new board to the Heads build hopefully you only need to modify
the coreboot configuration and add a top-level image configuration.

* Copy one of the existing image configurations, such as `config/x230-qubes.config`
to your new image, `config/newarch-qubes.config`, and edit the file to change
`BOARD=x230` to `BOARD=newarch`.

* Either copy one of the existing config files, such as the
`config/coreboot-x230.config` to `config/coreboot-newarch.conf`,
or use your existing coreboot `.config` file in its place.  You'll want to
be sure that it is using a Linux payload named `./bzImage` and a Linux initrd
named `./initrd.cpio.xz`.  No command line options are necessary.

* If you want to use `menuconfig` to reconfigure the coreboot file,
it is a little tricky since we have an external config file.  From the
top level of the Heads directory you can run:

```shell
make \
  -C build/coreboot-4.5 \
  DOTCONFIG=../../config/coreboot-newarch.config \
  menuconfig
```

* [Source](https://github.com/osresearch/heads/issues/667#issuecomment-638971582)
 For Lenovo yx30 (t430, x230 and others, x230 tested only here) have the
 following space available for BIOS region in IFD per output of
 `ifdtool -x bottom.rom` extracted from original bottom SPI chip, initially
 upgraded to latest BIOS version (2.75 for x230):

```text
Found Region Section
FLREG0:    0x00000000
  Flash Region 0 (Flash Descriptor): 00000000 - 00000fff
FLREG1:    0x0bff0500
  Flash Region 1 (BIOS): 00500000 - 00bfffff
FLREG2:    0x04ff0003
  Flash Region 2 (Intel ME): 00003000 - 004fffff
FLREG3:    0x00020001
  Flash Region 3 (GbE): 00001000 - 00002fff
FLREG4:    0x00001fff
  Flash Region 4 (Platform Data): 00fff000 - 00000fff (unused)
```

Important notes:

* We cannot change the CBFS_REGION and make it larger alone (bigger BIOS)
  without having IFD regions adapted so that ME region smaller and freed space
  being given to the BIOS region. Providing IFD.bin alone could resolve that.
  Ignoring this can result in a device booting with no screen output from
  coreboot or not properly resuming from suspend.
* When calling flash.sh, we are actually asking flashrom to take the image.rom
  and flash the BIOS region out of it.
* Consequently, it seems that flashrom doesn't validate that there is enough
  space and just overwrites what is there in SPI flash, resulting in a
  non-booting machine.

Solution:

* We could have generic flash script (common to xx30) flash both Flash
 Descriptor and BIOS regions from prepared xx30 images.
* Since those IFD regions are aligned, this means that flashed IFD
 should probably become:

 ```text
        Flash Region 1 (BIOS): 00500000 - 0001b000
        Flash Region 2 (Intel ME): 00003000 - 0001afff
  ```

* But that means that ME will need to be neutered and flashed externally prior
  of flashing the BIOS region and IFD over unlocked IFD.  Instructions for this
  external flashing use case, freeing and modifying ifd to reduce ME size and
  expend BIOS region in consequence of liberated space can be [found here](https://github.com/corna/me_cleaner/wiki/External-flashing#neutralize-and-shrink-intel-me-useful-only-for-coreboot)

* Run `make BOARD=$BOARD_DIR` (where `$BOARD_DIR` is the board directory
  (`x230`, `x230-flas`h and others found under `./boards` directory) to setup
  the coreboot tree, using the new coreboot config file.  This will create the
  output directory `build/$BOARD_DIR/*.rom`, the rom name should be named
  `coreboot.rom`.

* If things don't work, please open an issue on [https://github.com/osresearch/heads/issues](https://github.com/osresearch/heads/issues)
