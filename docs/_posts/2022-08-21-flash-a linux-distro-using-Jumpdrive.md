---
layout: post
title:  "Flash a linux distro using Jumpdrive to a PinePhone"
date:   2022-08-19 19:19:05 +0530
categories: jekyll update
---

If you do not want keep your os distro on the sd card inserted to the [PinePhone](https://www.pine64.org/pinephone/),
[download the jumpdriver](https://github.com/dreemurrs-embedded/Jumpdrive/releases/) and  flash it to the sd card in hand.

You may choose a tool of your choice for flashing, but [balenaEtcher](https://github.com/dreemurrs-embedded/Jumpdrive/releases/) is my favorite.

Once the JumpDrive is flashed into the sd card, insert the card to the phone and boot it. By default the phone boots from SD card and starts running in Jumpdrive mode.
The phone reads something like this.
![image](/600px-Jumpdrive.jpg){: width="250" }

Connect your phone to a computer.

Open balenaEtcher and in the source, you will find your phone just like another USB connected.

This shows that the phone is ready and go ahead to flash your os of choice you already downloaded.

Once the flashing is completed successfully disconnect the phone, remove the sd drive and reboot.

Voila!!

![Ubuntu-Touch](/Ubuntu-Touch.png){: width="500"}

Your PinePhone now boots from the internal eMMc drive with your favorite Linux flavor.