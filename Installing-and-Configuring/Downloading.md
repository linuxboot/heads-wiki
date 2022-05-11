---
layout: default
title: Step 1 - Downloading Heads
nav_order: 2
permalink: /Downloading/
parent: Installing and configuring
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

What to download
===
- Boot your Heads laptop, go into [Recovery Shell](/Recovery) to make sure if you should download [Legacy or Maximized](/Prerequisites#legacy-vs-maximized-boards) board ROMs:
  - If aiming to go to Maximized firmware, make sure you have unlocked Intel Flash Descriptor (IFD) and ME regions on initial flash
    - *This is compatible* ![CanBeFlashedToMaximizedRom](https://user-images.githubusercontent.com/827570/167728631-85a5ca9e-48f6-4d4f-8544-532fa75bf5d3.jpeg)
      - If you have no warning/error (see above), you can proceed with a manual flashrom command to migrate once and for all to Maximized board: ![InternalUpgradeToMacimizedROM](https://user-images.githubusercontent.com/827570/167729694-6ff8da60-986a-4ec3-9b2d-4fa94e42d3fa.jpeg)
    - *!!!This is not compatible!!!* ![CantBeInternallyUpgradeToMaximizedROM](https://user-images.githubusercontent.com/827570/167728658-731362da-a676-4610-becb-ff94f2ff48b1.jpeg)
      - You either have to download a non-maximized ROM version, or have to flash externally the (xx30 boards: top and bottom) ROM(s) for your platform's maximized board configuration. This needs to be done once, after which you can upgrade from the GUI.

Downloading Heads from CircleCI
===
Alternatively, you can download ROMs directly from CircleCI for recent commits.

A lot of efforts have been put into CircleCI so that most of the most used 
community platforms have their ROMs generated and downloadable as artifacts
from CircleCI. 

Note that CircleCI keeps artifacts for 30 days after build time.

Considering Heads is a rolling release as of now, it is advised to wait a day
or two prior of flashing a ROM downloaded from CircleCI if you do not own a 
external programmer, to make sure everything absolutely no regression has been 
reported. Maybe wait even longer when last merge was linked to coreboot upgrade 
or kernel upgrade, just to be sure. Watch the [most recently reported issues](https://github.com/osresearch/heads/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc).

Heads project is hosted on GitHub where most of the Pull Requests (PR) are 
[merged and closed after review(purple)](https://github.com/osresearch/heads/pulls?q=is%3Apr+is%3Aclosed+sort%3Aupdated-desc). 

Most of the changes happening in the codebase are policy related (scripts).
Those changes can normally be tested on any other platform and are normally
tested at least on two different platforms to make sure no regression happen.
The general advise here is to review the latest PR linked
to last commit that was merged and make sure that some [other owners of the
same platform you use](https://github.com/osresearch/heads/issues/692) have 
tested the changes if you are in doubt.

If you do not own an [external programmer](/Installing-and-Configuring/Prerequisites), please
be less adventurous and make sure that no coreboot/linux version was recently
pushed without other board owners having tested and reported positive results
for your board.

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

Upgrading firmware
===
The documentation related to upgrading is [here](/Updating)











