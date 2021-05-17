---
title: "AD Test Lab"
description: "Active Directory Test Lab"
lead: ""
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu: 
  dfir:
    parent: "guides"
weight: 100
toc: true
---

This Guide helps you to setup a basic lab to play around with a active directory, windows event logs and so on.

## Inspiration / Tutorials
The following guide is based on a variety of sources:
- [Lab Building Guide: Virtual Active Directory](https://medium.com/@vartaisecurity/lab-building-guide-virtual-active-directory-5f0d0c8eb907)
- [Building an Effective Active Directory Lab Environment for Testing ](https://adsecurity.org/?p=2653)
- [Creating Active Directory Labs for Blue and Red Teams](https://sec-consult.com/blog/detail/creating-active-directory-labs-for-blue-and-red-teams/)
- [Github Infose Link Collection](https://github.com/rmusser01/Infosec_Reference/blob/master/Draft/Building_A_Lab.md)
- [Threat Hunting Lab (Part II) : Sending PfSense Netflow data to Elastic Stack](https://hilo21.com/th/threat-hunting-lab-part-ii-sending-pfsense-netflow-data-to-elastic-stack/)
- [Intercepting HTTPS Traffic Using the Squid Proxy Service in pfSense](https://turbofuture.com/internet/Intercepting-HTTPS-Traffic-Using-the-Squid-Proxy-in-pfSense)
- [WEF-Subscriptions](https://github.com/palantir/windows-event-forwarding/tree/master/wef-subscriptions)

### Future Ideas / Resources
- [Managing Windows Server with Puppet Part 7: Installing Active Directory](https://petri.com/managing-windows-server-with-puppet-part-7-installing-active-directory)
- [BadBlood Script](https://github.com/davidprowe/BadBlood)
- [Active Directory Kill Chain Attack & Defense](https://github.com/infosecn1nja/AD-Attack-Defense)
- [ADSecurity Blog](https://adsecurity.org/)
- [Azure AD - Attack and Defense Playbook](https://github.com/Cloud-Architekt/AzureAD-Attack-Defense)

## Eval images
If you don't have a msdn subscription, you can use eval images from Microsoft[^1]. They are valid for 180 days and the period can be extended for 30 or 90 days.

**Windows 7:**
It looks like Windows 7 is not available as Eval version any more, however I found the following working link in the technet forum[^2]:

**IE/Edge Testing images**
Beside of the eval images, there are also the IE/Edge Virtual Machines available[^3].

## Lab Overview
The Lab guide covers the following setup:

- Root Domain: umbrella.corp
- Child Domain: racoon.umbrella.corp
- 1 domain controller (root DC — umbrella.corp): DC01,
- 1 domain controller (child DC — racoon.umbrella.corp): DC02
- 1 server (parent — umbrella.corp): APP01
- 1 workstation (parent — umbrella.corp): 
- 1 workstation (child — racoon.umbrella.corp)

***DC01*** <br>
Functions as Primary DC in Forest umbrella.corp

_Details_
- CPU: 2 Cores
- RAM: 4 gb
- Storage: 40 GB Thin
- OS: Windows Server 2019
- IP: 10.2.1.101, Gateway: 10.2.1.1
- Domain name: umbrella.corp
- NETBIOS: UMBRELLA

***DC02*** <br>
Functions as Child DC in Forest umbrella.corp

_Details_
- CPU: 2 Cores
- RAM: 4 gb
- Storage: 40 GB Thin
- OS: Windows Server 2016
- IP: 10.2.1.101, Gateway: 10.2.1.1, DNS: 10.2.1.101
- Domain name: raccoon.umbrella.corp
- NETBIOS: RACCOON

***APP01*** <br>
Windows Server 2012 - Application Server

_Details_
- CPU: 2 Cores
- RAM: 4 gb
- Storage: 40 GB Thin
- OS: Windows Server 12
- Domain: umbrella.corp
- IP: 10.2.1.150, Gateway: 10.2.1.1, DNS: 10.2.1.101

***WIN10-01*** <br>
Windows 10 Client in Forrest umbrella.corp
- CPU: 2 Cores
- RAM: 4 gb
- Storage: 40 GB Thin
- OS: Windows 10
- Domain: umbrella.corp
- IP: DHCP, DNS: 10.2.1.101

***Win7-01*** <br>
Windows 7 Client
_(Currently on Datastore 1)_ 
- CPU: 2 Cores
- RAM: 4 gb
- Storage: 40 GB Thin
- OS: Windows 7
- Hostname: WIN7-01
- Domain: raccoon.umbrella.corp

### Topology
The routing is done with a pfsense.

**WIN Client Network**
- 10.1.90.x
- DHCP: 10.1.90.1.2-10.90.1.100

**WIN Server Network**
- 10.2.1.x
- DHCP: 10.2.1.1-10.2.1.100

**DMZ / linux server network**
- 10.5.2.x
- DHCP: 10.5.2.2-10.5.2.100

**WAN Network**
- 10.22.22.22

## Setup DC01
1. Language: en
2. Operating System: Windows Server 2019 Standard Evaluation (Desktop Experience)
3. Custom: Install Windows only (advanced)
4. Select Drive 0
5. Set Administrator Password
6. Install VMTools and updates
7. Change Computer Name to _DC01_
8. Configure Network Adapter
    - Set IP, Subnetmask, Gateway and DNS
9. Server Manager -> 1 Configure this local server -> Add roles and features
    - Role based installation
    - DC01
    - Active Directory Domain Services 
    - Restart the destination server automatically if required
10. Server Manager -> Wait for yellow warning sign next to flag in top bar
    - Promote this server to a domain controller
    - Add a new forest: _umbrella.corp_
    - Set a DSRM password
    - Install
11. Validate in Powershell
    - Get-ADDomain
12. Setup dns reverse lookup zone:
    - DNS Manager -> Reverse Lookup Zone -> right click "New Zone"
    - Check Primary Zone and Store the zone in Active Directory
    - Type Network ID: 10.2.1, Select Reverse lookup zone name
    - Right click new zone -> New Pointer (PTR) -> Host IP: 10.2.1.101, Host name: dc01.umbrella.corp
    - repeat zone for 10.90.1

### Add Accounts
- Domain Admin:
    1. Server Manager -> Active Directory Users and Computers
    2. umbrella.corp -> Users -> Right Click -> New User
    3. User logon name: redqueen
    4. Set Password, Uncheck _User must change password at next logon_, Check _Password never expires_
    5. Select User redqueen -> Right Click -> Properties -> Member Of
        add to Administrators, Domain Admins, Enterprise Admins and Schema Admins

- Domain User:
    1. Server Manager -> Active Directory Users and Computers
    2. umbrella.corp -> Users -> Right Click -> New User
    3. User logon name: charles

## Setup Child DC02
1. Language: en
2. Operating System: Windows Server 2016 Standard Evaluation (Desktop Experience)
3. Custom: Install Windows only (advanced)
4. Select Drive 0
5. Set Administrator Password
6. Install VMTools and updates
7. Change Computer Name to _DC01_
8. Configure Network Adapter
    - Set IP, Subnetmask, Gateway and DNS
9. Server Manager -> 1 Configure this local server -> Add roles and features
    - Role based installation
    - DC01
    - Active Directory Domain Services 
    - Restart the destination server automatically if required
10. Server Manager -> Wait for yellow warning sign next to flag in top bar
    - Promote this server to a domain controller
    - Add a new domain to an existing forest: 
        - Select domain type: Child Domain
        - Parent domain name: umbrella.corp -> Select
        - Use Umbrella\Administrator account
        - New Domain name: raccoon
    - Set a DSRM password
    - Install
11. Validate in Powershell
    - Get-ADDomain
    - Get-ADTrust -Filter *

### Add Accounts
- Domain Admin:
    1. Server Manager -> Active Directory Users and Computers
    2. umbrella.corp -> Users -> Right Click -> New User
    3. User logon name: timothy
    4. Set Password, Uncheck _User must change password at next logon_, Check _Password never expires_
    5. Select User timothy -> Right Click -> Properties -> Member Of
        add to Administrators, Domain Admins

## Setup Application Server APP01
1. Language: en
2. Operating System: Windows Server 2012 Standard Evaluation (Desktop Experience)
3. Custom: Install Windows only (advanced)
4. Select Drive 0
5. Set Administrator Password
6. Install VMTools and updates
7. Change Computer Name to _APP01_
8. Configure Network Adapter
    - Set IP, Subnetmask, Gateway and DNS

## Client Setup
We are currently using default windows settings.

### WIN7-01
After fresh install, the following steps are required:
1. Install [.NET 4](https://www.microsoft.com/en-US/download/details.aspx?id=17718)
2. Install SP1/SP2/Update, because it's not available anymore on the Windows website, you can try use the version from [Chip](https://www.chip.de/downloads/CHIP-Windows-7-Update-Pack-64-Bit_69083447.html)

### JOIN Clients to Domain
1. Set DNS server to DC IP: 10.2.1.101
2. Computer name, domain, and workgroup -> Member of Domain: umbrella.corp

## PFSense Settings

### AD Domain Resoving
1. Services -> DNS Resolver -> General Settings -> Domain Overrides -> Add
    - Domain: umbrella.corp
    - IP Address: 10.2.1.101
2. Repeat for other domains
    - Domain: raccoon.umbrella.corp
    - IP Address: 10.2.1.102

### Squid Proxy Setup
1. System -> Package Manager -> Available Packages -> _squid_ and _lightsquid_
2. Services -> Squid Proxy Server -> Local Cache
    - Hard Disk Cache Size: 3000
    - Memory Cache Size: 2048
    - Maximum Object Size in RAM: 40000
3. System -> Certificate Manager -> CAs -> add
    - Descriptive Name: pfsense CA
    - Country Code: US
    - State: Arklay County
    - City: Raccoon City
    - Organization: Umbrella Corporation
4. Export CA
5. ON DC01
    - Group Policy Management -> Forest umbrella.corp -> 
    Domains -> 
    umbrella.corp ->
    Group Policy Objects -> 
    Right click -> New 
    - Name: pfSense Certificate
    - Right click on pfSense Certificate -> Edit
    - Computer Configuration -> 
    Policies -> 
    Windows Settings -> 
    Security Settings -> 
    Public Key Policies -> 
    Trusted Root Certification Authorities -> 
    Right Click -> Import
    -  Select pfsense CA File
    Right Click on Domains -> umbrella.corp -> Link an Existing GPO
    - select pfSense Certificate
     Right Click on Domains -> umbrella.corp -> Domain Controller -> Link an Existing GPO
    - select pfSense Certificate
    - to immediately apply issue gpupdate /force on a client
    - Copy policy to raccoon.umbrella.corp and repeat linking steps
6. Services -> Squid Proxy Server -> General
    - Enable Squid Proxy: On
    - Proxy Interface(s): WINCLIENT, DMZ, WINSERVER,loopback
    - Outgoing Network Interface: WAN
    - Transparent HTTP Proxy: on
    - Transparent Proxy Interface(s): WINCLIENT, DMZ, WINSERVER
    - Bypass Proxy for these Source IPs: 10.22.22.22
    - Bypass Proxy for these Destination IPs: 10.22.22.22
    - Enable Access Login: on
    - HTTPS/SSL Interception: on
    - SSL Intercept Interfaces: WINCLIENT,DMZ,WINSERVER
    - CA: pfsense CA

## Windows Event Logs

### Setup Event Event Collecting
***Warning: This setup is incomplete***

On DC1:
1. In Powershell: Enable-PSRemoting
    -> On Client check <br>
    ``` Invoke-Command -ComputerName <COLLECTORHOSTNAME> -ScriptBlock {1} ``` <br>
    -> if returns 1 = success
2. Open Event Viewer -> Click _Subscriptions_ -> Yes
3. In CMD: 
    ```console
    wevtutil gl security
    ```
    -> copy value of channelAccess
4. Open _Group Policy Management_ -> umbrella.corp -> Domains -> umbrella.corp -> right click -> Create a GPO in this domain, and Link it here: _EventForwarding_
5. Edit GPO 
    -> Computer Configuration 
    -> Policies 
    -> Administrative Templates 
    -> Windows Components 
    -> Event Forwarding 
    -> Configure target subscription manager
6. Set SubscriptionManagers to: Server=http://dc01.umbrella.corp:5985/wsman/SubscriptionManager/WEC,Refresh=60
7. Click on
    -> Computer Configuration 
    -> Policies 
    -> Administrative Templates 
    -> Windows Components 
    -> Event Log Service
    -> Security
    -> Configure log access
    insert channelAccess value from previous step
8. Click on
    -> Computer Configuration 
    -> Policies 
    -> Windows Settings
    -> System Services
    -> Windows Remote Management -> Automatic
9. Not sure what fixed the problem but on server 2019 try out 
    ```console
    netsh http add urlacl url=https://+:5986/wsman/ sddl=D:(A;;GX;;;S-1-5-80-569256582-2953403351-2909559716-1301513147-412116970)(A;;GX;;;S-1-5-80-4059739203-877974739-1245631912-527174227-2996563517)
    ``` 
10. Event Viewer -> Subscription -> Create Subscription
    - Subscription name: Security
    - Source computer initiated
        -> Add Domain Computers: WIN10-01, APP01
    - Select Events: Security
    - Advanced -> Minimize Latency
11. ON DC1 run:  ``` wecutil ss Security /l:en-US ```


 [^1]: [Microsoft Eval Center](https://www.microsoft.com/de-de/evalcenter/evaluate-windows-server-2019)
 [^2]: [Windows 7 Eval Image](http://care.dlservice.microsoft.com/dl/download/evalx/win7/x64/EN/7600.16385.090713-1255_x64fre_enterprise_en-us_EVAL_Eval_Enterprise-GRMCENXEVAL_EN_DVD.iso)
 [^3]: [Windows Browser Virtual Machine Images](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)