---
layout: post
title:  "Create a local NAS at home: Share Pine with Apple 🍍🍏"
date:   2025-01-27 10:25:00 +0530
categories: jekyll update
published: true
---
<div style="text-align: center;">
  <img src="\images\2024-01-18-Apple-Pine-NAS\apple-pine-nas.webp" style="width: 500px; height: auto;">
</div>
<div style="text-align: center;">
Courtesy: DALL-E</div>



---

<br>
<br>

### 🚨 The Problem: iCloud Full, Now What?


You wake up to a frustrating notification on your iPhone:

> **"Your iCloud storage is full. Upgrade to 2TB?"**

Apple’s solution? Pay more every month.

But what if you could take control of your storage and host your own NAS(Network Attached Storage)—accessible from your iPhone, iPad, Mac, Windows, and Linux devices?

Enter the PinePhone NAS—a low-power, always-on storage solution using Samba (SMB) and a USB SSD to create a private iCloud alternative.

Of course, it is not a robust NAS, blistering fast or a cloud based solution that you take anywher eyou move around.

But the general use case, atleast for me was found to be quite adequete to have a decent performing NAS in my local network.

I would always have something like a PinePhone handy, when I some tinker jobs, are spinning in my head.

>Raspberry pi or ArmSoM Sige7 or any popular single board computers would have been a build for the purpose; but I am too hesitant to spend a penny today.

*In fact, I was half way through the challenge (or fun 😜 ) of transforming a real mobile phone into a network storage.*

----

### 🛠️ Setting Up PinePhone as a NAS

We'll turn a PinePhone running Mobian into a Samba-powered NAS that works with iOS Files, macOS Finder, Windows Explorer, and Android file managers.

#### 1️⃣ Connect an External SSD


```bash
lsblk -f

```


Mount it:


```bash
sudo mkdir -p /mnt/nas_storage
sudo mount /dev/sda1 /mnt/nas_storage

```

To automount on boot, add this to /etc/fstab:

```ini
UUID=<your-ssd-uuid> /mnt/nas_storage ext4 defaults 0 0

````
Verify:

```bash
df -h
```
---

#### 2️⃣ Install & Configure Samba

Install Samba:

```bash
sudo apt update && sudo apt install samba -y

```

Edit the Samba config:

```bash
sudo vi /etc/samba/smb.conf

```

Add:
```ini
[global]
   unix extensions = no
   wide links = yes
   follow symlinks = yes
   max protocol = SMB3
   mangled names = no

[NAS_Share]
   comment = PinePhone NAS
   path = /mnt/nas_storage
   browseable = yes
   writable = yes
   guest ok = no
   force user = mobian
   force group = mobian
   create mask = 0777
   directory mask = 0777
   valid users = mobian

```


Restart Samba:

```bash
sudo systemctl restart smbd nmbd

```
---

#### 3️⃣ Access NAS from iPhone, Mac, Windows

📲 iPhone & iPad
1. Open the Files app.
2. Tap Browse → … (More) → Connect to Server.
3. Enter:

```cpp
smb://192.168.x.x(local.domain)/NAS_Share

```
4. Login as mobian, and boom—your own iCloud replacement!


💻 macOS Finder

1. Press Cmd + K in Finder.
2. Enter:

```cpp
smb://192.168.x.x(local.domain)/NAS_Share

```
3. Authenticate and mount the drive.

🖥️ Windows

1. Open File Explorer.
2. In the option `Add a network location` enter:

```
\\192.168.x.x\NAS_Share

```
3. Login and start copying files.

---

### 🔄 Make Your NAS Persistent

💡 Want the NAS to stay mounted across reboots? On macOS/Linux, add this to /etc/fstab:

```arduino

//192.168.x.x/NAS_Share /mnt/nas cifs username=mobian,password=yourpassword,iocharset=utf8 0 0

```
---

### 🚀 The Result: A Personal Cloud

You now have: 

✅ A private, expandable cloud

✅ No monthly fees

✅ Seamless integration with Apple, Windows, and Linux

---

### 📢 What we have achieved
No more rent on cloud storage and you own one now.

Your PinePhone NAS is now a fully functional, cross-platform home cloud—a perfect escape from iCloud's limits.

🛠 What’s next? Add encryption, enable remote access, or partition for readable and writable sections.

Since, I have a shared storage now, next I am thinking, how to back up my Gmail and make it a read-only inbox over the network.

I will come with some cool ideas on it sooner...

👉 Would you build your own NAS? Drop your thoughts below! 🚀

Happy Coding....
