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

With the new Nix build system, building Heads has become more streamlined.

Please refer to the [Building Heads](https://github.com/linuxboot/heads/blob/master/README.md#building-heads) section in the [Heads README](https://github.com/linuxboot/heads/blob/master/README.md) for updated instructions on how to build Heads using the Nix build system's produced docker image reproducibly.

For more information, you can also check out the [pull request #1661](https://github.com/linuxboot/heads/pull/1661) which provides additional details and updates to the build process.

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
 single rom generated through `make BOARD=$BOARD` commands, where $BOARD is the
 name of the board that can be found under `board` directory of the git
 downloaded tree. You can do the build verbose by doing `make BOARD=$BOARD V=1`

 You can also run make in debug mode to trace errors in the buildsystem while you try to improve it through `make -d BOARD=$BOARD V=1`

 Make Heads for another board (`XXX` should be the name of your board in ./boards and YY the number of CPUs you want to build with):

 ```shell
 docker run -e DISPLAY=$DISPLAY --network host --rm -ti -v $(pwd):$(pwd) -w $(pwd) tlaurion/heads-dev-env:latest -- make BOARD=XXX CPUS=YY
 ```

 The resulting rom file for a x86 board will be either `./build/x86/XXX/XXX.rom` or
  `./build/x86/XXX/heads-XXX-vYYYY-gZZZZZZZ.rom` (`XXX` should be the name of your board in
  `./boards, vYYYY the pinned Heads version and ZZZZZZ the commit id from which your build comes from`).

Please continue to the corresponding flashing guide for your device.

More options and detail about Heads modules under [Makefile]({{ site.baseurl }}/Makefile/)
