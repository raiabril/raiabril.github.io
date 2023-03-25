---
title: Useful commands
date: 2023-03-24 20:55:00 +0800
categories: [HTB]
tags: [Commands]
pin: false
---

## Start listener

```console
# For Linux
sudo nc -nvlp 4444

# For Windows
sudo rlwrap nc -nvlp 443
```

## Listening web service

```console
sudo python3 -m http.server 80
```
