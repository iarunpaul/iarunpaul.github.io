---
layout: post
title:  "Gmail backup as Read-Only Local Mail Server"
date:   2025-02-01 22:30:00 +0530
categories: jekyll update
published: true
---
<div style="text-align: center;">
  <img src="\images\2025-02-01-backingup-gmail2025-02-01-backingup-gmail\gmail-local-mail-server.webpgmail-local-mail-server.webp" style="width: 500px; height: auto;">
</div>
<div style="text-align: center;">
Courtesy: DALL-E</div>



---

<br>
<br>

### Introduction

Gmail is a great email service, but when we deal with so many email for most your communications and accomplishments, we may end up hanging out `gmail` cloud storage at stake and blocking your emai service could be disasterous at times. 

Having a local backup ensures that you retain access to your important emails, even if your account is locked or Google services are down. 

Even better, if you could run a local mail server with a read-only back up, ahh... nothing like that.

But its easier than you think of.

In this blog, we’ll cover how to backup Gmail and set up a read-only local mail server using `Dovecot`. This setup allows you to access your emails from email clients like Thunderbird, or in devices like iPhone, Android, or any IMAP client, while preventing accidental deletions or modifications.


----



#### Step 1: Exporting Emails from Gmail

1. Use Google Takeout

Google provides [Google Takeout](https://takeout.google.com/) to export your Gmail data:

Visit Google Takeout.

Select Mail and ensure MBOX format is selected.

Click Next Step, choose file size and type.

Click Create Export and wait for Google to process it.

Once ready, download the MBOX file.


----



#### Step 2: Setting Up a Local Mail Server

I am using the NAS set up I have detailed in the [Share Pine with Apple](https://blog.iarunpaul.com/jekyll/update/2025/01/27/share-pine-with-apple.html)

We will use `Dovecot`, an IMAP server, to allow access to the backed-up emails on your local network.

1. Install Dovecot

On a Linux server (Debian/Ubuntu). I am using my PinePhone here:


```bash
sudo apt update
sudo apt install dovecot-core dovecot-imapd
```


Enable and start Dovecot:

```bash
sudo systemctl enable dovecot
sudo systemctl start dovecot
```

2. Configure `Dovecot` to Serve MBOX Files

Edit the `Dovecot` configuration:

```bash

sudo vi /etc/dovecot/conf.d/10-mail.conf
```


Set the mail_location to point to your MBOX backup:


```ini
mail_location = mbox:/var/mail/backup:INBOX=/var/mail/backup/inbox

```

Ensure the folder exists and is accessible:

```bash

sudo mkdir -p /var/mail/backup
sudo chown -R dovecot:dovecot /var/mail/backup
sudo chmod -R 750 /var/mail/backup

```


3. Import Gmail MBOX File

Move your Google Takeout MBOX file to the server:

```bash

scp Gmail-backup.mbox user@server:/var/mail/backup/inbox

```

Use the following command to import emails from the MBOX file:

```bash
sudo doveadm -D import -u backup "mbox:/var/mail/backup" INBOX ALL
```


To monitor the import process in real-time:

```bash
sudo journalctl -u dovecot -f
```

After the import, verify the number of messages imported:

```bash
sudo doveadm mailbox status -u backup messages INBOX

```


If necessary, force a reindex for accurate search:


```bash
sudo doveadm index -u backup INBOX
sudo doveadm force-resync -u backup INBOX
```

Ensure the permissions are correct:

```bash

sudo chown dovecot:dovecot /var/mail/backup/inbox
sudo chmod 640 /var/mail/backup/inbox

```

4. Set Dovecot to Read-Only Mode

Edit Dovecot’s configuration file:

```bash

sudo vi /etc/dovecot/conf.d/90-plugin.conf

```

Add:

```ini

plugin {
  mail_readonly = yes
}

```
Restart Dovecot:

```bash

sudo systemctl restart dovecot

```


----



#### Step 3: Accessing Your Local Mail Server

Once `Dovecot` is running, you can access your emails via any IMAP client (Thunderbird, iPhone, Android, Outlook).

1. Find Your Server’s Local IP

On your Linux machine, run:

```bash

ip a | grep inet

```

Example output:

```ini

inet 192.168.1.100/24

```

Your server IP is 192.168.1.100.

2. Configure Thunderbird (Windows/Linux)

Open Thunderbird → Click Menu → Account Settings.

Click Add Mail Account.

Enter:

Email Address: backup@local

Password: (Leave blank or use a dummy password)

Click Configure Manually and enter:

IMAP Server: 192.168.1.100

Port: 143 (or 993 for SSL)

Authentication: None (since it’s read-only)

Click Done.

3. Configure iPhone (iOS Mail App)

Go to Settings `→` Mail `→` Accounts.

Tap Add Account `→` Other `→` Add Mail Account.

Enter:

Name: Gmail Backup

Email: backup@local

Password: (Leave blank or use a dummy password)

Select IMAP and enter:

Server: 192.168.1.100

Port: 143 (or 993 for SSL)

Authentication: None

Save and check if emails appear.

4. Import MBOX Files in Thunderbird

Thunderbird does not support MBOX files by default. Use ImportExportTools NG:

Open Thunderbird → Click `☰ ` Menu → `Add-ons and Themes`.

Search for ImportExportTools NG and install it.

Right-click Local Folders → ImportExportTools NG → Import mbox file.

Select "Import directly one or more mbox files" and click OK.

Choose your MBOX file and click Open.

Now your emails should be accessible in Thunderbird.

----



#### Step 4: Enabling Search for Faster Email Access

By default, search may not work well because Dovecot does not index email content. Enable Full-Text Search (FTS) to improve searching.

1. Enable FTS in Dovecot

Edit the plugin configuration:

```bash

sudo nano /etc/dovecot/conf.d/90-plugin.conf

```

Add:

```ini
plugin {
  fts = lucene
  fts_lucene = partial=4 full=20
}
```
Restart `Dovecot`:

```bash
sudo systemctl restart dovecot
```

2. Rebuild the Search Index

Run:

```bash
sudo doveadm index -u backup INBOX
sudo doveadm force-resync -u backup INBOX

```

Now, Thunderbird and mobile clients should be able to search emails quickly.


----



#### Conclusion

With this setup, you now have a Gmail backup stored locally, accessible via IMAP but in read-only mode. This ensures that:

✅ Emails are safe and accessible at any time.

✅ No accidental deletions or modifications can occur.

✅ You can search and browse emails using Thunderbird, iPhone, or any IMAP client.

This is an ideal self-hosted email archive solution. Let me know if you have any questions or need further tweaks! 🚀

*Happy Coding...*

