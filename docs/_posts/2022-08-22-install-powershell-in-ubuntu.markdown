---
layout: post
title:  "Install Poweshell in Ubuntu"
date:   2022-08-22 17:27:00 +0530
categories: jekyll update
---

Sometimes you may need to make the poershell shell to live inside the bash shell.

Thankfully powershell modules are fully compatable and portable in Linux distro.

For Ubuntu you may use this script:

```bash
# Update the list of packages
sudo apt-get update
# Install pre-requisite packages.
sudo apt-get install -y wget apt-transport-https software-properties-common
# Download the Microsoft repository GPG keys
wget -q "https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb"
# Register the Microsoft repository GPG keys
sudo dpkg -i packages-microsoft-prod.deb
# Update the list of packages after we added packages.microsoft.com
sudo apt-get update
# Install PowerShell
sudo apt-get install -y powershell
# Start PowerShell
pwsh
```

And, that's it you can run the powershell commands now from a bash prompt...

{% highlight powershell %}
iarunpaul@DESKTOP-XYZ:~/code/docs$ Write-Output "This is a powershell command."
{% endhighlight  %}