Keys and passwords in Heads
====

* Management Engine signing key
* Bootguard ACM fuses (hash of OEM public key)
* TPM Owner password (used to initialize counters, NVRAM spaces, etc)
* TPM Endorsement Key
* TPMTOTP secret (shared with phone authenticator)
* TPM disk encryption key (stored in the TPM, sealed with PCRs and encrypted with disk unlock key)
* Disk unlock key (entered by the user on every boot to unseal the TPM disk encryption key)
* Disk recovery key (created when the system is first installed, used rarely, if the TPM fails or if the PCRs change)
* LUKS disk encryption key (two copies, one encrypted with TPM disk encryption key, one encrypted with disk recovery key)
* User GPG private key (stored in a hardware token, not available to normal system)
* User GPG public key (stored in the ROM, used to validate xen, Linux dom0, etc)
* User login password
* Root password

Management Engine and Bootguard ACM fuses
---
The very first key used in the system is Intel's public key that signs the Management Engine firmware partition table in the SPI flash.  This key is stored in the on-die ROM of the ME and the ME will not start up if this signature does not match.  An attacker who controls this key (which is highly unlikely) can subvert the Bootguard checks as well as the measured boot process.

The [Bootguard fuses](https://trmm.net/Bootguard) fuses provide protection against most "evil maid" attacks against the firmware.  The hash of the ACM signing key is set in write-once fuses in the CPU chipset and during the CPU bringup phase the ME and the CPU microcode cooperate in some undocumented way to validate the "Startup ACM" in the SPI flash.  Since this key is fused into hardware, an evil maid attack would need to replace the CPU to install malicious firmware into the SPI flash.  An attacker who controls this key can flash new firmware via hardware means (and possibly remotely via software, unless other steps are taken).

TPM Owner password
---
As part of setting up Heads the TPM is "owned" by the user and the owner password is set.  This clears all existing NVRAM and spaces (but does not reset counters?).

Are there any consequences of an attacker controlling this key?

TPMTOTP shared secret
---
Since humans have trouble doing RSA public key cryptography in their brains, Heads uses TPMTOTP to let the system attest to the user that the firmware is unmodified.  During system setup a random 20-byte value is generated and shared (via QR code) to the user's phone as well as sealed with the correct TPM PCR values into the TPM NVRAM.  On subsequent boots the TPM will unseal the secret if and only if the PCRs match, and the computer then generates a one-time password based on the current clock time, which the user can compare to the value displayed on their phone.  A new secret must be generated each time the firmware is updated since this will change the PCRs.

If an attacker can control this shared secret (such as by directly sending PCR values into the TPM) they can install malicious firmware in the SPI flash and generate valid TOTP codes.

TPM disk encryption key
---
The TPM NVRAM also stores one of the disk encryption keys, which is encrypted with the user's disk unlock password and sealed with the TPM PCR values for the firmware and the LUKS headers on the disk.  On every boot the user types in their password and the TPM will unseal and decrypt the disk encryption key if and only if the firmware is unmodified and the password matches.  Since the TPMTOTP one-time code matched, the user can have confidence that the firmware is unmodified before they enter their password.

The sealed blob is not secret since it is both encrypted and sealed, but if an attacker can extract this unsealed and decrypted key they can decrypt the data on the disk.  If they extract the TPM they can set the PCRs to the correct values and attempt to brute force the unlock code, although the TPM should provide some rate limiting. TODO: Can the TPM also flush the keys if too many attempts are made?

Disk recovery key
---
During initial system setup the disk is encrypted with a user chosen passphrase.  This key is only entered if the TPM PCRs have been changed, such as following a firmware update, or if the disk has been moved to a new machine and the user needs to back up the code.

If an attacker gains control of this recovery key they can decrypt the disk.


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
