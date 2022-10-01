# Skynet

| [![Jenkins](../../_static/images/skynet.png)](https://tryhackme.com/room/skynet) |
|:--:|
| [https://tryhackme.com/room/skynet](https://tryhackme.com/room/skynet) |

A vulnerable Terminator themed Linux machine. 

## Deploy and compromise

### Scanning

First run a simple port scan (without Ping)

	# nmap -Pn -p- <IP address target machine> -oN portscan

portscan:

```text
# Nmap 7.92 scan initiated Sat Oct  1 01:57:07 2022 as: nmap -Pn -p- -oN portscan 10.10.62.253
Nmap scan report for 10.10.62.253
Host is up (0.052s latency).
Not shown: 65529 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds

# Nmap done at Sat Oct  1 01:58:03 2022 -- 1 IP address (1 host up) scanned in 56.21 seconds
```

Run an `-A` scan on the open ports:

	nmap -Pn -T4 -A -p22,80 <IP address target machine> -oN servicescan

servicescan:

```text
# Nmap 7.92 scan initiated Sat Oct  1 02:02:38 2022 as: nmap -Pn -T4 -A -p22,80,110,139,143,445 -oN servicescan 10.10.62.253
Nmap scan report for 10.10.62.253
Host is up (0.042s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: SASL CAPA AUTH-RESP-CODE RESP-CODES TOP UIDL PIPELINING
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: LITERAL+ more have post-login Pre-login IMAP4rev1 IDLE ENABLE listed LOGINDISABLEDA0001 ID OK SASL-IR LOGIN-REFERRALS capabilities
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (95%), Linux 5.4 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.16 (95%), Linux 3.1 (93%), Linux 3.2 (93%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (92%), Sony Android TV (Android 5.0) (92%), Android 5.0 - 6.0.1 (Linux 3.4) (92%), Linux 3.12 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h39m59s, deviation: 2h53m12s, median: 0s
| smb2-time: 
|   date: 2022-10-01T01:02:53
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2022-09-30T20:02:53-05:00

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   53.21 ms 10.9.0.1
2   53.38 ms 10.10.62.253

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Oct  1 02:02:55 2022 -- 1 IP address (1 host up) scanned in 17.78 seconds
```