---
layout: default
title: Binary Blobs
permalink: /Binary-Blobs/
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

Coreboot specs
===

Intel
====

- xxx0: [gm45 bridge, Montevina: no FSP, no ME: X200, T400, T500, R500, X300](https://doc.coreboot.org/mainboard/lenovo/montevina_series.html) : **no QubesOS support there** (no proper vt-d2)
- [xx20](https://doc.coreboot.org/mainboard/lenovo/x2xx_series.html): [Sandy bridge, no FSP. ME<10: BUP module required only: X220/T420/T520](https://doc.coreboot.org/mainboard/lenovo/Sandy_Bridge_series.html)
- xx30: [Ivy bridge, no FSP. ME<10: ROMP and BUP required: X230/T430/W530](https://doc.coreboot.org/mainboard/lenovo/Ivy_Bridge_series.html) Z220 CMT and others

Additional required Intel blobs:
=====

- [FSP is present in all Broadwell+ platforms](https://doc.coreboot.org/soc/intel/fsp/index.html)

ME status on different boards models
=====

- [Removed in ME <=6 (xxx0)](https://libreboot.org/faq.html#intelme)
- [Deactivated+Neutered ME in ME 6 <= 10 (xx20 BUP/xx30 BUP+ROMP)](https://github.com/corna/me_cleaner/wiki/How-does-it-work%3F#me-versions-from-60-nehalem-to-10x-broadwell-1)
- [Deactivate+Partially Neutered (BUP, RBE, Kernel and syslibs modules **REQUIRED** in ME > 11)](https://github.com/corna/me_cleaner/wiki/How-does-it-work%3F#me-versions-from-11x-skylake-1)
- [Soft disable/HAP disable bit possible on ME 12+ (**PoC BE CAUTIOUS**)](https://github.com/corna/me_cleaner/pull/384)
- [xx30, xx20](https://github.com/corna/me_cleaner/wiki/How-does-it-work%3F#me-versions-from-60-nehalem-to-10x-broadwell): ME 6 <= 10
- [Skylake, Kabylake, Whiskeylake and newer](https://github.com/corna/me_cleaner/wiki/How-does-it-work%3F#me-versions-from-11x-skylake): ME >= 11
- Intel ME then changed its name to Converged Security Management Engine (CSME), where HAP bit can be flipped, but modules cannot be removed anymore.

AMD
====

- [AMD fam15h](https://doc.coreboot.org/soc/amd/family15h.html?highlight=amd) (**eg: kgpe-d16**)
- [PSP in all models after fam15h](https://doc.coreboot.org/soc/amd/psp_integration.html)

Power9
====

- Blobless.
