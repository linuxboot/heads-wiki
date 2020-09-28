---
layout: default
title: Building Heads
permalink: /Building/
nav_order: 2
parent: Installing and configuring
---

Building Heads
===

Heads is supposed to be a [reproducible build](https://reproducible-builds.org/)
 and as of [v0.1.0](https://github.com/osresearch/heads/releases/tag/v0.1.0) it
 achieved this goal.  The downside is that the initial build can take a very
 long time as it downloads and builds all of the its dependencies.  One issue
 right now is that it builds not just one, but *two* cross compilers and as a
 result takes about 45 minutes.  Luckily subsequent builds only take about 30
 seconds to produce a full coreboot and Linux ROM image, but that first ones a
 doozy...

With a vanilla Debian 9 or Ubuntu 16.04 install, such as a digitalocean
droplet, you need to first install some support tools. This takes a
short while, so get a cup of coffee and
[install host build requirements packages as specified here](https://github.com/osresearch/heads/blob/master/.circleci/config.yml#L10-L11)

On a Fedora machine, [install host build requirements packages as specified here](https://github.com/osresearch/heads/blob/master/.gitlab-ci.yml#L19).

For emulation and analysis with UEFITool, install `qemu` and `qt5-devel`:

```shell
dnf install -y qemu qt5-devel
```

Clone the tree:

```shell
git clone https://github.com/osresearch/heads
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


Useful targets, stored under the `board` directory of the git tree.

Generated roms are generally found under build/$BOARD/$BOARD.rom

Generic
---

Generally, everything that is needed to flash the SPI flash of a board is a
 single rom generated through `make BOARD=$BOARD` command, where $BOARD is the
 name of the board that can be found under `board` directory of the git
 downloaded tree.

 Make Heads for another board (`XXX` should be the name of your board in ./boards):

 ```Makefile
 make BOARD=XXX
 ```

 The resulting rom file will be either `./build/XXX/XXX.rom` or
  `./build/XXX/coreboot.rom` (`XXX` should be the name of your board in
  `./boards`).

Make for a specific configuration
---

Some boards have a two SPI flash chip configuration and need special care.

x230
----


For the Thinkpad x230 there are two boards in `./boards`, x230-flash and x230.
 x230-flash is externally flashable and contains a smaller package that will
 let us boot to the Heads recovery shell. x230 is only internally flashable and
 contains all of Heads. Since we will end up using both x230-flash and x230, it
 makes the most sense to build both now.

Initial SPI2 (4MB) flash chip
-----

x230 boards needs their 4MB SPI2 to be initially externally flashed,
 while the 12MB rom needs to be flashed internally from within Heads to make
 sure to not screw up with ME, contained in the SPI1 flash (8MB bottom flash
 chip under keyboard)

The following make command generates a self-contained, externally flashable rom
 for the SPI2 (4MB BIOS, top SPI flash under keyboard).

```Makefile
make BOARD=x230-flash
```

Resulting rom is found under build/x230-flash/x230-flash.rom

Subsequent flashing (upgrades)
-----

The following make command will generate a 12MB `coreboot.rom` under the
build/x230 directory.

```Makefile
make BOARD=x230
```

The `coreboot.rom` is the one needed to flash rom updates from within Heads in
respect of ME. This is done with the help of the `flashrom-x230.sh` script from
Heads recovery shell.

Please continue to the corresponding flashing guide for your device.


More options and detail about Heads modules under [Makefile](/Makefile.md)
