---
layout: post
title:  "Flash a linux distro using Jumpdrive to a PinePhone"
date:   2022-08-19 19:19:05 +0530
categories: jekyll update
---

If you dont want keep your os distro on the sd card inserted to the PinePhone,
download the jumpdriver flash it using balena etcher to an sd card.

Insert the card to the phone and boot it. By default the phone boots from SD card and starts running in Jumpdrive mode.

Connect your phone to a computer.

You fill find your phone just like another USB connected.

![image](/600px-Jumpdrive.jpg){: width="250" }
{{ /600px-Jumpdrive.jpg:img?width=250 alt='jmpd-image' }}

Flash your os of choice you already downloaded to this USB you just got listed in the balena etcher.

Once the flashing is completed successfully disconnect the phone, remove the sd drive and reboot.

Voila!!

![Ubuntu-Touch](/Ubuntu-Touch.png){: width="500"}

Your PinePhone now boots from the internal eMMc drive with your favorite Linux flavor.