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
 host tools and paths are now bleeding into the final ROM. An alternative build
 system is being proposed [here](https://github.com/osresearch/linux-builder),
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
short while, so get a cup of coffee and [install host build requirements packages as specified here](https://github.com/osresearch/heads/blob/master/.circleci/config.yml#L10-L11)

On a Fedora machine, [install host build requirements packages as specified here](https://github.com/osresearch/heads/blob/master/.gitlab-ci.yml.deprecated#L19).

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

 The resulting rom file will be either `./build/XXX/XXX.rom` or
  `./build/XXX/heads-XXX-vYYYY-gZZZZZZZ.rom` (`XXX` should be the name of your board in
  `./boards, vYYYY the pinned Heads version and ZZZZZZ the commit id from which your build comes from`).

Please continue to the corresponding flashing guide for your device.

More options and detail about Heads modules under [Makefile](https://github.com/osresearch/heads/blob/master/Makefile)

Downloading Heads
===
Alternatively, you can download ROMs directly from CircleCI for recent commits.
A lot of efforts have been put into CircleCI so that most of the most used 
community platforms have their ROMs generated and downloadable as artifacts
from CircleCI. Note that CircleCI keeps artifacts for 30 days after build time.
Considering Heads is a rolling release as of now, it is advised to wait a day
or two prior of flashing a ROM downloaded from CircleCI if you do not own a 
external programmer, to make sure everything is kosher. Maybe wait even longer
when its a question of coreboot upgrade or kernel upgrade, just to be sure.

Heads is hosted on github, and most of the Pull Requests (PR) are merged 
upstream after review. Most of the changes happening in the codebase are
policy related, and those policy changes can normally be tested easily 
on a specific platform it it needed and have no effect on platforms that
do not use it. The general advise here is to review the latest PR linked
to the commit that was merged and make sure that some [other owners of the
same platform you use](https://github.com/osresearch/heads/issues/692) has tested the changes if you are in doubt.

How to download
---

- Go to Heads GitHub repository and click on latest commit's green check
![2022-05-10-161753](https://user-images.githubusercontent.com/827570/167725941-e6fcad76-2549-4ffe-88ac-33f92545397b.png)
- Select your desired platform (hopefully, [choosing maximized builds over legacy counterparts](https://osresearch.net/Prerequisites#legacy-vs-maximized-boards) and choosing hotp variants if you already own a [compatible USB Security dongle](https://osresearch.net/Prerequisites#usb-security-dongles-aka-security-token-aka-smartcard)). Here, we choose x230-hotp-maximized as an example.
![2022-05-10-161811](https://user-images.githubusercontent.com/827570/167726540-d8ce8d1f-f25a-4ff2-b4e6-2e88e5051cd8.png)
- This brings us to CircleCI web interface. First, we scroll and click on the `Make Board` step so we can see the hash of the ROM we are about to download
![2022-05-10-161907](https://user-images.githubusercontent.com/827570/167726969-5ff7fdfc-81df-4a2e-b552-0ec2ec4aa659.png)
- This opens the build log and shows the last output of the board being built. Highlight/copy the hash for future comparison.
![2022-05-10-162016](https://user-images.githubusercontent.com/827570/167727116-a7559cd4-6db2-4fd2-a4a4-a254a4add0eb.png)
- Scroll back up and select the `Artifacts` tab.
![2022-05-10-162037](https://user-images.githubusercontent.com/827570/167727221-b158912b-c798-4002-8d9a-93a6fbf14f85.png)
- This opens the artifacts that were saved for that commit id. We are interested in the ROM we are interested to download, which is the full ROM in this upgrade example, as opposed to `bottom` and `top` ROM images which are aimed to be flashed externally.
![2022-05-10-162056](https://user-images.githubusercontent.com/827570/167727408-1e1c23bb-5afb-4ead-806f-7c65d58ab906.png)
- Save the link to your already prepared and mounted USB drive locally
![2022-05-10-162112](https://user-images.githubusercontent.com/827570/167727582-2c15cdc1-c1ec-4289-8548-7b9afc79a40b.png)
- Validate the checksum against saved checksum
![2022-05-10-162349](https://user-images.githubusercontent.com/827570/167727751-44d6ba06-29f5-48ea-b955-48db4edbe251.png)
- If you saved it locally, pass it over to sys-usb once you verified checksum for compartmentalization
![2022-05-10-162649](https://user-images.githubusercontent.com/827570/167727877-32606e55-4601-4ff8-ad3b-916cb8bde922.png)
- Mount your USB drive and move the ROM to it
![2022-05-10-162825](https://user-images.githubusercontent.com/827570/167727965-7a7e9e7f-73fa-4d4c-a0ac-83b30d38584d.png)
![2022-05-10-162915](https://user-images.githubusercontent.com/827570/167728027-a5918a1e-4c8d-478c-8365-a0b512da0944.png)
- Unmount your USB drive by pressing the eject button.
- Boot your Heads laptop, go into recovery shell to make sure you know what you are doing
  - If aiming to go to maximized firmware, make sure you have unlocked Intel Flash Descriptor (IFD) and ME regions on initial flash
    - This is compatible![CanBeFlashedToMaximizedRom](https://user-images.githubusercontent.com/827570/167728631-85a5ca9e-48f6-4d4f-8544-532fa75bf5d3.jpeg)
      - If you have no warning/error (see above), you can proceed: ![InternalUpgradeToMacimizedROM](https://user-images.githubusercontent.com/827570/167729694-6ff8da60-986a-4ec3-9b2d-4fa94e42d3fa.jpeg)
    - This is not compatible ![CantBeInternallyUpgradeToMaximizedROM](https://user-images.githubusercontent.com/827570/167728658-731362da-a676-4610-becb-ff94f2ff48b1.jpeg)
      - You either have to download a non-maximized ROM version, or flash externally the top and bootom ROMs of the maximized board.  














