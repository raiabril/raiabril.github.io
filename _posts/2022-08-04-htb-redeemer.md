---
title: HTB Redeemer
date: 2022-08-04 00:55:00 +0800
categories: [HTB, Starting Point]
tags: [Linux, Redis, Enumeration, Anonymous/Guest access, HTB]
pin: false
---

![redeemer-htb](https://www.hackthebox.com/storage/avatars/cdf77651ab0a4eca65acd5cf388b4c66.png)

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

We get the response, the machine is Linux.

```console
PING 10.129.73.196 (10.129.73.196) 56(84) bytes of data.
64 bytes from 10.129.73.196: icmp_seq=1 ttl=63 time=57.5 ms

--- 10.129.73.196 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 57.510/57.510/57.510/0.000 ms

```

### NMAP

To scan the `target` to find open ports and possible vulnerabilities we use `nmap`.

First, simple `TCP` scan without `DNS` resolution and ping discovery, to all the ports and with the version detection.

```console
nmap -n -Pn -sV -p- $TARGET -vvv -oG allPorts
```

The service running is a Redis, which is an in-memory database.

```console
PORT     STATE SERVICE REASON  VERSION
6379/tcp open  redis   syn-ack Redis key-value store 5.0.7
```

### Redis

If you don't have you can install redis tools.

```console
sudo apt install redis-tools -y
```

It looks like the Redis is open, we just need to connect.

```console
redis-cli -h $TARGET
```

The reference for Redis commands is [here](https://redis.io/commands/)

The important commands here are:

- `INFO` it will help to see the information and stats of the Redis server. As we can see it's a master and we have 1 database `db0`
- `SELECT 0` to select the database.
- `KEYS *` to display all the available keys.
- `GET flag` to see the flag.
