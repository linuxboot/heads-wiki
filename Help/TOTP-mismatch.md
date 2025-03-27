---
layout: default
title: TOTP mismatch
permalink: /TOTP-mismatch/
nav_order: 1
parent: Help
---

# TOTP mismatch

There are three possibilities why your TOTP mismatches with that on your phone.

1. The time on your laptop has drifted off and is no longer in sync with your phone.
2. You have updated the firmware.
3. Your firmware has been manipulated.

## Dissolve the issue.

### **1. Set the hardware clock to the current time**

Please note: If the date shown in the Heads menu is in the year 1970 your CMOS battery is dead. You have to replace it before your continue with this guide.

> - To get the current time open [this site](https://time.is/GMT) on another device (JavaScript needs to be enabled to see the seconds continuously).
>	*Do not change the time zone! Otherwise your TOTPs will mismatch again.*
> - Boot your laptop.
> - In the Heads menu go to: *Options* -> *TPM/TOTP/HTOP Options* -> *TOTP/HTOP does not match after refresh, troubleshoot* -> *Execute rescue shell*
> - Select "Yes" on the newly opened window (you don't have to write down the commands shown there, they will be shown in the command line).
> - Execute the following command. Replace HH:MM:SS with the time shown on the website (HH = hours, MM = minutes, SS = seconds).
>
>	*It is recommended to enter the next full minute and execute the command the moment this minute is reached.
>	E.g.: If it is 19:29:08, replace HH:MM:SS with 19:30:00 and press return the moment it gets half past 7.*
>
>	`date -s HH:MM:SS && hwclock -w`
> - Reboot your laptop with `reboot`.


Your TOTP should match again with that on your phone.

If the TOTP doesn't match after reboot consider those points:
- Have you replaced HH:MM:SS with the time of the time zone GMT-0?
  Other time zones than GMT-0 will generate TOTPs of the future or past because the alogrithm uses [Unix time](https://en.wikipedia.org/wiki/Unix_time)
- Is the date and time shown in the Heads menu the one you have just entered?
  If not, your CMOS battery is likely dead. Replace it and set the time again.
- Could your phones time be off?

If the TOTP still isn't matching consider the following.

### **2. You have updated the firmware**

When Heads and the firmware get updated the TOTP will mismatch afterwards.

If you have just forgot to reset the TOTP secret after the the update set a new TOTP secret.
In the Heads menu go to *Options* -> *TPM/TOTP/HOTP Options* -> *Generate new TOTP/HTOP secret*.

If you **haven't done anything** in this regard, the third cause is possible. 

### **3. Your firmware has been manipulated**

<hr>
**Please note:** This possibility is the most unprobable. Make sure that you have followed thoroughly the steps in solution 1!
<hr>

At this point you can assume you have been compromised.

In general you can build Heads again on a trusted machine and flash it. Afterwards reinstall Qubes.

**But** please consider that the malware might be hidden in some other firmware (battery, keyboard) which is not meassured by Heads.
Buying a new laptop directly from the store, flash Heads and install Qubes on it is probably the safest option.

References and further reading.
---
- [Why do I have to update my clock in Heads?](https://zeronet.now.im/1DMb3CV66qZPwJqkgm4z12nu8BrAwDoD4g/?Post:44:Setting+time+into+Heads+when+TOTP+code+is+bad)
- How TOTP works: [RFC](https://tools.ietf.org/html/rfc6238) and [Wikipedia](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm)

