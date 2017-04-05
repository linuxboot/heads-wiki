Keys and passwords in Heads
====

* Bootguard ACM fuses (hash of OEM public key)
* TPM Owner password (used to initialize counters, NVRAM spaces, etc)
* TPMTOTP secret (shared with phone authenticator)
* TPM disk encryption key (stored in the TPM, sealed with PCRs and encrypted with disk unlock key)
* Disk unlock key (entered by the user on every boot to unseal the TPM disk encryption key)
* Disk recovery key (created when the system is first installed, used rarely, if the TPM fails or if the PCRs change)
* LUKS disk encryption key (two copies, one encrypted with TPM disk encryption key, one encrypted with disk recovery key)
* User GPG private key (stored in a hardware token, not available to normal system)
* User GPG public key (stored in the ROM, used to validate xen, Linux dom0, etc)
* User login password
* Root password

TODO: Document the security implications of each key

PCRs
====
The actual assignment needs to be updated in the code; there are outstanding issues (
[MRC cache](https://github.com/osresearch/heads/issues/150),
[SMM reloc](https://github.com/osresearch/heads/issues/13),
[double measurement](https://github.com/osresearch/heads/issues/15)
) that need to be resolved as well.  Until then this is a rough draft.

0: Boot block

1: ROM stage

2: RAM stage (and MRC? this should be separate)

3: Heads Linux kernel and initrd

4: Boot mode (0 during `/init`, then `recovery` or `normal-boot`)

5: Heads Linux kernel modules

6: Drive LUKS headers
