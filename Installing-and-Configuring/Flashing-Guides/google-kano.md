---
layout: default
title: Acer Chromebook Spin 714 (KANO)
permalink: /heads-flashing/
nav_order: 1
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Acer Chromebook Spin 714 (KANO)
===


## CCD with Suzyq cable
With Chromebooks there is an option called "Closed Case Debugging".  A special usb cable called a SuzyQ cable
is required.  A good guide on using the SuzyQ cable is 
[MrChromebox](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html).


### Disable Hardware Write Protection
The first thing you need to do is remove the battery of the laptop to disable hardware
write protection (but then we disable hardware write protection with the echo commands to /dev/ttyUSB0?)

TODO: insert images from https://github.com/linuxboot/heads/pull/2133

Connect the USB-C end of the Suzy-Q cable to the CCD port on your ChromeOS device
(usually left USB-C port) and the USB-A end to your Linux device
Verify the cable is properly connected:

```
$ ls /dev/ttyUSB*
/dev/ttyUSB0  /dev/ttyUSB1  /dev/ttyUSB2
```

This command should return 3 items: ttyUSB0, ttyUSB1, and ttyUSB2.
If not, then your cable is connected to the wrong port or is upside down.
Adjust and repeat command until output is as expected.

TODO: Do we need to run `gsctool` from ChromeOS?  I did not have to but maybe the previous owner of my Kano
device did it?  [source](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html#step-1-enabling-closed-case-debugging-ccd)

Now we disable software write protection:

```
sudo -s
echo "wp false" > /dev/ttyUSB0
echo "wp false atboot" > /dev/ttyUSB0
echo "ccd reset factory" > /dev/ttyUSB0
```

## Extra step (What is this really doing?)
KANO requires an extra step before we flash with flashrom:

```
# Get the baud rate
$ sudo stty -F /dev/ttyUSB2
speed 9600 baud; line = 0;

sudo minicom -D /dev/ttyUSB2

run: apshutdown
wait 5s
run: gpioset en_S5_rails 1
```

[source](https://forum.chrultrabook.com/t/flashrom-error-when-trying-to-unbrick-with-suzyq/8789/2?u=stonework5729)

## Backup
TODO: add section about backing up rom first.

## Flash heads

```
sudo flashrom --programmer raiden_debug_spi:target=AP,custom_rst=True \
  --chip "W25Q256JV_M" \
  --write heads-kano-202607081551-v0.2.1-3112-gb9dfd39.rom
```
