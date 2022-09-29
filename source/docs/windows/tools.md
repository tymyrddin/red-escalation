# Windows escalation tools

* [WinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) enumerates a target system to uncover privilege escalation paths. You can also download the precompiled 
executable or a `.bat` script. The output can be lengthy and sometimes difficult to read. It is recommended to always 
redirect the output to a file.
* [PrivescCheck](https://github.com/itm4n/PrivescCheck) is a PowerShell script that searches for common privilege escalations on a target system. 
It is an alternative to WinPEAS without having to execute a binary file. It may be necessary to bypass execution 
policy restrictions with the `Set-ExecutionPolicy` cmdlet.
* [Windows Exploit Suggester - Next Generation (WES-NG)](https://github.com/bitsadmin/wesng) will run on the attack 
machine, making way less noise. You run the `systeminfo` command on the target system, directing the output to a 
`.txt` file that you will need to move to your attacking machine. Before using it, update the database with 
`# python -m wes.py --update`, then use it on the downloaded `.txt` file: `# wes.py systeminfo.txt`