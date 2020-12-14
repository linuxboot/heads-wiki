---
layout: default
title: General Building
nav_order: 1
permalink: /general-building/
parent: Step 1 - Building Heads
grand_parent: Installing and configuring
---

<!-- markdownlint-disable MD033 -->
<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>
<!-- markdownlint-enable MD033 -->

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

Heads buils should be fully reproducible on any Linux-ish system
 ([OSX build is not supported](https://github.com/osresearch/heads/issues/96)).
 If you don't get the same hashes as reported on the release page, please file
 an issue against the [reproducible build milestone](https://github.com/osresearch/heads/milestone/1).

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

Builds of Heads should be reproducible unless issues are currently known,
 [see Heads milestone #1](https://github.com/osresearch/heads/milestone/1) for
 more detail, which means that Heads will build in the exact same way on
 different computers. Because of this, as a user, you can guarantee that Heads
 has built correctly and has not been tampered with.

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

Please continue to the corresponding flashing guide for your device.

More options and detail about Heads modules under [Makefile](/Makefile/)
