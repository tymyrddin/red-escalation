# Game Zone


| [![Jenkins](../../_static/images/suit.png)](https://tryhackme.com/room/gamezone) |
|:--:|
| [https://tryhackme.com/room/gamezone](https://tryhackme.com/room/gamezone) |

## Obtain access via SQLi 

### Exploration

| ![Game Zone](../../_static/images/suit.png)
|:--:|
| Welcome page clues |

A [reverse image search](https://www.reverseimagesearch.com/) gave the name of the agent (47).

### Scanning

First run a simple port scan (without Ping)

	nmap -Pn -p- <IP address target machine> -oN portscan

Run an `-A` scan on the open ports:

	nmap -Pn -T4 -A -p,,, <IP address target machine> -oN servicescan

## Using SQLMap 

## Cracking password with John

## Exposing services with reverse SSH tunnels 

## Privilege escalation with Metasploit 

