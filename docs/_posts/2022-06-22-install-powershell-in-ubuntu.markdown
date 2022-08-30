---
layout: post
title:  "PowerShell in Bash Shell"
date:   2022-06-22 17:27:00 +0530
categories: jekyll update
---

As the interoperability is increasing day by  day, you may need to make the powershell shell to live inside the bash shell.

Thankfully powershell modules can be packaged and are fully compatable in Linux distro.

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

And, that's it, you can run the powershell commands now from a bash prompt...
```bash
PowerShell 7.2.6
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell
Type 'help' to get help.

PS /home/user/userhome>
```
The powershell comes as an extension, hence you can try some interoperational commands...

Run PoweShell commands with ease
{% highlight powershell %}
PS /home/user/userhome> Write-Output "This is a powershell command."
{% endhighlight  %}

Create a multiline Powershell var
{% highlight powershell %}
PS /home/user/userhome> $iAmPWSHVar=@"
>> I am a
>> powershell var
>> "@
PS /home/user/userhome> $iAmPWSHVar
I am a
powershell var
{% endhighlight  %}

Now lets try a combo with the hardcore Linux `sed` replacement with the powershell variable as input, thats fun ...
{% highlight powershell %}

PS /home/user/userhome> $iAmPWSHVar | sed 's/powershell/powershellBash/g'
I am a
powershellBash var
{% endhighlight  %}

This comes really handy, in cases like when you follow a tutorial in Powershell, but you have to deal only from a bash shell...