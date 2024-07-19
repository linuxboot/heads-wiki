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

Insert OS installation media into one of the USB3 ports (blue on Thinkpads).
 [For certain OSes](https://github.com/osresearch/heads/tree/master/initrd/etc/distro/keys),
 Heads boot process supports standard OS ISO bootable media (where the USB
 drive contains the ISO installation media alongside of its detached signature).
 For other OS, you will need to create USB installation media with using `dd` or
 `unetbootin` etc.).  

For supported OSes, on a EXT3/EXT4/ExFat formatted partition on USB drive, you can
 put the ISO image along with a trusted detached signature in the root directory:
```shell
/Qubes-R4.0-x86_64.iso
/Qubes-R4.0-x86_64.iso.asc
/tails-amd64-3.7.iso
/tails-amd64-3.7.iso.sig
```

- Some distros will require additional options to boot directly from ISO.  See [Boot config files](/BootOptions) for more information.
- Boot from USB by Boot menu options, or by calling `usb-scan` from the recovery shell.
  - Select the install boot option for your distro of choice and work through the standard OS installation procedures (including setting up LUKS disk encryption if desired)
- Reboot and your new boot option should be available through boot options: show boot options.

Each ISO file is verified for integrity and authenticity before booting so that you can be sure Live distros and
 installation media are not tampered with or corrupted, so this route is preferred when
 available.  You can also sign the ISO with your own key **from Heads Recovery Shell 
 menu option (Options-> Exit to recovery shell)** :

```shell
mount-usb --mode rw #Loads USB controller kernel modules, scan for partitions, ask which one to mount under /media
cd /media # Change directory to /media to do the detach-sign operation
gpg --detach-sign <iso_name> # You can use TAB keyboard's key for autocompletion of file names here. This requires a provisioned USB security dongle: Factory Reset/Re-Ownership
reboot # Will remount everything in Read Only and sync changes to block devices automatically.
```
Then boot from detached signed ISO after having rebooted your machine.
 To do so, from main menu, select: Options-> Boot Options-> USB boot
 Select your ISO from list, see detached signature validation succeeding, then select GRUB parsed options with label and main boot parameters from the options provided to you.

Compatibility
===

Heads requires unencrypted `/boot`.  Graphical OSes generally have the best support.  Debian live installers,
 Fedora Workstation (or spins), Qubes, and PureOS all work well.

* *For Debian*:
    1. Use a live desktop image.  The network installer image does not work on all systems.
    2. Ensure `/boot` is unencrypted.  Debian 11 defaults to a single encrypted partition, so you must
       partition manually.  This may not apply to all Debian derivatives.
        * Create one 1G ext4 partition mounted at `/boot`
        * Create a LUKS container with one ext4 partition mounted at `/`.
        * For swap, you can create a swapfile later on the encrypted root, or create a swap partition.
* *For Fedora*: The default partitioning works, but `/` is btrfs by default, which Heads' recovery console does
 not support.  Use ext4 instead for recovery console support.
* *For Qubes*: Be sure to disconnect USB tokens during configuration on first boot.  Otherwise, the Qubes
 installer may prevent the creation of a sys-usb qube if they are detected as keyboards (HID devices).  If you
 are using a USB keyboard, follow the
 [Qubes instructions for USB keyboards](https://www.qubes-os.org/doc/usb-qubes/#usb-keyboards).
* *For PureOS*: The default installation works.

Default Boot and Disk Unlock
===

If you want to set a default option so that you don't have to choose at every
 boot, you can do so from the menu by selecting 'd' on the confirmation screen.
 You will also be able to seal your Disk Unlock Key into the TPM, which would be unsealed only when provided with the good TPM disk encryption key passphrase and when firmware measurement and LUKS header are the same as when the Disk Unlock Key was sealed when booting from detached signed default boot option selection.

This should work for Qubes OS, Fedora, Debian (and derivatives).


Installing Qubes 4.X
===
Qubes OS and Tails can boot directly from ISO when provided with accompanying detached signatures (iso.asc or iso.sig), thanks to 
distribution signing keys being provided under Heads, permitting to validate both integrity and authenticity of the ISOs prior of booting into them.

Plug in the EXT3/EXT4/ExFat formatted USB stick containing Qubes iso and iso.asc files into one of the USB port and
 boot it from USB mode:

![1-Heads-Options](https://user-images.githubusercontent.com/827570/156627927-7239a936-e7b1-4ffb-9329-1c422dc70266.jpeg)
![2-Heads-Boot-Options](https://user-images.githubusercontent.com/827570/156627934-8051cd38-ad5e-452d-b340-9d13317f33b8.jpeg)
![3-Heads-USB-Boot-Option](https://user-images.githubusercontent.com/827570/156627936-8fd43c94-fd2f-448b-a84c-eadd9166434e.jpeg)
![4-Heads-USB-Boot-Options-ISOs](https://user-images.githubusercontent.com/827570/156627940-c726b6e0-b74a-4b46-ae7b-b0727cdff05b.jpeg)
![5-Heads-USB-Boot-ISO-verification-Selection-of-ISO-boot-option](https://user-images.githubusercontent.com/827570/156627944-1cc0ad56-d0b2-4552-8ee7-a159871038f7.jpeg)
![6-Heads-USB-Boot-ISO-verification-Selection-of-ISO-boot-kexec](https://user-images.githubusercontent.com/827570/156627950-6ec7e3e9-a13e-4c2c-920c-c61fbb74af1f.jpeg)


If that completes with no errors it will launch the Xen hypervisor, kernel and initrd provided from ISO and start the Qubes installer:
![7-Q41-first-screen](https://user-images.githubusercontent.com/827570/156627951-bc29e472-db90-4a01-a870-0ab2ffa70c2c.jpeg)
![8-Q41-Select-Installation-destination](https://user-images.githubusercontent.com/827570/156627954-e5681533-80cd-41b8-9b47-4ecd2cb7d132.jpeg)

Use default QubesOS partitioning scheme for QubesOS 4.x:
![9-Q41-Destination-automatic-partitioning-with-encryption](https://user-images.githubusercontent.com/827570/156627956-1dc8e56e-5a3b-403c-bf67-5235b5150ce7.jpeg)
![10-Q41-Destination-automatic-partitioning-with-encryption-done](https://user-images.githubusercontent.com/827570/156627958-b6d9e8b9-0c7b-4469-9a3b-c512b2982d41.jpeg)


The Disk Recovery Key that you enter here will be used as the
"recovery password" later.  It should be a long value since you won't
have to enter it very often; only when upgrading the Heads firmware, 
when setting a new boot default and desiring to change TPM released disk encryption 
key (Disk Unlock Key), or if there is a need to recover the disk on an external machine.
![11-Q41-Destination-automatic-partitioning-with-encryption-disk-reovery-passphrase-prompt](https://user-images.githubusercontent.com/827570/156627961-ff29a794-459e-4470-8f51-7574736d0c65.jpeg)
![12-Q41-Destination-automatic-partitioning-with-encryption-disk-reovery-passphrase-confirmation](https://user-images.githubusercontent.com/827570/156627972-64163ad3-7093-433c-bf1d-116ccbb7a3ae.jpeg)
![13-Q41-Destination-automatic-partitioning-addtitional-step-on-existing-install_reclaim](https://user-images.githubusercontent.com/827570/156627973-0b5df668-6170-403b-baf9-149499ce51d0.jpeg)
![14-Q41-Destination-automatic-partitioning-addtitional-step-on-existing-install_reclaim_delete_all](https://user-images.githubusercontent.com/827570/156627977-20ea5e55-3ff4-4473-bfeb-b71088097274.jpeg)
![15-Q41-Destination-automatic-partitioning-addtitional-step-on-existing-install_reclaim_delete_all-reclaim](https://user-images.githubusercontent.com/827570/156627978-760371ce-d0ab-4915-9601-6c0436f9e845.jpeg)
![16-Q41-user-creation](https://user-images.githubusercontent.com/827570/156627980-32ca88c1-7ea3-43d9-b7e7-f2a569f2b202.jpeg)
![17-Q41-user-creation-passphrase](https://user-images.githubusercontent.com/827570/156627984-0a833010-f11f-4ff5-823d-6e6a4e5a36e0.jpeg)
![18-Q41-Begin-installation](https://user-images.githubusercontent.com/827570/156627987-d2da1179-bc8c-4bb0-b471-05eba03efaae.jpeg)
![19-Q41-package_installation](https://user-images.githubusercontent.com/827570/156627989-90ac14aa-0374-418c-89ac-4e21750ff659.jpeg)
![20-Q41-package_installation2](https://user-images.githubusercontent.com/827570/156627992-54cdcac2-da5f-4e34-9a89-b7fd4aa53667.jpeg)
![21-Q41_installation_complete-reboot_system](https://user-images.githubusercontent.com/827570/156627998-c698ddc6-f565-4332-b31e-d87effb25035.jpeg)
First stage install is finished. 

Disconnect your USB Security dongle (and any external keyboard/mouses) prior of going further. Otherwise Qubes might detect those as USB Keyboards (HID devices) and will prevent sys-usb from being created properly:
![22-Heads_Options_to_boot_options](https://user-images.githubusercontent.com/827570/156628000-b48415ce-f525-4140-94b2-63c31c044dc0.jpeg)
![23-Heads_Boot_options_to_unsafe_boot](https://user-images.githubusercontent.com/827570/156628003-6d086bd9-f96d-436d-a28d-0d71b469c75e.jpeg)
![24-Heads_unsafe_boot](https://user-images.githubusercontent.com/827570/156628007-d6b8b1e1-9305-4c54-b444-519d8a77f893.jpeg)
![25-Heads_unsafe_boot_confirmation](https://user-images.githubusercontent.com/827570/156628011-000e6ca6-b3a8-4ad5-856e-07fe257fa807.jpeg)
![26-Heads_unsafe_boot_confirmation_select_dynamic_option](https://user-images.githubusercontent.com/827570/156628014-539a5819-bb61-48ca-b1f3-375fe7de3f21.jpeg)
![27-Q41_disk_recovery_key_passphrase-prompt](https://user-images.githubusercontent.com/827570/156628016-b8840c69-cb68-4419-88cf-f9b1a8b29474.jpeg)
![28-Q41_second_stage_install_main](https://user-images.githubusercontent.com/827570/156628018-889d8b46-4303-4ff0-a4cd-a2600042af83.jpeg)
![29-Q41_options_selection_done](https://user-images.githubusercontent.com/827570/156628021-d3ec34af-e620-48b8-aa39-45d84c968769.jpeg)
![30-Q41_options_selection_done_finish](https://user-images.githubusercontent.com/827570/156628023-2c49d8ac-3394-4c90-bab8-ae02afa0bf5e.jpeg)

You should now have Qubes 4.1 installed!


Taking ownership of the states
===

 
Taking ownership of the TPM
====
Heads keeps TPM and HOTP rollback counters under /boot. Since we just installed, those doesn't exist and we need to create them. 
First things first, we need to acknowledge current firmware state for the newly installed OS.

![Heads-Options_After-Install](https://user-images.githubusercontent.com/827570/156663999-c0b30f06-10c6-4f84-aedc-cc979826a3d2.jpeg)
![Heads-TPM_TOTP_HOTP](https://user-images.githubusercontent.com/827570/156664003-a912e566-378b-41ab-b364-6ba9cd289888.jpeg)
![Heads-TPM_reset_option](https://user-images.githubusercontent.com/827570/156664015-caecc4a4-6400-45c9-8841-110b251fb2b5.jpeg)
![Heads-TPM_reset_option_confirmation](https://user-images.githubusercontent.com/827570/156664020-7c54a240-ef77-40de-b7d7-834574bd361c.jpeg)
![Heads-TPM_reset_seals_TOTP_And_HOTP](https://user-images.githubusercontent.com/827570/156664023-adf85064-3556-4758-8d50-b9e2e93370e7.jpeg)
That's it. You now have TOTP scanned over your preferred TOTP smartphone app, or have entered manually the challenge secret under your favorite external TOTP app on another computer because you do not own a smartphone.

Signing /boot content
====

Now that firmware state is sealed under TPM and remotely attested through TOTP/HOTP, now is the time to sign /boot content until your next dom0 upgrade, which will most probably update Xen, initrd and kernel binaries, as well as grub configuration. This will be prompted automatically when selecting default boot option, since we have no digests nor detached signature of /boot content as of now.

**NOTE** : It is advisable to remove all USB security dongles (yubikeys and others) and only plug in the corresponding one when you want to update kernel settings to avoid some issues.


![Heads_default_boot_after_sealing](https://user-images.githubusercontent.com/827570/156664026-f6b03eaf-3f38-4b14-8ecc-db6f7078e209.jpeg)
![Heads_warns_about_no_hashes](https://user-images.githubusercontent.com/827570/156664029-ba065887-edb7-4111-881f-597ec6a1a33d.jpeg)
![Heads_warns_about_no_default_after_signing](https://user-images.githubusercontent.com/827570/156664031-756d8f31-6ed5-4e26-aa75-8da969a05fa5.jpeg)


Setting a new boot default
====

If you choose to add the Disk Unlock Key to the TPM, you'll need to specify
 which LUKS volume.  A default Qubes install will work if you leave the
 'Encrypted LVM group?' response blank and enter `/dev/sda2` when asked about
 'Encrypted devices?'.  For more details see the TPM Disk Unlock Keys
 section below. You'll then be asked to enter the Disk Recovery Key as well as
 the new boot passphrase you'll use to unseal that key.
 
![Heads_prompts_to_set_default_boot_option](https://user-images.githubusercontent.com/827570/156664033-362c0853-75f8-4c60-bd8f-3d5cbf4546c7.jpeg)
![Heads_prompts_to_set_default_boot_option_confirmation](https://user-images.githubusercontent.com/827570/156664039-980ea2c8-0400-492a-a1cb-563ad5034824.jpeg)
![Heads_prompts_to_set_default_boot_option_setting_disk_unlock_key](https://user-images.githubusercontent.com/827570/156664043-50fcbb40-b748-49fc-881b-4ae9a02b1f13.jpeg)

Until next dom0 upgrade, this is the normal boot process
====
![Heads_HOTP_dongle_flashes_green](https://user-images.githubusercontent.com/827570/156664048-b1527c60-91d8-46c1-af3e-d70974fa23a7.jpeg)
![Heads_HOTP_Success_main_screen](https://user-images.githubusercontent.com/827570/156664050-00e623ab-f942-4477-aace-47813b2978d4.jpeg)
![Heads_default_boot_DiskUnlock_key_prompt_until_next_dom0_upgrade](https://user-images.githubusercontent.com/827570/156664055-e0adfa8d-e2d4-4f3f-af78-5fba97f8355b.jpeg)

When updating dom0 from Qubes OS update widget
====
You need to reboot directly after applying dom0 upgrades.
You should follow this Qube's forum [Verifying Installation](https://forum.qubes-os.org/t/verifying-installation/11739) post to investigate integrity of dom0 and /boot components.

You will then get a similar prompt when selecting the Default boot option:
![signal-2022-11-09-141307](https://user-images.githubusercontent.com/827570/200931081-c2c6ff23-2b5f-431c-89c8-f33428dbf0cf.jpeg)

This is the result of Heads having verified that /boot/kexec.sig detached signature file of kexec_*.txt digests are still valid, but that the content of kexec_hashes.txt differs for some of the files measured under that digest. Heads will require you to generetate new digests and sign them.


Then, Heads will ask you to define a new boot default if grub.cfg file has changed:

```text
!!! Boot entry has changed - please set a new default
```

Applying dom0 (or OS updates) that changed the boot related binaries and config files (updating the
   kernel, Xen, or the initramfs, etc) will modify /boot content.  If someone has tampered with your `/boot`
   partition, this can also happen, so if you're not sure of the situation,
   don't proceed and investigate. The onlyway to make sure you are the origin of the changes is to reboot and sign /boot content right after the upgrade. On Qubes OS, that should only happen when upgrading dom0. For Other OSes, that can happen in any unattended upgrades, which requires you to inspect system upgrade logs or be aware of updates propositions: if a kernel update is involved, you sure need to reboot and sign now.

Choose the first option again ('1'), then make it the new default ('d'), confirm
 that you're modifying the boot partition ('y'), and that you don't need to
 reseal the disk key ('n').  You'll be asked to insert your USB Security dongle
 and enter the GPG User PIN to sign the new configs and the system will reboot and allow
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

