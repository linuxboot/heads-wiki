---
layout: default
title: Qubes
permalink: /OS/Qubes
nav_order: 1
parent: Operating Systems
grand_parent: Installing and configuring
has_toc: true
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

![Disk Recovery Key]({ site.baseurl }}/images/Disk_encryption_recovery_key.jpg)

The Disk Recovery Key that you enter here will be used as the
"recovery password" later.  It should be a long value since you won't
have to enter it very often; only when upgrading the Heads firmware
or if there is a need to recover the disk on an external machine.

DO NOT lose the Disk recovery key. This key passphrase will need to be [reentered](/Keys/#tpm-disk-encryption-key).

This option is offered from the GUI (again lets not forget that going into recovery invalidates PCR measurements, and that having kernel modules loaded mismatch between the moment of setting the TPM disk encryption key will not fly. This is why this should be done from the GUI by saving a new boot default option and answering Y to `Do you wish to add a disk encryption to the TPM [y/N]`:


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

