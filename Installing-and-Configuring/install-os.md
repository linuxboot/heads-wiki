---
layout: default
title: Step 4 - Installing Qubes and other OSes
permalink: /InstallingOS/
nav_order: 8
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

Generic OS Installation
===

1. Insert OS installation media into one of the USB3 ports (blue on Thinkpads).
[For certian OSes](https://github.com/osresearch/heads/tree/master/initrd/etc/distro/keys)
 , the Heads boot process supports standard OS bootable media (where the USB
 drive contains the installation media which as created using `dd` or
 `unetbootin` etc.) as well as booting directly from verified ISOs on a plain
 old partition.  For example, if the USB drive has a single partition, you can
 put the ISO image along with a trusted signature in the root directory:

```shell
/Qubes-R4.0-x86_64.iso
/Qubes-R4.0-x86_64.iso.asc
/Fedora-Workstation-Live-x86_64-27-1.6.iso
/Fedora-Workstation-Live-x86_64-27-1.6.iso.sig
/tails-amd64-3.7.iso
/tails-amd64-3.7.iso.sig
```

Each ISO is verified before booting so that you can be sure Live distros and
 installation media are not tampered with, so this route is preferred when
 available.  You can also sign the ISO with your own key:

```shell
gpg --output <iso_name>.sig --detach-sig <iso_name>
```

Some distros require additional options to boot properly directly from ISO.  See
 [Boot config files](/BootOptions) for more information.
2. Boot from USB by either running `usb-scan` or reboot into USB boot mode (hit
 'u' before the normal boot)
3. Select the install boot option for your distro of choice and work through the
 standard OS installation procedures (including setting up LUKS disk encryption
 if desired)
4. Reboot and your new boot options should be available to be chosen by
 selecting 'm' at the boot screen

If you want to set a default option so that you don't have to choose at every
 boot, you can do so from the menu by selecting 'd' on the confirmation screen.
 You will also be able to seal your Disk Unlock Key using the TPM allowing
 you to use ensure only a boot passphrase and the proper PCR state can unlock this
 yet.

(\*) Ubuntu/Debian Note: These systems don't read `/etc/crypttab` in their
 initrd, so you need to adjust the crypttab in the OS and `update-initramfs -u`
 to have it attempt to use the injected key.  Due to oddities in the cryptroot
 hooks, you also need keyscript to be in `/etc/crypttab` even as a no-op
 `/bin/cat`:

`sda5_crypt UUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX /secret.key luks,keyscript=/bin/cat`

(Credit to [https://www.pavelkogan.com/2015/01/25/linux-mint-encryption/](https://www.pavelkogan.com/2015/01/25/linux-mint-encryption/)
 for this trick).

Installing Qubes
===

![Heads splash screen]({{ site.baseurl }}/images/Heads_splash_screen.jpg)

Plus in the USB stick with the R4.0 install media into one of the USB port and
 boot into USB mode (hit 'u' at the prompt), then boot using this option:

```text
2. Install Qubes R4.0 [kernel /isolinux/xen.gz console=none]
```

If that completes with no errors it will launch the Xen hypervisor from the
 ROM image and start the Qubes installer.  The first few seconds are run
 with an archaic video mode, so things appear a little weird, but once the dom0
 kernel initializes the graphics it should look right.

![Qubes partitioning]({{ site.baseurl }}/images/Qubes_partitioning.jpg)

Use default QubesOS partitioning scheme for QubesOS 4.x

![Disk Recovery Key]({{ site.baseurl }}/images/Disk_encryption_recovery_key.jpg)

The Disk Recovery Key that you enter here will be used as the
"recovery password" later.  It should be a long value since you won't
have to enter it very often; only when upgrading the Heads firmware
or if there is a need to recover the disk on an external machine.
You will need it again shortly, so don't lose it yet.

![Signing Qubes binaries in /boot]({{ site.baseurl }}/images/Signing_Qubes_binaries_in__boot.jpg)

Once Qubes has finished installing, you'll need to reboot and select the 'Boot
 menu' option by hitting 'm'.

Select the first boot option:

```text
1. Qubes, with Xen hypervisor [...]
```

Then make this the default boot entry by hitting 'd'.  This will also allow you
 to seal the Disk Unlock Key if the device supports it.

You will need to input the Disk Recovery Key here (almost for the last time),
 and this should start the final stage of the Qubes installer.  Under
 `Configure Qubes` you should select `Create USB qube holding all USB controllers`
 so that they are protected from outside devices.  This step takes a little
 while as the templates are configured...

Eventually this will be done and you can click "Finish", then Qubes will
give you a login screen with your login password.

If you choose to add the Disk Unlock Key to the TPM, you'll need to specify
 which LUKS volume.  A default Qubes install will work if you leave the
 'Encrypted LVM group?' response blank and enter `/dev/sda2` when asked about
 'Encrypted devices?'.  For more details see the TPM Disk Unlock Keys
 section below. You'll then be asked to enter the Disk Recovery Key as well as
 the new boot passphrase you'll use to unseal that key.

To start Heads now (and in the future), just hit 'y' for default boot.

This should start the final stage of the Qubes installer.  Under
'Configure Qubes' you should select `Create USB qube holding all USB controllers`
 so that they are protected from outside devices.  This step takes a little
 while as the templates are configured...

Eventually this will be done and you can click "Finish", then Qubes will
give you a login screen with your login password.

After the first reboot, the boot entry will be different post-installation, so
 after you hit 'y' to select default boot you will see a message:

```text
!!! Boot entry has changed - please set a new default
```

This will also happen on OS updates that changed the boot process (updating the
   kernel or the initramfs, etc.).  If someone has tampered with your `/boot`
   partition, this can also happen, so if you're not sure of the situation,
   don't proceed.

Choose the first option again ('1'), then make it the new default ('d'), confirm
 that you're modifying the boot partition ('y'), and that you don't need to
 reseal the disk key ('n').  You'll be asked to insert your USB Security dongle
 and enter the PIN to sign the new configs and the system will reboot and allow
 you to proceed as normal.

Installing extra software
---

```shell
sudo qubes-dom0-update
```

powertop is useful for debugging power drain issues. In dom0 run:

```shell
sudo qubes-dom0-update powertop
```

You might want to make the middle button into a scroll wheel. Add this to
 `/etc/X11/xorg.conf.d/20-thinkpad-scrollwheel.conf`

 <!-- markdownlint-disable MD013 -->

```text
Section "InputClass"
  Identifier  "Trackpoint Wheel Emulation"
  MatchProduct  "TPPS/2 IBM TrackPoint|DualPoint Stick|Synaptics Inc. Composite TouchPad / TrackPoint|ThinkPad USB Keyboard with TrackPoint|USB Trackpoint pointing device|Composite TouchPad / TrackPoint"
  MatchDevicePath  "/dev/input/event*"
  Option    "EmulateWheel"    "true"
  Option    "EmulateWheelButton"  "2"
  Option    "Emulate3Buttons"  "false"
  Option    "XAxisMapping"    "6 7"
  Option    "YAxisMapping"    "4 5"
EndSection
```

<!-- markdownlint-enable MD013 -->

You'll probably want to enable fan control, as described on [ThinkWiki](http://www.thinkwiki.org/wiki/Fan_control_scripts).

Disabling the ethernet might make sense to save power

