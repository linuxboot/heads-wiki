![Flashing Heads on an x230 at HOPE](https://pbs.twimg.com/media/CoKJhJHUkAA-MtS.jpg)

Installing Heads
===
These instructions are only for the Lenovo Thinkpad x230 and require physical access to the hardware. There are risks in installation that might brick your system and cause loss of data. You will need another computer to perform the flashing and building steps. If you want to experiment, consider [Emulating Heads](/Emulating-Heads) with qemu before installing it on your machine.

There are five major steps:
* Flashing the boot ROM
* Taking ownership of the TPM
* Installing Qubes
* Sealing disk encryption keys
* Signing Qubes installation

![Underside of the x230](https://farm9.static.flickr.com/8778/28686026815_6931443f6c.jpg)

Unplug the system and remove the battery while you're disassembling the machine! You'll need to remove the palm rest to get access to the SPI flash chips, which will require removing the keyboard. There are seven screws marked with keyboard and palm rest symbols.

![Keyboard tilted up](https://farm9.static.flickr.com/8584/28653973936_9cffb2a34e.jpg)

The keyboard tilts up on a ribbon cable. You can keep the cable installed, unless you want to swap the keyboard for the nice x220 model.

![Ribbon cable](https://farm8.static.flickr.com/7667/28608155181_fa4a2bfe45.jpg)

The palm rest trackpad ribbon cable needs to be disconnected. Flip up the retainer and pull the cable out. It shouldn't require much force. Once the palmrest is removed you can replace the keyboard screws and operate the machine without the palm rest. Since the thinkpad has the trackpoint, even mouse applications will still work fine.

![Flash chips](https://farm9.static.flickr.com/8581/28401826120_bd8a84e508.jpg)

There are two SPI flash chips hiding under the black plastic, labelled "SPI1" and "SPI2". The top one is 4MB and contains the BIOS and reset vector. The bottom one is 8MB and has the [Intel Management Engine (ME)](https://www.flashrom.org/ME) firmware, plus the flash descriptor.


Using a chip clip and a [SPI programmer](https://trmm.net/SPI_Flash), dump the existing ROMs to files. Dump them again and compare the different dumps to be sure that were no errors. Maybe dump them both a third time, just to be safe.

![Flashing x230 SPI flash](https://farm3.staticflickr.com/2889/33186538873_c1290ca6ec_z_d.jpg)

Ok, now comes the time to write the 4MB `x230.coreboot.bin` file to SPI2 chip. With my programmer and minicom, I hit i to verify that the flash chip signature is correctly read a few times, and then send `u0 400000`↵ to initiate the upload. I then drop to a shell with Control-A J and finally send the file with `pv x230.rom > /dev/ttyACM0`↵. A minute later, I resume minicom and hit i again to check that the chip is still responding.

Move the clip to the SPI1 chip and flash the 8 MB `x230.me.bin` (TODO: document how to produce this with me cleaner -> [Clean the ME firmware](Clean-the-ME-firmware)). This time you'll send the command `u0 800000`↵. This will wipe out the official Intel firmware, leaving only a stub of it to bring up the Sandybridge CPU before shutting down the ME. As far as I can tell there are no ill effects other than an inability to power off the machine without using the power switch.

Finally, remove the programmer, connect the power supply and try to reboot.

If all goes well, you should see the keyboard LED flash, and within a second the Heads splash screen appear. It currently drops you immediately into the shell, since the boot script portion has not yet been implemented. If it doesn't work, well, sorry about that. Please let me know what the symptoms are or what happened during the flashing.

Congratulations! You now have a Coreboot + Heads Linux machine. Adding your own signing key, installing Qubes and configuring tpmtotp are the next steps and need to be written.

Adding your PGP key
===
Heads uses your own GPG key to sign updates and as a result it needs the
key stored in the `initrd.cpio` file that is built as part of the ROM image.
In the future ([issue #182](https://github.com/osresearch/heads/issues/182))
it might be possible to install your own key from inside the recovery
shell; this would be preferable for many reasons.

Insert your Yubikey into the build machine and in the `heads` build directory,
clean out the old files, then invoke GPG's the "Card Edit" function with
it targetting the local directory:

```
rm ./initrd/.gnupg/*
gpg --card-edit --homedir ./initrd/.gnupg
```

Go into "Admin" mode and generate a new key inside the Yubikey:

```
admin
generate
```

Since this key can be replaced by replacing the ROM, it is not necessary
to make a backup unless you want to.
This will prompt you for the admin pin (`12345678` by default) and then
the existing pin (`123456`).  Follow the other prompts and eventually
you should have a key in `initrd/.gnupg/`




Configuring the TPM
===
There aren't very many good details on how to setup TPMs, so this section could use some work.

Taking ownership
---
If you've acquired the machine from elsewhere, you'll need to establish physical presence, perform a force clear and take ownership with your own password. Should the storage root key (SRK) be set to something other than the well-known password?

```
tpm-reset
```

There is something weird with enabling, presence and disabling. Sometimes reboot fixes the state.

tpmtotp
---

![TPMTOTP QR code](https://pbs.twimg.com/media/Cr8x7f6WEAEbBdq.jpg)

Once you own the TPM and have the final version of the `x230.rom` flashed, run `seal-totp` to generate a random secret, seal it with the current TPM PCR values and store the sealed value in the TPM's NVRAM. This will generate a QR code that you can scan with your google authenticator application and use to validate that the boot block, rom stage and Linux payload are un-altered.

![TPMTOTP output](https://farm8.static.flickr.com/7564/28580109172_5bd759f336.jpg)

On the next boot, or if you run `unseal-totp`, the script will extract the sealed blob from the NVRAM and the TPM will validate that the PCR values are as expected before it unseals it. If this works, the current TOTP will be computed and you can compare this one-time-password against the value that your phone generates.

This does not eliminate all firmware attacks (such as evil maid ones that replace the SPI flash chip), but when combined with the WP# pin and BP bits should eliminate a software only attack.

Installing Qubes
===
![Heads splash screen](https://farm3.staticflickr.com/2890/33999478295_5738432e82_z_d.jpg)

Boot into the recovery shell (hit 'r' at the prompt before the normal startup script tries to run) and plug the USB stick with the R3.2 install media into one of the USB3 ports (on the left side of the x230) and run the helper script to start the Qubes installer:

```
qubes-install
```

If that completes with no errors it will launch the Xen hypervisor from the x230's ROM image and start the Qube's installer.  The first few seconds are run with an archaic video mode, so things appear a little weird, but once the dom0 kernel initializes the graphics it should look right.

![Qubes partitioning](https://farm3.staticflickr.com/2858/33156102504_7fa25661c4_z_d.jpg)

My recommended partitioning scheme is to use LVM and to allocate 1G for `/boot` since it will hold the dm-verity hashes, 48G for `/`, 8G for swap and the rest for `/home`.  Don't adjust the filesystem labels or the volume group; this will be used by the startup script.

![Disk encryption recovery key](https://farm3.staticflickr.com/2883/33869761321_6c853894da_z_d.jpg)

The disk encrypt password that you enter here will be used as the
"recovery password" later.  It should be a long value since you won't
have to enter it very often; only when upgrading the Heads firmware
or if there is a need to recover the disk on an external machine.
You will need it again shortly, so don't lose it yet.

![Signing Qubes binaries in /boot](https://farm3.staticflickr.com/2939/33999478305_f0f61a4408_z_d.jpg])

Once Qubes has finished installing, you'll need to reboot into the Heads recovery shell.  The first reboot will fail with errors about "`/boot/boot.hashes does not exist`", since it doesn't..  The first step is to copy the Heads Xen to the `/boot` drive, sign them and let Qubes finish its initialization.

```
mount -o rw,remount /boot
cp /bin/xen.gz /boot/xen-4.6.4-heads.gz
mount -o ro,remount /boot
qubes-boot /boot/xen-4.6.4-heads.gz /boot/vm... /boot/initramfs...
```

You will need to input the disk recovery key here (almost for the last time),
and this should start the final stage of the Qubes installer.  Under
`Configure Qubes` you should select `Create USB qube holding all USB controllers` so that they are protected from outside devices.  This step takes a little
while as the templates are configured...

Eventually this will be done and you can click "Finish", then Qubes will
give you a login screen with your login password.

Now we need to make a small change to have Qubes' initramfs correctly find the TPM encrypted keys.  Open a dom0 terminal (click on the Q in the upper left and select `Terminal Emulator`).  In this shell run this perl command and rebuild the initrd with dracut (takes several seconds):

TODO: how to force all of crypttab to be emitted?

```
sudo perl -pi -e 's: none: /secret.key' /etc/crypttab
sudo dracut --force
```


Reboot by selecting "Logout" and "Reboot" and you should be back to the
Heads recovery shell with the `boot.hashes` error.  Insert your Yubikey and run
(hit tab to autocomplete the file names):


```
qubes-update /boot/xen-4.6.4-heads.gz /boot/vmlinux... /boot/initramfs...
```

This should prompt you for the TPM owner password to create the new
counter (only the first time), then for your GPG card's password.
It will output `/boot/boot.hashes` and `/boot/boot.hashes.asc`
with the signed hashes of the executables.

Lastly you'll need to seal the disk encryption keys with a disk unlock key
that you will enter everytime.  Run `seal-key` and it will prompt you for
the disk recovery key (the long one you entered above), a disk unlock
key that you will enter on every boot, and on your first run also the
TPM owner password to create the NVRAM space.


Installing extra software
---
dom0 probably has updates available. You'll want to install them before switching `/` to read-only and signing the hashes:

```
sudo qubes-dom0-update
```

You'll need the dm-verity tools to enable hashing

```
sudo qubes-dom0-update veritysetup
```

powertop is useful for debugging power drain issues. In dom0 run:

```
sudo qubes-dom0-update powertop
```

You might want to make the middle button into a scroll wheel. Add this to `/etc/X11/xorg.conf.d/20-thinkpad-scrollwheel.conf`

```
Section "InputClass"
	Identifier	"Trackpoint Wheel Emulation"
	MatchProduct	"TPPS/2 IBM TrackPoint|DualPoint Stick|Synaptics Inc. Composite TouchPad / TrackPoint|ThinkPad USB Keyboard with TrackPoint|USB Trackpoint pointing device|Composite TouchPad / TrackPoint"
	MatchDevicePath	"/dev/input/event*"
	Option		"EmulateWheel"		"true"
	Option		"EmulateWheelButton"	"2"
	Option		"Emulate3Buttons"	"false"
	Option		"XAxisMapping"		"6 7"
	Option		"YAxisMapping"		"4 5"
EndSection
```

You'll probably want to enable fan control, as described on [ThinkWiki](http://www.thinkwiki.org/wiki/Fan_control_scripts).

Disabling the ethernet might make sense to save power

Read-only root
---
There are some changes to Qubes' files that have to be made first. [Patches were posted to the qubes-devel list](https://groups.google.com/forum/?fromgroups#!topic/qubes-devel/hG93VcwWtRY), although they need to be updated.

TODO: write a script to apply all of these fixes

Hashing the / partition and setting up dm-verity
---
Signing /boot
---
TPM Disk encryption keys
---
The keys are currently derived only from the user passphrase, which is expanded via the LUKS expansion algorithm to increase the time to brute force it. For extra protection it is possible to store the keys in the TPM so that they will only be released if the PCRs match.

*This section is an early draft*

There are two tools in the Heads ROM image for working with the TPM keys. `seal-key` will generate a new key, seal it with the current PCRs and add a TPM passphrase, then store it into the TPM NVRAM. `unseal-key` will extract it from the NVRAM and request the user passphrase to decrypt/unseal it. If the PCRs do not match, the TPM will reject the attempt (and hopefully dump keys after too many tries?).

To setup the drive encryption, generate and seal a new key. Then unseal it to create `/tmp/secret.key` in the initial ramdisk. Delete the old keys from the root, home and swap partitions (can this use disk labels?):

```
cryptsetup luksKillSlot /dev/sda2 1
cryptsetup luksKillSlot /dev/sda3 1
cryptsetup luksKillSlot /dev/sda5 1
```

Then add the (now cleartext) key to each partition:

```
cryptsetup luksAddKey /dev/sda2 /tmp/secret.key
cryptsetup luksAddKey /dev/sda3 /tmp/secret.key
cryptsetup luksAddKey /dev/sda5 /tmp/secret.key
```

NOTE: should the new LUKS headers be measured and the key re-sealed with those parameters? This is what the Qubes AEM setup uses and is probably a good idea (although we've already attested to the state of the firmware).

This is where things get messy right now. The key file can not persist on disk anywhere, since it would allow an adversary to decrypt the drive. Instead it is necessary to unseal/decrypt the key from the TPM and then bundle the key file into a RAM copy of Qubes' dom0 initrd on each boot. The initramfs format allows concatenated cpio files, so it is easy for the Heads firmware to inject files into the Qubes startup script.

Hardware hardening
===
Soldering jumpers on WP# pins, setting BP bits, epoxy blobs.
