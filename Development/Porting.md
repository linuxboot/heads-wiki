---
layout: default
title: Porting Heads
permalink: /Porting/
nav_order: 3
parent: Development
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

Prerequisites for porting Heads
===
Prerequisites: 
1. coreboot port:  since Heads is a coreboot payload, the board must have fully completed and actively maintained coreboot support. Any known issues should be acceptable to end users. Exceptions include the Purism and Dasharo coreboot forks, where coreboot is used from their respective forks.
2. TPM module: Measured boot and the usable security features are at the core of what Heads is about. These features depend on the TPM. Therefore, the board must have a TPM module. If it uses fTPM and you plan to neuter the Intel Management Engine (ME), the fTPM must remain functional under coreboot even after neutering ME. Otherwise, a dTPM is required for this board.
3. The final flash layout must have enough space for the Heads payload. Concretely, the size of the flash chip(s) must be at least be 12MB and the size of the BIOS region must be large enough to take the Heads payload. Usually, this means shrinking another region, the ME, and reallocating the freed space to the BIOS region. However, changing the ME blob or flash layout is not always possible as Intel Bootguard prevents booting with such modifications in most modern boards unless the board vendor chose otherwise.
4. Technical skills: the person porting the board must have basic knowledge of coreboot, Linux, Bash, Git, and Python to complete the port. While the community is committed to helping, an alternative option is a financial contribution for [consultancy services](https://osresearch.net/Consultation-Services/).
5. External programmer: an external programmer is required to flash Heads and, if necessary, to recover from a brick.
6. Users and Board-testers: There should be multiple users and testers interested in using the board, as this makes maintenance and testing easier and benefits the entire ecosystem. You may, of course, port it for yourself first and allow others to join later. However, you must commit to testing—especially after coreboot version bumps or Linux kernel updates. If testing is not done in a timely manner, the board will be moved to an "unmaintained, untested" status. This would be unfortunate, potentially a waste of time, and disappointing for everyone involved.
7. GPU: dGPUs are problematic in Heads for various reasons. While successful ports on older machines with dGPUs existed (e.g. [t530 and w530](https://github.com/linuxboot/heads/commit/a854144e2dbf14700491e3f1cb6f88e12d22197b)), security may be affected, and there is currently no clear solution for that. In case you are curious, please read a longer discussion on the Matrix coreboot channel starting with this [post](https://matrix.to/#/!EhaGFZyYcbyhdSgStq:matrix.org/$4OuJ9tPr8a_U6pwWgmjQxL0hnmpBzTATAHVl9gMvnfQ?via=matrix.org&via=tchncs.de&via=dodoid.com). 

The scheme depicts a port cycle.
![Port cycle]({{ site.baseurl }}/images/Porting_cycle.png)

Files needed to be created/modified are depicted on the scheme.
![Files overview]({{ site.baseurl }}/images/Files_scheme.png)

Prerequisites for building the ROM
===
The goal is to build a ROM before the code review. `NEW_BOARD` refers to the name of your board (e.g., `t480`). It is recommended to clone the Heads GitHub repository and create a new branch: 
```shell
git clone https://github.com/linuxboot/heads.git
git checkout -b Poc_NEW_BOARD
```

modules/coreboot:
---
coreboot fork that will be used should be added under `heads/modules/coreboot`. This will be the version you reference from the board config files. You should aim for building the board with an existing coreboot version, usually the main version that most boards use. If you need a different coreboot version, consider patching an existing version without breaking any other boards depending on it, of course. However, if you need to add a new coreboot fork, it should be done here. This is not ideal and not recommended, as there are already 2 different coreboot forks. It often leads to maintenance and resources burden.

binary blobs:
---
Check the coreboot configuration for the ported board. This will indicate which binary blobs are required and where they are expected to be located. Heads expects architecture-/board-specific scripts under `blobs/*` which do the magic, called by the main makefile that handles everything in Heads. These scripts must create reproducible binary blobs when invoked. The blobs should be placed in the `blobs/NEW_BOARD/`. Make sure the coreboot config file references the correct path for each blob and the blobs are added to `.gitignore` so that they are not accidentally committed to git.
Generally,  three binary blobs are required: Management Engine (ME), Intel Flash Descriptor Region (IFD), and Gigabit Ethernet (GBE). The IFD and GBE can be extracted from a donor board using coreboot’s ifdtool. For more details, refer to the [upstream documentation](https://doc.coreboot.org/util/ifdtool/layout.html). On some boards it may be necessary to provide more blobs to coreboot, for instance an MRC blob for RAM initialization if coreboot cannot distribute the blob itself for legal reasons.

Please note, if the ME is neutered, the IFD, coreboot CBFS region, and ME neutering space should be adjusted accordingly. Rationale: the ME region defined under IFD must fit. With the IFD ME region reduced, the BIOS region can grow with the freed ME space. With the BIOS region augmented, the CBFS region of the coreboot configuration must be increased to fit the optimized space (all current Heads boards use this approach).

Please note the GBE MAC address should be forged to: `00:DE:AD:C0:FF:EE MAC`. It can be done with [nvmutil](https://libreboot.org/docs/install/nvmutil.html). Due to licensing restrictions, the ME firmware cannot be uploaded to the GitHub. However, scripts can be used to build it locally and within CircleCI (a gray area legally, but still possible). GBE and IFD blobs can be uploaded directly to GitHub. Please explain how they were obtained in the commit message.
* Note: When calling scripts in Nix-based environments, Python must be invoked explicitly, as Nix does not allow executing Python scripts directly from files. One can use last clean example for t480: `python ./finalimage.py` will work and just `./finalimage.py` will not work. 
The blobs folder should have a script.sh which handles downloading, deactivating ME etc. It should also contain a README.md file briefly explaining the process. Hashes of the blobs should be stored either in `README.md` or in `hashes.txt file`. Furthermore, the script must ensure the integrity of the blobs it produced by comparing a SHA256 hash.

NEW_BOARD.mk:
---
Create a new `heads/targets/NEW_BOARD.mk` file which deals with calling blobs/script.sh*, download and extraction, and placing the blobs in correct location. This will be used by the main makefile and defines the board specific targets and dependencies, such as the binary blobs. For additional details, please see [Makefiles]({{ site.baseurl }}/Development/make-details.md).

board.config: 
---
There should be a file `NEW_BOARD-hotp-maximized.config`, which inherits all the parameters from `NEW_BOARD-maximized.config` and adds  HOTP verification. Those files should be located in the folder `heads/boards/NEW_BOARD-hotp-maximized` and `heads/boards/NEW_BOARD-maximized`, respectively. This is Heads specific configuration and should be adapted from a similar platform. The top of the board config files is also a good place to put some technical board-specific documentation, known issues etc.
You should point in `NEW_BOARD-hotp-maximized` and `NEW_BOARD-maximized` to the coreboot version e.g. `export CONFIG_COREBOOT_VERSION=X.Y.Z.`- that was modified under `modules/coreboot` (see corresponding section above). Additionally, the configurations should reference the appropriate coreboot and linux configs created below:
```
CONFIG_COREBOOT_CONFIG=config/coreboot-NEW_BOARD.config
CONFIG_LINUX_CONFIG=config/linux-NEW_BOARD.config
```
* For early testing, enabling debug mode is a good idea. However, these options should be removed before merging:
```
export CONFIG_DEBUG_OUTPUT=y
export CONFIG_ENABLE_FUNCTION_TRACING_OUTPUT=y
```
Note: This may cause glitches where the screen output appears overwritten until an arrow key is pressed. This is normal for DEBUG mode, as the additional tracing output uses the same console as fbwhiptail. The issue will disappear once debug options are removed from the board configs.
* It is important to define the correct TPM configuration (TPM 1.2 vs. TPM 2.0), ensuring they are mutually exclusive, similar to the coreboot configuration (described in coreboot config below). The selected option misses `#`:
```
#TPM2 requirements
export CONFIG_TPM2_TOOLS=y
export CONFIG_PRIMARY_KEY_TYPE=ecc
```
```
#TPM1 requirements
#export CONFIG_TPM=y
```
In the configuration, the binary blobs must be specified as build targets, as coreboot depends on these blobs. It is the line at the bottom of the board config `BOARD_TARGETS := NEW_BOARD`. It ensures that the main Heads makefile builds the targets specified in the `NEW_BOARD.mk` file.

coreboot.config:
---
You need a coreboot configuration for the new board. Ideally, this config should be adapted from the closest possible platform. You can inspect existing configurations for boards in the master branch and select one with a similar architecture, if available. Next, you can create a coreboot config by copying it from the closest platform and adapt it accordingly. You may use coreboot's `make menuconfig` for `NEW_BOARD`:
```shell
cp config/coreboot-CLOSEST_PLATFORM.config config/coreboot-NEW_BOARD.config
./docker_repro.sh make coreboot.modify_and_save_oldconfig_in_place
```
Note, the configuration needs to define the correct path references to all binary blobs, that are not provided by coreboot, on most architectures this includes at least `IFD`, `ME` and `GBE` (see above). The configuration file path should be: `heads/config/coreboot-NEW_BOARD.config`.
* Note:
TPM measured boot (```CONFIG_TPM_MEASURED_BOOT=y```) and verified boot (```CONFIG_VBOOT_LIB=y```) should be enabled. Ensure that you select CONFIG_TPM_MEASURED_BOOT and CONFIG_VBOOT_LIB in Kconfig. Furthermore, enable the TPM and pick the correct TPM version for the board. 
Other parameters depend on the board. It is up to you to determine the correct settings, as not all community members will have access to your board. `git diff` between `coreboot-CLOSEST_PLATFORM.config` and `coreboot-NEW_BOARD.config` may help.

linux.config:
---
This should be adapted from a similar board. The file path should be `heads/config/linux-NEW_BOARD.config`.

CircleCI: 
---
Modify `heads/.circleci/config.yml` to add support for the `NEW_BOARD`. Initially, configure it to depend directly on musl-cross-make. At this stage, do not reuse caches—this simplifies debugging and ensures a clean build process. 
```
# NEW_BOARD is based on `X.Y.Z`. coreboot release, not sharing any buildstack from now, depend on muscl-cross cache
      - build:
          name: NEW_BOARD-hotp-maximized
          target: NEW_BOARD-hotp-maximized
          subcommand: ""
          requires:
            - x86-musl-cross-make
```
CircleCI creates reproducible builds, allowing users to verify that the "flashable" roms were indeed produced by the given source code. In the development/debugging phase, it helps to ensure that you talk about the same build. It optimizes the collaboration between peers-board owners. Moreover, CI builds are significantly faster than local builds, reducing overall development time. As circleCI always builds all boards it makes it easier to find regressions early.

* Optional: local builds.
If you need to build locally (e.g. to ensure that some features work) pay attention to the helper functions at the end of the Makefiles. It is strongly recommended that local builders review the end of the Makefiles (including modules/*files), as these helper functions were designed to facilitate coreboot and Linux version bumps.
For a completely clean build (the most radical approach), either clone the Heads repository again or remove all build artifacts using:
```bash
./docker_repro.sh BOARD=NEW_BOARD real.clean
```
Building all tools needed will take a while depending on the number of cores and RAM available on your machine. For a less radical approach, run:
```shell
./docker_repro.sh BOARD=NEW_BOARD real.remove_canary_files-extract_patch_rebuild_what_changed
```
Then, inject any changes into the coreboot fork canary file so that the build system refetches, repatches, and rebuilds the coreboot fork:
```shell
echo "bogus" | sudo tee build/x86/coreboot-t480/.canary
```
Note, patches that attempt to create files that are not expected to exist but exist will fail, showing at console what files already existed that couldn't be created. In this case, you need to remove them manually `rm -rf`, and restart the build with `./docker_repro.sh BOARD=NEW_BOARD` which will progress until each modules/* required per board config is successfully built.
	
Testing
===
For thorough testing—especially for a new board—using the following template (tasks to check) may be beneficial. Copy and paste this template into the GitHub message window when reporting test results. Mark completed tasks with an `x` inside the brackets [ ]. Replace `X.Y.Z` with the relevant information. It is recommended also to upload relevant screenshots For additional safety, you may wish to remove the metadata from these files.
```
- [ ] Successful external flash link to circleci: https://X.Y.Z. from commit `X.Y.Z.` using external programmer model `X.Y.Z.` on `X.Y.Z.` Voltage mode
- [ ] Boots successfully after the flashing: 
- [ ] Setting clock prompt on first reboot: ok if triggered correctly after initial flashing and CMOS battery disconnected
- [ ] Clean boot detected (no keyring, nothing installed on disk): usb boot proposed and followed
- [ ] Boots on usb
- [ ] OS `X.Y.Z.` install and reboot
- [ ] Heads functionality- no pubkey detected, but OS detected -> OEM-Factory-reset proposed. Done with `X.Y.Z.` hardwarekey e.g. nk3
- [ ] On reboot after re-ownership: generate new HOTP/TOTP
- [ ] wifi works based on OS `X.Y.Z.`
- [ ] PR0
    - [ ] flashprog -p internal (not locked)
    - [ ] lock_chip (locks the platform with PR0, if PR0 patch applied in fork or under `patches/coreboot-X.Y.Z.` and coreboot config contain proper preparation of the platform)
    - [ ] flashprog -p internal (reports locked)
```

Debugging
===
Debugging needs to be activated first.
The relevant config changes will be added under CBFS either `config` at build time or `config.user` override through Heads GUI and flashed to SPI flash to be useable on next reboot.

You can then either flash back a ROM without debugging, or turn off debugging (informational output or quiet mode, see GUI pictures below)

Enabling DEBUG + TRACE in board config prior of build
---

Debugging can either be enabled from the board config adding the following (see qemu board configs which have debugging) prior of building:
```
#Enable DEBUG output
export CONFIG_DEBUG_OUTPUT=y
export CONFIG_ENABLE_FUNCTION_TRACING_OUTPUT=y
```

Enabling DEBUG + TRACE from Heads GUI
---

<img width="1162" height="960" alt="Screenshot_20260130_122935" src="https://github.com/user-attachments/assets/1ce6f156-036a-4d65-9fd1-a90002c13ab3" />
<img width="1162" height="960" alt="Screenshot_20260130_123014" src="https://github.com/user-attachments/assets/e8bab3fb-88ad-4d16-b5b9-40e4c1963857" />
<img width="1162" height="960" alt="Screenshot_20260130_123034" src="https://github.com/user-attachments/assets/7f15002e-0c2f-4205-8f90-f63e1d42578f" />
<img width="1162" height="960" alt="Screenshot_20260130_123047" src="https://github.com/user-attachments/assets/ce2c4be0-5c9a-48bf-bd1f-380e73551183" />
<img width="1162" height="960" alt="Screenshot_20260130_123110" src="https://github.com/user-attachments/assets/0bd62a97-3115-4d6a-86cc-6709a0403de6" />

The relevant config changes will be added under CBFS `config.user` and flash those changes to SPI flash to be useable on next reboot.

Obtaining logs
---

Once debugging is active, generate error or state prior of error and enter Recovery shell console. 
Plug in a formatted USB thumb drive (fat32/exfat/ext3/ext4) and then type 
```shell
mount-usb --mode rw
cp /tmp/debug.log /media
cbmem -L > /media/coreboot_measured_boot.txt
cbmem -1 > /media/coreboot_cbmem_console.txt
cbmem -t > /media/coreboot_cbmem_stages_timestamps.txt
lsusb > /media/lsusb.txt
lspci -vv > /media/lspci.txt
[... any other relevant information redirected to a file under /media where usb thub drive mounted...]
umount /media
```
And then provide log files in a subsequent comment. That would be really helpful if something is still wrong. Of course, this should be modified based on the problem faced.

Pull Request (PR)
===
Create an initial PR, with commits relative roughly to the steps above. 1 commit should correspond to 1 major bullet point. Note that Heads is all about reproducibility. Therefore, your commit messages should explain why you made the change and _how_ you got there. For instance, explain how to create a certain binary blob or better write a script that can be run by everyone.

Then lets collaborate in the PR. Create a heads fork in your own repository, follow the [contribution guide](https://github.com/linuxboot/heads/blob/master/CONTRIBUTING.md) as you go, make sure you sign and sign-off commits and don't stress too much if this seems a lot. Get a PR started and we will collaborate there. 

Remove debugging mode:
===
One of last commits should remove the debugging mode, functions tracing and use quiet mode instead.

Eg (DEBUG + TRACING commented out or `=n`)
```
#Enable DEBUG output
#export CONFIG_DEBUG_OUTPUT=y
#export CONFIG_ENABLE_FUNCTION_TRACING_OUTPUT=y
export CONFIG_QUIET_MODE=y
```

Write a flashing/disassembling guide
===
Take pictures as you disassemble your board and flash the Heads ROM. You can use them later to write a guide. This will help less technical community members using the board. Your effort will improve the ecosystem. 
If you cannot manage to finish writing the guide but talented enough to finish the port just create an issue on the GitHub and drop pictures. We will try to find time to help.
The guide is essential part of the port.

Contribute to maintenance
===
Refer to [Contributing on GitHub page](https://github.com/linuxboot/heads/blob/master/CONTRIBUTING.md)

Heads is free as free beer.
In the spirit of open-source software, free knowledge, and communal goodwill, support the Heads and the open source development [Insurgo Initiative - Open Collective](https://opencollective.com/insurgo).
The supply of free beer cannot be infinite. If everyone takes without giving back, at some point, the keg runs dry, and so does the goodwill.
If you enjoy free beer, contribute in some form—whether by bringing code, docs, or financial support. The cycle must be maintained to keep the ecosystem alive. No one wants to be the last guest realizing the fridge is empty and no one restocked it. Together, it is possible to keep the free beer spirit alive—open, shared, and always refreshing. Cheers!

You may examine all [github PRs with the label 'port'](https://github.com/linuxboot/heads/issues?q=label%3Aport+) to gather additional information. Good luck! 
