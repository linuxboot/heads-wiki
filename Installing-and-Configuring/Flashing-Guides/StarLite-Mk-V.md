---
layout: default
title: Star Labs StarLite Mk V
permalink: /StarLite-Mk-V-flashing/
published: false
nav_order: 6
parent: Step 2 - Flashing Guides
grand_parent: Installing and configuring
---

Star Labs StarLite Mk V
===

> **Port status:** The StarLite Mk V port is still
> [under review](https://github.com/linuxboot/heads/pull/2164). This page is draft
> documentation for port testers and is excluded from the published site. It
> can be published after the exact Heads revision has passed the porting
> checklist and the code PR has merged.

The StarLite Mk V uses a debug connector rather than a clip directly on the SPI
flash. Follow the
[Star Labs external programmer guide](https://support.starlabs.systems/hc/star-labs/articles/starlite-mk-v-installingrecovering-firmware-with-external-programmer)
for disassembly, battery disconnection and connection of the FPC cable, debug
board and CH341A programmer.

Read the [SPI Programmer Best Practices guide]({{ site.baseurl }}/SPI-Programmer-Best-Practices/)
before connecting the programmer.

## Back up the flash

With the StarLite battery and charger disconnected, take two complete reads:

```shell
sudo flashrom -p ch341a_spi -r starlite-backup-1.rom
sudo flashrom -p ch341a_spi -r starlite-backup-2.rom
sha256sum starlite-backup-1.rom starlite-backup-2.rom
cmp starlite-backup-1.rom starlite-backup-2.rom
```

Stop if `cmp` reports a difference. Keep both backups on another device. They
contain the original descriptor, Intel ME, EC, settings and per-device data
needed for recovery.

## Flash Heads

After the port has merged, build the `starlabs_lite_adl` target and verify the
ROM hash against the build output. Before merge, only port testers with recovery
hardware should use the exact head commit from the code PR and its corresponding
CI artifact. Set `HEADS_ROM` to the resulting
`heads-starlabs_lite_adl-*.rom` file.

The Heads ROM contains common build data outside the `COREBOOT` FMAP region.
Do not use the whole-image write command at the end of the general Star Labs
recovery guide with a Heads ROM. Preserve the existing descriptor, Intel ME,
EC, SMMSTORE and per-device regions by writing only `COREBOOT`:

```shell
HEADS_ROM=path/to/heads-starlabs_lite_adl-version.rom
sudo flashrom -p ch341a_spi --fmap -i COREBOOT \
  -N -w "$HEADS_ROM"
```

`-N` limits automatic verification to the included `COREBOOT` region; it does
not disable that verification. Do not use `-n`. If the write or verification
fails, leave the programmer connected and restore `starlite-backup-1.rom`
before attempting to boot.
