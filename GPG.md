Generating a new card
===

 NOTE: *gpg 2.1.21 is required to create 4096 bits keys, which is not provided with heads for the moment.*

```
diamond:~/clean/heads: gpg --card-edit --homedir ./initrd/.gnupg

gpg: detected reader `Yubico Yubikey NEO OTP+CCID 00 00'
Application ID ...: D2760001240102000006053147090000
Version ..........: 2.0
Manufacturer .....: unknown
Serial number ....: 05314709
Name of cardholder: [not set]
Language prefs ...: [not set]
Sex ..............: unspecified
URL of public key : [not set]
Login data .......: [not set]
Signature PIN ....: forced
Key attributes ...: 2048R 2048R 2048R
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 3 3
Signature counter : 0
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]
General key info..: [none]

gpg/card> admin
Admin commands are allowed

gpg/card> generate
Make off-card backup of encryption key? (Y/n) n

Please note that the factory settings of the PINs are
   PIN = `123456'     Admin PIN = `12345678'
You should change them using the command --change-pin

gpg: 3 Admin PIN attempts remaining before card is permanently locked

Please enter the Admin PIN
                 
Please enter the PIN
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Thu 12 Apr 2018 09:36:15 AM EDT
Is this correct? (y/N) y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: Heads Firmware
Email address: heads@osresearch.net
Comment: 
You selected this USER-ID:
    "Heads Firmware <heads@osresearch.net>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
gpg: generating new key
gpg: please wait while key is being generated ...
gpg: key generation completed (7 seconds)
gpg: signatures created so far: 0
gpg: generating new key
gpg: please wait while key is being generated ...
gpg: key generation completed (18 seconds)
gpg: signatures created so far: 1
gpg: signatures created so far: 2
gpg: generating new key
gpg: please wait while key is being generated ...
gpg: key generation completed (13 seconds)
gpg: signatures created so far: 3
gpg: signatures created so far: 4
gpg: key 9F7861CD marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, classic trust model
gpg: depth: 0  valid:   3  signed:   7  trust: 0-, 0q, 0n, 0m, 0f, 3u
gpg: depth: 1  valid:   7  signed:   2  trust: 5-, 0q, 0n, 0m, 2f, 0u
gpg: next trustdb check due at 2017-10-06
pub   2048R/9F7861CD 2017-04-12 [expires: 2018-04-12]
      Key fingerprint = 7679 00D2 E314 B920 91C2  ED88 7E28 A403 9F78 61CD
uid                  Heads Firmware <heads@osresearch.net>
sub   2048R/B4DB844D 2017-04-12 [expires: 2018-04-12]
sub   2048R/7E276412 2017-04-12 [expires: 2018-04-12]


gpg/card>
diamond:~/clean/heads: gpg --change-pin
gpg: detected reader `Yubico Yubikey NEO OTP+CCID 00 00'
gpg: OpenPGP card no. D2760001240102000006053147090000 detected

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 3
gpg: 3 Admin PIN attempts remaining before card is permanently locked

Please enter the Admin PIN
                 
New Admin PIN
                     
New Admin PIN
PIN changed.     

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1

Please enter the PIN
           
New PIN
               
New PIN
PIN changed.     

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? q
```

Fully resetting a card
===
If you mess up the admin pin it is necessary to do a full reset of
the PGP applet.  These instructions worked for me:
https://developers.yubico.com/ykneo-openpgp/ResetApplet.html
I had to install `scdaemon` and start `gpg-agent` to make them
work, but eventually that `gpg-connect-agent -r yubikey.reset`
script did the right thing:

```
/hex
scd serialno
scd apdu 00 20 00 81 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 81 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 81 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 81 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 83 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 83 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 83 08 40 40 40 40 40 40 40 40
scd apdu 00 20 00 83 08 40 40 40 40 40 40 40 40
scd apdu 00 e6 00 00
scd apdu 00 44 00 00
```

