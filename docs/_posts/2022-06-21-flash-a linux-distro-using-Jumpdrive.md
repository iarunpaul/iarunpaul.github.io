---
layout: post
title:  "Flash a linux distro using Jumpdrive to a PinePhone"
date:   2022-06-21 17:12:05 +0530
categories: jekyll update
---

If you do not want keep your os distro on the sd card inserted to the [PinePhone](https://www.pine64.org/pinephone/),
[download the jumpdriver](https://github.com/dreemurrs-embedded/Jumpdrive/releases/) and  flash it to the sd card in hand.

You may choose a tool of your choice for flashing, but [balenaEtcher](https://www.balena.io/etcher/) is my favorite.

Once the JumpDrive is flashed into the sd card, insert the card to the phone and boot it. By default the phone boots from SD card and starts running in Jumpdrive mode.

The phone reads something like this.

![image](/images/2022-06-21-flash-a linux-distro-using-Jumpdrive/600px-Jumpdrive.jpg){: width="250" }

Connect your phone to a computer.

Open balenaEtcher and in the source, you will find your phone just like another USB connected.

This shows that the phone is ready and go ahead to flash your os of choice you already downloaded.

Once the flashing is completed successfully disconnect the phone, remove the sd drive and reboot.

Voila!!

![Ubuntu-Touch](/images/2022-06-21-flash-a linux-distro-using-Jumpdrive/Ubuntu-Touch.png){: width="500"}

Your PinePhone now boots from the internal eMMc drive with your favorite Linux flavor.