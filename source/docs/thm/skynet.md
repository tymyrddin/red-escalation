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

### Investigating SMB

    # smbclient -L 10.10.62.253
    Password for [WORKGROUP\root]:
    
        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        anonymous       Disk      Skynet Anonymous Share
        milesdyson      Disk      Miles Dyson Personal Share
        IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
    Reconnecting with SMB1 for workgroup listing.
    
        Server               Comment
        ---------            -------
    
        Workgroup            Master
        ---------            -------
        WORKGROUP            SKYNET

Aha, there's an anonymous share. Connect:

    # smbclient //10.10.62.253/anonymous
    Password for [WORKGROUP\root]:
    Try "help" to get a list of possible commands.
    smb: \>  

Explore the anonymous share:

    smb: \> dir
      .                                   D        0  Thu Nov 26 16:04:00 2020
      ..                                  D        0  Tue Sep 17 08:20:17 2019
      attention.txt                       N      163  Wed Sep 18 04:04:59 2019
      logs                                D        0  Wed Sep 18 05:42:16 2019
    
            9204224 blocks of size 1024. 5831512 blocks available
    smb: \> get attention.txt
    getting file \attention.txt of size 163 as attention.txt (1.0 KiloBytes/sec) (average 1.0 KiloBytes/sec)
    smb: \> cd logs
    smb: \logs\> dir
      .                                   D        0  Wed Sep 18 05:42:16 2019
      ..                                  D        0  Thu Nov 26 16:04:00 2020
      log2.txt                            N        0  Wed Sep 18 05:42:13 2019
      log1.txt                            N      471  Wed Sep 18 05:41:59 2019
      log3.txt                            N        0  Wed Sep 18 05:42:16 2019
    
            9204224 blocks of size 1024. 5831512 blocks available
    smb: \logs\> mget *
    Get file log2.txt? y
    getting file \logs\log2.txt of size 0 as log2.txt (0.0 KiloBytes/sec) (average 0.6 KiloBytes/sec)
    Get file log1.txt? y
    getting file \logs\log1.txt of size 471 as log1.txt (2.8 KiloBytes/sec) (average 1.4 KiloBytes/sec)
    Get file log3.txt? y
    getting file \logs\log3.txt of size 0 as log3.txt (0.0 KiloBytes/sec) (average 1.1 KiloBytes/sec)
    smb: \logs\> exit

Find hidden files or directories using gobuster:

    # gobuster dir -u http://10.10.62.253/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt -t 50
    ===============================================================
    Gobuster v3.1.0
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.62.253/
    [+] Method:                  GET
    [+] Threads:                 50
    [+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.1.0
    [+] Extensions:              php,html,txt
    [+] Timeout:                 10s
    ===============================================================
    2022/10/01 02:21:00 Starting gobuster in directory enumeration mode
    ===============================================================
    /index.html           (Status: 200) [Size: 523]
    /admin                (Status: 301) [Size: 312] [--> http://10.10.62.253/admin/]
    /css                  (Status: 301) [Size: 310] [--> http://10.10.62.253/css/]  
    /js                   (Status: 301) [Size: 309] [--> http://10.10.62.253/js/]   
    /config               (Status: 301) [Size: 313] [--> http://10.10.62.253/config/]
    /ai                   (Status: 301) [Size: 309] [--> http://10.10.62.253/ai/]    
    /squirrelmail         (Status: 301) [Size: 319] [--> http://10.10.62.253/squirrelmail/]
    /server-status        (Status: 403) [Size: 277]                                        
                                                                                           
    ===============================================================
    2022/10/01 02:33:34 Finished
    ===============================================================

A squirrelmail entry. 

### Brute-forcing squirrelmail