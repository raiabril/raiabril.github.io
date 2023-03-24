---
title: HTB Squashed
date: 2023-03-05 22:22:00 +0800
categories: [HTB,Easy]
tags: [Network,Vulnerability Assessment,Common Services,Authentication,Apache,X11,NFS,Reconnaissance,User Enumeration,Impersonation,Arbitrary File Upload,HTB]
pin: false
---

![Antique](https://www.hackthebox.com/storage/avatars/2b64823934eb46f2c531a0b650a03d60.png)

## Reconnaissance/Intelligence Gathering

In this step we collect the target information available in public repositories or sources. We do everything passively.

## Scanning and enumeration

Now it's time to start the active scanning.

As always, we define our TARGET and hosts file of our machine to facilitate the process.

```console
TARGET=10.10.11.191
echo "10.10.11.191 squashed.htb" | sudo tee -a /etc/hosts
```

We launch a single `TCMP` probe to check ping.

```console
ping -c 1 $TARGET
```
Ping is working

### NMAP

To scan the `target` to find open ports and possible vulnerabilities we use `nmap`.

First, simple `TCP` scan without `DNS` resolution and ping discovery, to all the ports and with the version detection.

```console
nmap -n -Pn -p- -sV -vvv -oG allPorts $TARGET

Discovered open port 80/tcp on 10.10.11.191
Discovered open port 22/tcp on 10.10.11.191
Discovered open port 111/tcp on 10.10.11.191
Discovered open port 2049/tcp on 10.10.11.191
Discovered open port 50061/tcp on 10.10.11.191
Discovered open port 37953/tcp on 10.10.11.191
Discovered open port 38277/tcp on 10.10.11.191
```

Lot of ports open.

#### 80/TCP Apache server
```console
ffuf -fc=404 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://squashed.htb/FUZZ

```

#### 111/TCP/UDP - Portmapper
```console
rpcinfo squashed.htb

   program version netid     address                service    owner
    100000    4    tcp6      ::.0.111               portmapper superuser
    100000    3    tcp6      ::.0.111               portmapper superuser
    100000    4    udp6      ::.0.111               portmapper superuser
    100000    3    udp6      ::.0.111               portmapper superuser
    100000    4    tcp       0.0.0.0.0.111          portmapper superuser
    100000    3    tcp       0.0.0.0.0.111          portmapper superuser
    100000    2    tcp       0.0.0.0.0.111          portmapper superuser
    100000    4    udp       0.0.0.0.0.111          portmapper superuser
    100000    3    udp       0.0.0.0.0.111          portmapper superuser
    100000    2    udp       0.0.0.0.0.111          portmapper superuser
    100000    4    local     /run/rpcbind.sock      portmapper superuser
    100000    3    local     /run/rpcbind.sock      portmapper superuser
    100005    1    udp       0.0.0.0.157.180        mountd     superuser
    100005    1    tcp       0.0.0.0.195.141        mountd     superuser
    100005    1    udp6      ::.191.24              mountd     superuser
    100005    1    tcp6      ::.175.193             mountd     superuser
    100005    2    udp       0.0.0.0.167.167        mountd     superuser
    100005    2    tcp       0.0.0.0.149.133        mountd     superuser
    100005    2    udp6      ::.220.240             mountd     superuser
    100005    2    tcp6      ::.224.9               mountd     superuser
    100005    3    udp       0.0.0.0.171.213        mountd     superuser
    100005    3    tcp       0.0.0.0.160.5          mountd     superuser
    100005    3    udp6      ::.166.218             mountd     superuser
    100005    3    tcp6      ::.219.151             mountd     superuser
    100003    3    tcp       0.0.0.0.8.1            nfs        superuser
    100003    4    tcp       0.0.0.0.8.1            nfs        superuser
    100227    3    tcp       0.0.0.0.8.1            nfs_acl    superuser
    100003    3    udp       0.0.0.0.8.1            nfs        superuser
    100227    3    udp       0.0.0.0.8.1            nfs_acl    superuser
    100003    3    tcp6      ::.8.1                 nfs        superuser
    100003    4    tcp6      ::.8.1                 nfs        superuser
    100227    3    tcp6      ::.8.1                 nfs_acl    superuser
    100003    3    udp6      ::.8.1                 nfs        superuser
    100227    3    udp6      ::.8.1                 nfs_acl    superuser
    100021    1    udp       0.0.0.0.163.0          nlockmgr   superuser
    100021    3    udp       0.0.0.0.163.0          nlockmgr   superuser
    100021    4    udp       0.0.0.0.163.0          nlockmgr   superuser
    100021    1    tcp       0.0.0.0.148.65         nlockmgr   superuser
    100021    3    tcp       0.0.0.0.148.65         nlockmgr   superuser
    100021    4    tcp       0.0.0.0.148.65         nlockmgr   superuser
    100021    1    udp6      ::.202.177             nlockmgr   superuser
    100021    3    udp6      ::.202.177             nlockmgr   superuser
    100021    4    udp6      ::.202.177             nlockmgr   superuser
    100021    1    tcp6      ::.147.75              nlockmgr   superuser
    100021    3    tcp6      ::.147.75              nlockmgr   superuser
    100021    4    tcp6      ::.147.75              nlockmgr   superuser
```

#### 2049 - NFS Service
```console
showmount -e 10.10.11.191 

Export list for 10.10.11.191:
/home/ross    *
/var/www/html *

```

## Gaining access
We will mount the home directory of the user ross into a temporary folder. In order to mount, you must use root privileges.

```console
mktemp -d
sudo mount -t nfs -o vers=2 10.10.11.191:/home/ross /tmp/tmp.aCuJtaIhO2 -o nolock
```
We can navigate through the files of ross and find the Keepass DB.



## Internal reconnaissance

## Maintaining access

### Command and Control

## Privilege escalation / lateral movements

## Exfiltration

## Covering tracks

Not needed as we are talking of a HTB machine...