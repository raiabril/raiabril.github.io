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
TARGET=10.129.73.196
echo "10.129.73.196 redeemer.htb" | sudo tee -a /etc/hosts
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

## Gaining access

## Internal reconnaissance

## Maintaining access

### Command and Control

## Privilege escalation / lateral movements

## Exfiltration

## Covering tracks

Not needed as we are talking of a HTB machine...