---
title: HTB Template
date: 2052-08-03 20:55:00 +0800
categories: [HTB]
tags: [Network, Easy, Internal, Telnet, CVE-2012-5519, SNMP, HTB]
pin: false
---

## Reconnaissance/Intelligence Gathering

In this step we collect the target information available in public repositories or sources. We do everything passively.

## Scanning and enumeration

Now it's time to start the active scanning.

As always, we define our TARGET and hosts file of our machine to facilitate the process.

```console
TARGET=<ip_of_the_machine>
sudo nano /etc/hosts # Add a line with the machine IP /t antique.htb
```

We launch a single `TCMP` probe to check ping.

```console
ping -c 1 $TARGET		# => Ping is working
```

## Gaining access

## Internal reconnaissance

## Maintaining access

### Installation

### Command and Control

## Privilege escalation / lateral movements

## Exfiltration

## Maintaining access

Not needed as we are talking of a HTB machine...

## Covering tracks

Not needed as we are talking of a HTB machine...