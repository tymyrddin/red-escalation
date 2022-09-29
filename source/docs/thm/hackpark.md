# HackPark

| [![Jenkins](../../_static/images/it.png)](https://tryhackme.com/room/hackpark) |
|:--:|
| [https://tryhackme.com/room/hackpark](https://tryhackme.com/room/hackpark) |

## Brute-force login with Hydra

### Exploring

A short exploration of the site gave me a username and a login page. 

| ![Welcome to hackpark](../../_static/images/it.png)
|:--:|
| Welcome page clues |

A [reverse image search](https://www.reverseimagesearch.com/) gave the name of the clown.

### Scanning

First run a simple port scan (without Ping)

	nmap -Pn -p- <IP address target machine> -oN portscan

Run an `-A` scan on the open ports:

	nmap -Pn -T4 -A -p,,, <IP address target machine> -oN servicescan

## Compromise the machine 

## Privilege escalation

## Privilege escalation without Metasploit 
