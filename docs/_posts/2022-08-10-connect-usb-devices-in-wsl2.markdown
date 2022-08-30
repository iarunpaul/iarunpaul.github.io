---
layout: post
title:  "Connect USB devices in WSL2"
date:   2022-08-10 00:27:00 +0530
categories: jekyll update
# published: false
---

With the WSL kernel version of 5.10.60.1 or later it is possibel to connect USB devices to WSL2 and use it in the Linux distro.

Install the [latest release of usbipd-win](https://github.com/dorssel/usbipd-win/releases) on the windows.

From the WSL terminal, install the user space tools for USB/IP and a database of USB hardware identifiers.

On Ubuntu 20.04 LTS, run these commands:
```bash
sudo apt install linux-tools-5.4.0-77-generic hwdata
sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/5.4.0-77-generic/usbip 20
```
From the *administrator* command prompt on Windows, run the command to list all usb devices connected to Windows:
```powershell
usbipd wsl list
```
Select the bus ID of the device you’d like to attach to WSL and run this command. You’ll be prompted by WSL for a password to run a sudo command.
```powershell
usbipd wsl attach --busid <busid>
```
From the Linux distro in WSL2, run `lsusb` *(or the similar command depending on the distro you handle)* to list the attached USB devices. You can see the device you have attached and can interact with it using normal Linux tools. You may need to configure udev rules to allow non-root users to access the device.

For disconnecting the device, run the command from an administrator command prompt on Windows.
```powershell
usbipd wsl detach --busid <busid>
```
