---
title: fcrackzip
date: 2022-10-29 20:55:00 +0800
categories: [Tools]
tags: [ZIP, crack, password]
pin: false
---

[fcrackzip](https://manpages.ubuntu.com/manpages/xenial/man1/fcrackzip.1.html) is a very nice tool to crack password protected files using dictionaries and brute force.

To install we just need to run `apt` (for Ubuntu/Debian).
```console
$ sudo apt install fcrackzip
```

To test how it works let's create a zip file with our flag.
```console
$ sudo zip --password abc123 file.zip Desktop/user.txt
```

Then, let's try to break it up using a dictionary.
```console
$ fcrackzip -D -p /usr/share/john/password.lst -u -v -c 'a1' file.zip
found file 'Desktop/user.txt', (size cp/uc     53/    41, flags 9, chk 66d3)


PASSWORD FOUND!!!!: pw == abc123
```
Brute force is also effective in this case.

```console
$ fcrackzip -b -l 1-6 -u -v -c 'aA1!' file.zip     
found file 'Desktop/user.txt', (size cp/uc     53/    41, flags 9, chk 66d3)
checking pw aavpv1                                  

PASSWORD FOUND!!!!: pw == abc123
```
