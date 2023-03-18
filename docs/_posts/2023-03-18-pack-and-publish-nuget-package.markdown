---
layout: post
title:  "Pack and publish a NuGet package"
date:   2022-08-30 00:27:00 +0530
categories: jekyll update
# published: false
---

I wanted to run `ip` command on my Ubuntu 18.04.

The package was not installed, so tried a package search....

```bash
root@nginx$ apt list -a ip
```

and

```bash
root@nginx$ dpkg --list ip
```

Lists nothing...

<br>

The `apt-file` utility is a saviour here.

If `apt-file` is not yet installed, you can use the `apt list -a apt-file` and get it installed.

Now you can find the package we use by the command its known for..

```bash
root@nginx$ apt-file update

root@nginx$ apt-file search -regexp '/bin/ip$'
```

It yields the package name as the ouput.

```bash
iproute2: /bin/ip
```

You can also search the package with another command
```bash
 apt-file search --regexp '/bin/route'
 ```
 The result will be like 
 ```bash
iproute2: /usr/bin/routef
iproute2: /usr/bin/routel
```


Now you can install the package by
```bash
root@nginx$ apt install iproute2
```
Check if the package is installed.

```bash
root@nginx$ dpkg -l iproute2
```
which should show something like

```bash
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                     Version           Architecture      Description
+++-========================-=================-=================-=====================================================
ii  iproute2                 4.15.0-2ubuntu1.3 amd64             networking and traffic control tools
```




