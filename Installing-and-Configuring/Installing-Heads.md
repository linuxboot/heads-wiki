---
layout: default
title: Installing Heads
permalink: /Installing-Heads/
nav_order: 4
parent: Installing and configuring
---

![Flashing Heads on an x230 at HOPE]({{ site.url }}/images/Flashing_Heads_on_an_x230_at_HOPE.jpg)

Installing Heads
===

These instructions are only for the Lenovo Thinkpad x230 and require physical
 access to the hardware. There are risks in installation that might brick your
 system and cause loss of data. You will need another computer to perform the
 flashing and building steps. If you want to experiment, consider
 [Emulating Heads](Emulating-Heads.md) with qemu before installing it on your
 machine.

There are five major steps:

* Flashing the boot ROM
* Taking ownership of the TPM
* Installing Qubes
* Sealing disk encryption keys
* Signing Qubes installation

![Underside of the x230]({{ site.url }}/images/Underside_of_the_x230.jpg)

Unplug the system and remove the battery while you're disassembling the machine!
 You'll need to remove the palm rest to get access to the SPI flash chips, which
 will require removing the keyboard. There are seven screws marked with keyboard
 and palm rest symbols.

![Keyboard tilted up]({{ site.url }}/images/Keyboard_tilted_up.jpg)

The keyboard tilts up on a ribbon cable. You can keep the cable installed,
 unless you want to swap the keyboard for the nice x220 model.

![Ribbon cable]({{ site.url }}/images/Ribbon_cable.jpg)

The palm rest trackpad ribbon cable needs to be disconnected. Flip up the
 retainer and pull the cable out. It shouldn't require much force. Once the
 palmrest is removed you can replace the keyboard screws and operate the machine
 without the palm rest. Since the thinkpad has the trackpoint, even mouse
 applications will still work fine.

![Flash chips]({{ site.url }}/images/Flash_chips.jpg)

There are two SPI flash chips hiding under the black plastic, labelled "SPI1"
 and "SPI2". The top one is 4MB and contains the BIOS and reset vector. The
 bottom one is 8MB and has the [Intel Management Engine (ME)](https://www.flashrom.org/ME)
 firmware, plus the flash descriptor.

Using a chip clip and a [SPI programmer](https://trmm.net/SPI_flash), dump the
 existing ROMs to files. Dump them again and compare the different dumps to be
 sure that were no errors. Maybe dump them both a third time, just to be safe.

![Flashing x230 SPI flash]({{ site.url }}/images/Flashing_x230_SPI_flash.jpg)

Ok, now comes the time to write the 4MB `build/x230-flash/x230-flash.rom` file
 to SPI2 chip. With my programmer and minicom, I hit i to verify that the flash
 chip signature is correctly read a few times, and then send `u0 400000`↵ to
 initiate the upload. I then drop to a shell with Control-A J and finally send
 the file with `pv x230.rom > /dev/ttyACM0`↵. A minute later, I resume minicom
 and hit i again to check that the chip is still responding.

Move the clip to the SPI1 chip. Read out the chip using `flashrom -r`, keep
 a copy as backup and run `ifdtool -u` on it to enable writing to the flash
 from software later. Also, [clean the ME firmware](Clean-the-ME-firmware).
 This will wipe out the official Intel firmware, leaving only a stub of it to
 bring up the Sandybridge CPU before shutting down the ME. As far as I can
 tell there are no ill effects. Flash back your modified 8MB image. This
 time you’ll send the command u0 800000↵. (you can also use the
 [Skulls project](https://github.com/merge/skulls/tree/master/x230)'s
 `external_install_bottom.sh -m` script to do all this work on the SPI1 chip
 automatically).

Finally, remove the programmer, connect the power supply and try to reboot.

If all goes well, you should see the keyboard LED flash, and within a second the
 Heads recovery splash screen will appear. It currently drops you immediately
 into the shell, to allow you to flash the full 12MB `x230.bin` Heads rom (or
 `build/x230/coreboot.rom` if you've built it locally). If it doesn't work,
 well, sorry about that. Please let me know what the symptoms are or what
 happened during the flashing.

Congratulations! You now have a Coreboot + Heads Linux machine. Adding your own
 signing key, installing Qubes and configuring tpmtotp are the next steps.

On insert a USB drive containing the 12MB Heads rom and mount it using:

```shell
mount-usb
```

This will load the USB kernel modules and mount your drive at `/media`.
