# Types of shells

Shells are used for interfacing with a Command Line environment (CLI) like the common `bash` or `sh` programs in Linux, and the `cmd.exe` and `Powershell` on Windows. When targeting remote systems it is sometimes possible to force an application running on the server (such as a webserver, for example) to execute arbitrary code. When this happens, we want to use this initial access to obtain a shell running on the target.

## Reverse shell

Reverse shells are when the target is forced to execute code that connects back to the attack computer. This requires setting up a listener which would be used to receive the connection on the attack machine. Reverse shells are a good way to bypass firewall rules that may prevent you from connecting to arbitrary ports on the target.

1. Start a listener on attack machine
2. Start a reverse shell on target machine

## Bind shell

Bind shells are when the code executed on the target is used to start a listener attached to a shell directly on the target. This would then be open up a port to receive on that you can connect to, and obtain remote code execution. This has the advantage of not requiring any configuration on the attack network, but may be prevented by firewalls protecting the target.

1. Start a listener on target machine
2. Connect to listener on target from attack machine



