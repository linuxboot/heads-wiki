---
layout: default
title: Upgrading Heads
permalink: /Updating/
nav_order: 99
parent: Installing and configuring
---

![Flashing Heads on an x230 at HOPE]({{ site.baseurl }}/images/Flashing_Heads_on_an_x230_at_HOPE.jpg)

Upgrading Heads
===

The first time you install Heads, you'll need a
[hardware flash programmer](https://trmm.net/SPI_flash) to be able to
replace the existing vendor firmware.  Subsequent upgrades can be
performed via software, although you'll probably want a hardware programmer
since we don't have a fail-safe recovery mechanism in the event of
a bad flash or buggy firmware.

Additionally, *reflashing the firmware will change the TPM PCRs*.
This will require generating a new TPM TOTP token.  If you stored the decryption key for your drive in the TPM it will be lost.  Be sure you have your TPM owner's password and your disk encryption recovery key or passphrase available since the disk key is not accessible to the recovery shell.  This is by design.

Recovery shell
---

![Recovery shell]({{ site.baseurl }}/images/Recovery_shell.jpg)

If the flash protection bits are set correctly it is not possible to
rewrite the firmware from the normal OS.  You'll need to reboot
to the Heads [recovery shell](/RecoveryShell/). Hold 'r' on boot or choose 'Recovery Shell' in the heads GUI.

Internal Flashing
---

Reconnect power to the laptop and you should be able to boot into the Heads
 recovery shell.

Plug your USB flash drive into the laptop that you used to build Heads. If your
 USB drive is already formatted as ext4 or you are confident you can format it
 then just move the coreboot.rom file to the usb drive. Otherwise, find your usb
 drive using fdisk:

```shell
sudo fdisk -l
```

Format your usb drive as ext4 (My usb drive is /dev/sdb):

```shell
sudo mkfs.ext4 /dev/sdb1
```

These are the commands I used to create a directory ~/usb/ and mount my usb
 drive there, but you can mount it wherever you want:

```shell
mkdir ~/usb
sudo mount /dev/sdb1 ~/usb/
```

Move the full Heads rom file to the usb drive:

```shell
sudo cp ~/heads/build/x230/coreboot.rom ~/usb/
```

Insert the usb drive into the Thinkpad x230 and mount it:

```shell
mount-usb
```

You should now see the file coreboot.rom in /media:

```shell
ls /media/
```

Internally flash coreboot.rom (This command will write to both SPI flash chips
  as if they are one 12Mb chip):

```shell
flash.sh -c /media/coreboot.rom
```

Wait for the flashing to finish and you should be able to reboot into Heads!

Mounting the USB media
---

![insmod]({{ site.baseurl }}/images/insmod.jpg)

The Heads boot process does not have USB or network drivers by default
and neither does the recovery shell (although this might change).
You need to load the Linux kernel modules, which will change the
default module PCR 5:

```shell
insmod /lib/modules/ehci-hcd.ko
insmod /lib/modules/ehci-pci.ko
```

When you insert the drive you'll see a console message about the partitions
on the new device.  Typically it will be the first partition, `/dev/sdb1`,
or sometimes just `/dev/sdb` if there is no partition table.  Make a
directory and mount the device read only:

```shell
mkdir /media
mount -o ro /dev/sdb1 /media
```

Flashing the ROM
---

![Mount and flash]({{ site.baseurl }}/images/Mount_and_flash.jpg)

There is a helper script `/bin/flashrom-x230.sh` that uses the x230
flash ROM layout and the Heads modified version of `flashrom` to
write to the chip.  One of the modifications is to avoid touching or
reading the ME section, so it is not necessary to have used the
[ME cleaner](/Clean-the-ME-firmware/) or unlocked the flash descriptor.

```shell
flashrom-x230.sh /media/x230.full.rom
```

![Flashrom]({{ site.baseurl }}/images/Flashrom.jpg)

If all goes well it will write for about a minute and then report
success.  Due to hacks in `flashrom`, it does not read back what it
wrote to verify, so hopefully it worked.

Reboot and verify that the new firmware is running.  You'll be dropped
into the recovery shell immediately since the TPM TOTP secret will not
be unlocked.  Since the first boot after flashing will also adjust
the MRC cache, it is necessary to do a second reboot to ensure that
the TPM values are at their persistent state
([issue #150](https://github.com/osresearch/heads/issues/150) aims to fix this).

Regenerating the TOTP token
---

![TPM TOTP QR]({{ site.baseurl }}/images/TPM_TOTP_QR.jpg)

After the second post-flash reboot, generate a new token and store the
QR code in your phone by running:

```shell
sealtotp.sh
```

This needs the TPM owner password to be able to define the NVRAM space.
(todo: [issue #151](https://github.com/osresearch/heads/issues/151)).

Resealing the disk encryption keys (optional)
---

If you want to store the disk decryption key in the TPM you will need to do that now.  From the recovery shell run generic-init.  When you get to the standard boot menu and after you verify the TOTP, select 'm'
 to go to the full boot menu.  Select the option you want (usually the first),
 make it the default by hitting 'd' and also say 'y' when asked to reseal the
 disk keys.
