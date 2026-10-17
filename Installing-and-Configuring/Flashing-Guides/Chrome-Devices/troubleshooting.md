---
layout: default
title: Troubleshooting
permalink: /google-devices-troubleshooting/
nav_order: 12
parent: Chrome Devices
grand_parent: Step 2 - Flashing Guides
---

# Hardware Write Protection
How to check the status of hardware write protection

```
# Check the baud rate
sudo stty -F /dev/ttyUSB0

sudo minicom -D /dev/ttyUSB0 -b <baud_rate from above>
< Press Enter >
wp
```

Here is it disabled:
```
Flash WP: disabled
 at boot: follow_batt_pres
```
