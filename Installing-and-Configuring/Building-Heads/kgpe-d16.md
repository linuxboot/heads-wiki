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

Install nix
[ -d /nix ] || sh <(curl -L https://nixos.org/nix/install) --no-daemon
. /home/user/.nix-profile/etc/profile.d/nix.sh

Enable flake support
mkdir -p ~/.config/nix
echo 'experimental-features = nix-command flakes' >>~/.config/nix/nix.conf

Build docker image
nix --print-build-logs --verbose develop --ignore-environment --command true
nix --print-build-logs --verbose build .#dockerImage && docker load < result

Build ISO
docker run -e DISPLAY=$DISPLAY --network host --rm -ti -v $(pwd):$(pwd) -w $(pwd) linuxboot/heads:dev-env
make BOARD=UNMAINTAINED_kgpe-d16_workstation-usb_keyboard
