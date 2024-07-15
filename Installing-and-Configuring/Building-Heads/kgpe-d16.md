---
layout: default
title: Asus KGPE-D16
permalink: /kgpe-d16-building/
nav_order: 1
parent: Step 1 - Building Heads
grand_parent: Installing and configuring
---

Asus KGPE-D16
====
Clone repo
git clone https://github.com/linuxboot/heads.git

Follow README.md instructions for building located here https://github.com/linuxboot/heads/blob/master/README.md

After having prepared nix and docker you can build using:
make BOARD=UNMAINTAINED_kgpe-d16_workstation-usb_keyboard
