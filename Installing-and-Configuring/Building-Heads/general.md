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
 since [v0.1.0](https://github.com/osresearch/heads/releases/tag/v0.1.0) since it
 achieved this goal. Unfortunately, things broke since release 0.2.1, since some
 host tools and paths are now bleeding into the final ROM. An [alternative build
 system is being proposed](https://github.com/osresearch/linux-builder),
 with it's own issues. More research and development needs to be done to bring
 Heads to be reproducible again, with some [additional thinking on reproducibility](https://github.com/osresearch/linux-builder/issues/1).
 Reproducible builds mean that you are supposed to get the same bit by bit output
 for the same commit. When this idea is confronted to time and inclusion of
 upstream code which also change and do not necessarily follow guidelines for
 reproducibility, things are more complicated then it seems. You can try to build
 0.2.1 for yourself today to see that it simply doesn't build anymore to get an
 idea of some of the problems of such concept if things care not carefully thought
 forward to be future proof, and get into [reproducible build milestone](https://github.com/osresearch/heads/milestone/1)
 to see current issues that need to be addressed. If you have redroducible build
 kung fu, please help.

The downside of the reproducibility goal is that the initial build can take a 
 very long time as it downloads and builds all of the its dependencies. One 
 issue right now is that it builds not just one, but *two* cross compilers and 
 as a result takes about 45 minutes.  Luckily subsequent builds only take about 
 30 seconds to produce a full coreboot and Linux ROM image, but that first ones 
 a doozy...

Heads builds will eventiually be fully reproducible again on any Linux-ish system
 ([OSX build is not supported](https://github.com/osresearch/heads/issues/96)).
 If you don't get the same hashes as reported on the release page, please 
 search/file an issue with the [reproducible build milestone](https://github.com/osresearch/heads/milestone/1).

With a vanilla Debian 11 or Ubuntu 20.04 install, such as a digitalocean
droplet, you need to first install some support tools. This takes a
short while, so get a cup of coffee and [install host build requirements packages as specified in the Debian continuous integration configuration](https://github.com/osresearch/heads/blob/master/.circleci/config.yml#L18)

On a Fedora machine, [install host build requirements packages as specified in the old Fedora-based continuous integration configuration](https://github.com/osresearch/heads/blob/master/.gitlab-ci.yml.deprecated#L19).

For emulation and analysis with UEFITool under Fedora, install `qemu` and `qt5-devel`:

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

Useful targets, stored under the `boards` directory of the git tree.


Generic
---

Generally, everything that is needed to flash the SPI flash of a board is a
 single rom generated through `make BOARD=$BOARD bootstrap` and `make BOARD=$BOARD` commands, where $BOARD is the
 name of the board that can be found under `board` directory of the git
 downloaded tree.

 Make Heads for another board (`XXX` should be the name of your board in ./boards and YY the number of CPUs you want to build with):

 ```Makefile
 make BOARD=XXX CPUS=YY
 ```

 The resulting rom file for a x86 board will be either `./build/x86/XXX/XXX.rom` or
  `./build/x86/XXX/heads-XXX-vYYYY-gZZZZZZZ.rom` (`XXX` should be the name of your board in
  `./boards, vYYYY the pinned Heads version and ZZZZZZ the commit id from which your build comes from`).

Please continue to the corresponding flashing guide for your device.

More options and detail about Heads modules under [Makefile]({{ site.baseurl }}/Makefile/)
