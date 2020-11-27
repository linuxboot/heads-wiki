---
layout: default
title: TOTP mismatch
permalink: /TOTP-mismatch
nav_order: ?
parent: Help
---

TOTP mismatch
===

My TOTP doesn't match with the one on my phone!
---

There are two possibilities why your TOTP mismatches with that on your authentication device. `(authentication device or phone?)`

1. The time on your laptop has drifted off and is no longer in sync with your phone.
2. You or your administrator have updated the firmware. `(Should the administrator thing be kept in here?)`
3. Your firmware has been manipulated.

Please keep in mind that the first and second point are *far more* likely than the third. `(It does depend a bit on the threat envionment of the user. Doesn't it? Or is it fair to phrase it this way, because it is written for beginners?)`

Dissolve the issue.
---

´(Add pictures?)´

1. Set the hardware clock to the current time.
Please note: If the date shown in the Heads menu is in the year 1970 your CMOS battery is dead. This is the battery which is giving the energy for your laptops internal clock. You have to replace it before your continue with this guide[^1].
[^1]: You will find a guide on the internet for this. ´(To general? What is the recommendation if the user has never disassembled its laptop? Repair shop -> Security implications. Do it by yourself? -> Needs a good guide.)´

> - To get the current time open [this site](https://time.is/GMT) on another device (JavaScript needs to be enabled to see the seconds continuously).
>	*Do not change the time zone! Otherwise your TOTPs will mismatch again.*
> - Boot your laptop.
> - In the Heads menu go to: *Options* -> *TOTP mismatches* -> *I forgot* -> *Execute rescue shell*
> - Select "Yes" on the newly opened window (you don't have to write down the commands shown there, they will be shown in the command line).
> - Type the following command and replace HH:MM:SS with the time shown on the website (HH = hours, MM = minutes, SS = seconds). Execute it by pressing Return.
>	*It is recommended to enter the next full minute and execute the command the moment this minute is reached.
>	E.g.: If it is 19:29:08, replace HH:MM:SS with 19:30:00 and press Return the moment it gets half past 7.*
>	`date -s HH:MM:SS && hwclock -w`
> - Reboot your laptop by typing `reboot` and pressing Return.


Your TOTP should match again with that on your phone.

If the TOTP doesn't match after reboot consider those points:
- Have you replaced HH:MM:SS with the time of the time zone GMT-0?
- Is the date and time shown in the Heads menu the one you have just entered? If not, your CMOS battery is dead. Replace it and set the time again.
- Could your phones time be off?

If the TOTP still isn't matching consider the following point.

2. You or you administrator have updated the firmware,

When Heads and the firmware get updated the TOTP will mismatch afterwards.
If you've got an administrator who maintains your devices ask her or him before following the next steps.

If your administrator or yourself have updated or altered the firmware the problem can likely be fixed by resetting the TOTP secret.
`(Add warning here? If there were manipulations by an attacker one would reset the secret and think anything is fine.)`
This can be fixed by resetting the TOTP secret.

TODO: ADD RESET GUIDE.

If you **haven't done anything** in this regard the third cause is possible. 

3. Your firmware has been manipulated.
**Please note: This possibility is not most unprobable. Make sure that you have followed thoroughly the steps in solution 1! If you did so, are you sure that you or your administrator haven't updated Heads and the firmware** `(Again, the thread environment question.)`

´´´
TODO: What are the recommendations for this one.
Reflash and restore the backup? What if the backup is infected etc.?
Contact someone from here? Contact the EFF? I think I have read something that they've got emergency teams for journalists and human right defenders.
´´´

More details about the issues cause.
---

´(Add tlaurions draft here? It is really well written.)´


References and further reading.
---
- [Why do I have to update my clock in Heads?](https://zeronet.now.im/1DMb3CV66qZPwJqkgm4z12nu8BrAwDoD4g/?Post:44:Setting+time+into+Heads+when+TOTP+code+is+bad)
- How TOTP works: [RFC](https://tools.ietf.org/html/rfc6238) and [Wikipedia](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm)
- Why time zones doesn't matter for TOTPs. The alogrithm uses [Unix time](https://en.wikipedia.org/wiki/Unix_time)
