---
title: "Tests: User Tracking - Unsuccessful Logins"
description: "Track successful user logins with windows event logs"
lead: ""
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu: 
  dfir:
    parent: "evtx"
weight: 40
toc: true
---

The following tests were carried out in the test lab, the setup is documented on page [AD Test Lab](/dfir/guides/ad_lab/). The logs were forwarded to an elastic search instance. The log details are copied from the kibana msg attribute.

<div class="alert alert-warning d-flex" role="alert">
  <div class="flex-shrink-1 alert-icon">👉 </div>
    <div class="w-100">While I was careful during the test execution and gathering of all the result data, human errors always can occur . </div>
</div>

## General Test Overview
- Source: Win10-01 (10.1.90.4, Windows 10)
- Destination: APP01 (10.2.1.150, Windows Server 2012)
- DC: DC01 (10.2.1.101, Windows Server 2019)
- Domain: umbrella.corp
- Domain User: umbrella\charles
- Domain Admin User: umbrella\redqueen

## Result Matrix

| Test Scenarios                           | Event IDs:                                       | 4625        | 4648   | 4768 | 4771 | 4776            |
|------------------------------------------|--------------------------------------------------|-------------|--------|------|------|-----------------|
| Local Login with Invalid Credentials     | Unprivileged Domain User                         | Source      |        |      | DC   |                 |
|                                          | Domain Administrator                             | Source      |        |      | DC   |                 |
|                                          | Local User                                       | Source      |        |      |      | Source          |
|                                          | Non-existing Domain User                         | Source      |        | DC   |      |                 |
|                                          | Non-existing Local User                          | Source      |        |      |      | Source          |
| RDP Login with Invalid Credentials       | Unprivileged Domain User with NLA                |             |        |      | DC   |                 |
|                                          | Unprivileged Domain User with NLA over NTLM      | Destination | Source |      |      | DC              |
|                                          | Unprivileged Local User with NLA                 | Destination | Source |      |      | Destination     |
|                                          | Non-existing Domain User with NLA                | Destination | Source | DC   |      | DC              |
|                                          | Non-existing Local User with NLA                 | Destination | Source |      |      | Destination     |
|                                          | Unprivileged Domain User without NLA             | Destination |        |      | DC   |                 |
|                                          | Local User without NLA                           | Destination |        |      |      | Destination     |
|                                          | Non-existing Domain User without NLA             | Destination |        | DC   |      |                 |
|                                          | Non-existing Local User without NLA              | Destination |        |      |      | Destination     |
| Local Login with Disabled Account        | Unprivileged Domain User                         | Source      |        | DC   |      |                 |
|                                          | Unprivileged Domain User - NTLM Enforced         | Source      |        |      |      | DC              |
| RDP Login with Disabled Account          | Unprivileged Domain User and NLA                 |             |        | DC   |      |                 |
|                                          | Unprivileged Domain User without NLA             | Destination |        | DC   |      |                 |
|                                          | Unprivileged Domain User and NLA - NTLM Enforced | Destination | Source |      |      | DC, Destination |
| Login with Expired Account               | Unprivileged Domain User                         | Source      |        | DC   |      |                 |
|                                          | Unprivileged Domain User - NTLM Enforced         | Source      |        |      |      | DC              |
| RDP Login with Expired Account           | Unprivileged Domain User and NLA                 | Destination | Source | DC   |      | DC              |
|                                          | Unprivileged Domain User without NLA             | Destination |        | DC   |      |                 |
|                                          | Unprivileged Domain User and NLA - NTLM Enforced | Destination | Source |      |      | DC              |
| SMB Share Mount with Invalid Credentials | Unprivileged Domain User                         |             |        |      | DC   |                 |
|                                          | Local User                                       | Destination | Source |      |      | Destination     |
|                                          | Non-existing Domain User                         | Destination | Source | DC   |      | DC              |
|                                          | Non-existing Local User                          | Destination | Source |      |      | Destination     |
| Kerberos Brute Force with Rubeus         |                                                  |             |        |      | DC   |                 |
| LDAP Brute Force                         |                                                  | DC          | Source | DC   | DC   | DC              |
| TALON Brute Force utility                |                                                  |             |        |      | DC   |                 |

## Local Login with Invalid Credentials

### Unprivileged Domain User
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4625  
	```
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	WIN10-01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               		2

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC000006A

	Process Information:
		Caller Process ID:       	0x5dc
		Caller Process Name:     	C:\Windows\System32\svchost.exe

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	127.0.0.1
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
  - On DC:
    - 4771 
	```
	Kerberos pre-authentication failed.

	Account Information:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:            	charles

	Service Information:
		Service Name:            	krbtgt/UMBRELLA

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	49972

	Additional Information:
		Ticket Options:          	0x40810010
		Failure Code:            	0x18
		Pre-Authentication Type: 	2

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number: 	
		Certificate Thumbprint:	
	```	

### Domain Administrator
- User: umbrella\redqueen
- Events recorded:
  - On Source:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	WIN10-01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	2

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC000006A

	Process Information:
		Caller Process ID:       	0x60c
		Caller Process Name:     	C:\Windows\System32\svchost.exe

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	127.0.0.1
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
  - On DC:
    - 4771 
	```	
	Kerberos pre-authentication failed.

	Account Information:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen

	Service Information:
		Service Name:            	krbtgt/UMBRELLA

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	49733

	Additional Information:
		Ticket Options:          	0x40810010
		Failure Code:            	0x18
		Pre-Authentication Type: 	2

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number: 	
		Certificate Thumbprint:		
	```	

### Local User
- User: win10-01\Localadmin
- Events recorded:
  - On Client:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	WIN10-01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	2

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	Localadmin
		Account Domain:          	WIN10-01

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC000006A

	Process Information:
		Caller Process ID:       	0x60c
		Caller Process Name:     	C:\Windows\System32\svchost.exe

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	127.0.0.1
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0

	```	
    - 4776
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:			MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:         			Localadmin
	Source Workstation:    			WIN10-01
	Error Code:            			0xC000006A
	```	

### Non-existing Domain User
- User: umbrella\john
- Events recorded:
  - On Client:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	WIN10-01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	2

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	john
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC0000064

	Process Information:
		Caller Process ID:       	0x60c
		Caller Process Name:     	C:\Windows\System32\svchost.exe

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	127.0.0.1
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
  - On DC:
    - 4768 
	```	
	A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:            	john
		Supplied Realm Name:     	UMBRELLA
		User ID:                 	S-1-0-0

	Service Information:
		Service Name:            	krbtgt/UMBRELLA
		Service ID:              	S-1-0-0

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	51832

	Additional Information:
		Ticket Options:          	0x40810010
		Result Code:             	0x6
		Ticket Encryption Type:  	0xFFFFFFFF
		Pre-Authentication Type: 	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		
	```	
### Non-existing Local User
- User: win10-01\helpdesk
- User: umbrella\john
- Events recorded:
  - On Client:
    - 4625
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	helpdesk
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000064
	```	
    - 4776
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	WIN10-01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               		2

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	helpdesk
		Account Domain:          	WIN10-01

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC0000064

	Process Information:
		Caller Process ID:       	0x60c
		Caller Process Name:     	C:\Windows\System32\svchost.exe

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	127.0.0.1
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	

## RDP Login with Invalid Credentials
- Source: Win10-01 (10.1.90.4)
- Destination: APP01 (10.2.1.150)

### Unprivileged Domain User with Network Level Authentication (NLA)
- User: umbrella\charles
- Events recorded:
  - On DC:
    - 4771 
	```	
	Kerberos pre-authentication failed.

	Account Information:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:            	charles

	Service Information:
		Service Name:            	krbtgt/UMBRELLA

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	52091

	Additional Information:
		Ticket Options:          	0x40810010
		Failure Code:            	0x18
		Pre-Authentication Type: 	2

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number: 	
		Certificate Thumbprint:		
	```	

### Unprivileged Domain User with NLA over NTLM
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4648
	```	
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA
		Logon ID:                	0x45EBB5
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:            	charles
		Account Domain:          	UMBRELLA
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:      	APP01.umbrella.corp
		Additional Information:  	APP01.umbrella.corp

	Process Information:
		Process ID:              	0x298
		Process Name:            	C:\Windows\System32\lsass.exe

	Network Information:
		Network Address:	-
		Port:			-
	```	
  - On Destination:
    - 4625
	```	
		An account failed to log on.

	Subject:
		Security ID:             	S-1-0-0
		Account Name:            	-
		Account Domain:          	-
		Logon ID:                	0x0

	Logon Type:               		3

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC000006A

	Process Information:
		Caller Process ID:       	0x0
		Caller Process Name:     	-

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	-
		Source Port:             	-

	Detailed Authentication Information:
		Logon Process:           	NtLmSsp 
		Authentication Package:  	NTLM
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
  - On DC:
    - 4776 
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:   		MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            		charles
	Source Workstation:       		WIN10-01
	Error Code:               		0xC000006A
	```	

### Unprivileged Local User with NLA
- User: APP01\Administrator
- Events recorded:
  - On Source:
    - 4648
	```	
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA
		Logon ID:                	0xDBCAE3
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:            	Administrator
		Account Domain:          	APP01
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:      	APP01.umbrella.corp
		Additional Information:  	APP01.umbrella.corp

	Process Information:
		Process ID:              	0x284
		Process Name:            	C:\Windows\System32\lsass.exe

	Network Information:
		Network Address:         	-
		Port:                    	-
	```	
  - On Destination:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-0-0
		Account Name:            	-
		Account Domain:          	-
		Logon ID:                	0x0

	Logon Type:               		3

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	Administrator
		Account Domain:          	APP01

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC000006A

	Process Information:
		Caller Process ID:       	0x0
		Caller Process Name:     	-

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	-
		Source Port:             	-

	Detailed Authentication Information:
		Logon Process:           	NtLmSsp 
		Authentication Package:  	NTLM
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
    - 4776 
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	Administrator
	Source Workstation:       	WIN10-01
	Error Code:               	0xC000006A
	```	

### Non-existing Domain User with NLA
- User: umbrella\smith
- Events recorded:
  - On Source:
    - 4648
	```	
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:            	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:           	redqueen
		Account Domain:         	UMBRELLA
		Logon ID:               	0xDBCAE3
		Logon GUID:             	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:           	smith
		Account Domain:         	UMBRELLA
		Logon GUID:             	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:     	APP01.umbrella.corp
		Additional Information: 	APP01.umbrella.corp

	Process Information:
		Process ID:             	0x284
		Process Name:           	C:\Windows\System32\lsass.exe

	Network Information:
		Network Address:        	-
		Port:                   	-
	```	
  - On Destination:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:            	S-1-0-0
		Account Name:           	-
		Account Domain:         	-
		Logon ID:               	0x0

	Logon Type:              	3

	Account For Which Logon Failed:
		Security ID:            	S-1-0-0
		Account Name:           	smith
		Account Domain:         	UMBRELLA

	Failure Information:
		Failure Reason:         	Unknown user name or bad password.
		Status:                 	0xC000006D
		Sub Status:             	0xC0000064

	Process Information:
		Caller Process ID:      	0x0
		Caller Process Name:    	-

	Network Information:
		Workstation Name:       	WIN10-01
		Source Network Address: 	-
		Source Port:            	-

	Detailed Authentication Information:
		Logon Process:          	NtLmSsp 
		Authentication Package: 	NTLM
		Transited Services:     	-
		Package Name (NTLM only):	-
		Key Length:             	0
	```	
  - On DC:
    - 4768
	```	
	A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:           	smith
		Supplied Realm Name:    	UMBRELLA
		User ID:                	S-1-0-0

	Service Information:
		Service Name:           	krbtgt/UMBRELLA
		Service ID:             	S-1-0-0

	Network Information:
		Client Address:         	::ffff:10.1.90.4
		Client Port:            	52276

	Additional Information:
		Ticket Options:         	0x40810010
		Result Code:            	0x6
		Ticket Encryption Type: 	0xFFFFFFFF
		Pre-Authentication Type:	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		
	```	
    - 4776 
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:  	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:           	smith
	Source Workstation:      	WIN10-01
	Error Code:              	0xC0000064
	```	

### Non-existing Local User with NLA
- User: APP\hacker
- Events recorded:
  - On Source:
    - 4648
	```	
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA
		Logon ID:                	0xDBCAE3
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:            	hacker
		Account Domain:          	APP01
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:      	APP01.umbrella.corp
		Additional Information:  	APP01.umbrella.corp

	Process Information:
		Process ID:              	0x284
		Process Name:            	C:\Windows\System32\lsass.exe

	Network Information:
		Network Address:         	-
		Port:                    	-
	```	
  - On Destination:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-0-0
		Account Name:            	-
		Account Domain:          	-
		Logon ID:                	0x0

	Logon Type:               	3

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	hacker
		Account Domain:          	APP01

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC0000064

	Process Information:
		Caller Process ID:       	0x0
		Caller Process Name:     	-

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	-
		Source Port:             	-

	Detailed Authentication Information:
		Logon Process:           	NtLmSsp 
		Authentication Package:  	NTLM
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
    - 4776 
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	hacker
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000064
	```	

### Unprivileged Domain User without NLA
- User: umbrella\charles
- Events recorded:
  - On Destination:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	APP01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	10

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC000006A

	Process Information:
		Caller Process ID:       	0xad4
		Caller Process Name:     	C:\Windows\System32\winlogon.exe

	Network Information:
		Workstation Name:        	APP01
		Source Network Address:  	10.1.90.4
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
  - On DC:
    - 4771 
	```	
	Kerberos pre-authentication failed.

	Account Information:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:            	charles

	Service Information:
		Service Name:            	krbtgt/UMBRELLA

	Network Information:
		Client Address:          	::ffff:10.2.1.150
		Client Port:             	49430

	Additional Information:
		Ticket Options:          	0x40810010
		Failure Code:            	0x18
		Pre-Authentication Type: 	2

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number: 	
		Certificate Thumbprint:		
	```	

### Local User without NLA
- User: APP01\Administrator
- Events recorded:
  - On Destination:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	APP01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	10

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	Administrator
		Account Domain:          	APP01

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC000006A

	Process Information:
		Caller Process ID:       	0x73c
		Caller Process Name:     	C:\Windows\System32\winlogon.exe

	Network Information:
		Workstation Name:        	APP01
		Source Network Address:  	10.1.90.4
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
    - 4776 
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	Administrator
	Source Workstation:       	APP01
	Error Code:               	0xC000006A
	```	

### Non-existing Domain User without NLA
- User: umbrella\smith
- Events recorded:
  - On Destination:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	APP01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	10

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	smith
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC0000064

	Process Information:
		Caller Process ID:       	0x5d4
		Caller Process Name:     	C:\Windows\System32\winlogon.exe

	Network Information:
		Workstation Name:        	APP01
		Source Network Address:  	10.1.90.4
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0

	```	
  - On DC:
    - 4768
	```	
	A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:            	smith
		Supplied Realm Name:     	UMBRELLA
		User ID:                 	S-1-0-0

	Service Information:
		Service Name:            	krbtgt/UMBRELLA
		Service ID:              	S-1-0-0

	Network Information:
		Client Address:          	::ffff:10.2.1.150
		Client Port:             	49452

	Additional Information:
		Ticket Options:          	0x40810010
		Result Code:             	0x6
		Ticket Encryption Type:  	0xFFFFFFFF
		Pre-Authentication Type: 	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		
	```	

### Non-existing Local User without NLA
- User: APP\hacker
- Events recorded:
  - On Destination:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	APP01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	10

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	hacker
		Account Domain:          	APP01

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC0000064

	Process Information:
		Caller Process ID:       	0x994
		Caller Process Name:     	C:\Windows\System32\winlogon.exe

	Network Information:
		Workstation Name:        	APP01
		Source Network Address:  	10.1.90.4
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
    - 4776 
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	hacker
	Source Workstation:       	APP01
	Error Code:               	0xC0000064
	```	

## Local Login with Disabled Account

### Unprivileged Domain User
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4625
	```
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	WIN10-01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	2

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	Account currently disabled.
		Status:                  	0xC000006E
		Sub Status:              	0xC0000072

	Process Information:
		Caller Process ID:       	0x5b4
		Caller Process Name:     	C:\Windows\System32\svchost.exe

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	127.0.0.1
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:		0
	```
  - On DC:
    - 4768 
	```
	A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:            	charles
		Supplied Realm Name:     	UMBRELLA
		User ID:                 	S-1-0-0

	Service Information:
		Service Name:            	krbtgt/UMBRELLA
		Service ID:              	S-1-0-0

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	49912

	Additional Information:
		Ticket Options:          	0x40810010
		Result Code:             	0x12
		Ticket Encryption Type:  	0xFFFFFFFF
		Pre-Authentication Type: 	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		
	```

### Unprivileged Domain User - NTLM Enforced
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4625
	```
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	WIN10-01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	2

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	Account currently disabled.
		Status:                  	0xC000006E
		Sub Status:              	0xC0000072

	Process Information:
		Caller Process ID:       	0x5b4
		Caller Process Name:     	C:\Windows\System32\svchost.exe

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	127.0.0.1
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```
  - On DC:
    - 4776 
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	charles
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000072
	```

### RDP Login with Disabled Account

### Unprivileged Domain User and NLA
- User: umbrella\charles
- Source: Win10-01 (10.1.90.4)
- Destination: APP01 (10.2.1.150)
- Events recorded:
  - On DC:
    - 4768 
	```
		A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:            	charles
		Supplied Realm Name:     	UMBRELLA
		User ID:                 	S-1-0-0

	Service Information:
		Service Name:            	krbtgt/UMBRELLA
		Service ID:              	S-1-0-0

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	49992

	Additional Information:
		Ticket Options:          	0x40810010
		Result Code:             	0x12
		Ticket Encryption Type:  	0xFFFFFFFF
		Pre-Authentication Type: 	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		
	```

### Unprivileged Domain User without NLA
- User: umbrella\charles
- Source: Win10-01 (10.1.90.4)
- Destination: APP01 (10.2.1.150)
- Events recorded:
  - On Destination:
    - 4625
	```
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	APP01$
		Account Domain:		UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	10

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	umbrella

	Failure Information:
		Failure Reason:          	Account currently disabled.
		Status:                  	0xC000006E
		Sub Status:              	0xC0000072

	Process Information:
		Caller Process ID:       	0xb60
		Caller Process Name:     	C:\Windows\System32\winlogon.exe

	Network Information:
		Workstation Name:        	APP01
		Source Network Address:  	10.1.90.4
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```
  - On DC:
    - 4768 
	```
	A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:            	charles
		Supplied Realm Name:     	umbrella
		User ID:                 	S-1-0-0

	Service Information:
		Service Name:            	krbtgt/umbrella
		Service ID:              	S-1-0-0

	Network Information:
		Client Address:          	::ffff:10.2.1.150
		Client Port:             	51022

	Additional Information:
		Ticket Options:          	0x40810010
		Result Code:             	0x12
		Ticket Encryption Type:  	0xFFFFFFFF
		Pre-Authentication Type: 	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		
	```

### Unprivileged Domain User and NLA - NTLM Enforced
- User: umbrella\charles
- Source: Win10-01 (10.1.90.4)
- Destination: APP01 (10.2.1.150)
- Events recorded:
  - On Source:
    - 4648
	```
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA
		Logon ID:                	0x13D2D4
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:            	charles
		Account Domain:          	umbrella
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:      	APP01.umbrella.corp
		Additional Information:  	APP01.umbrella.corp

	Process Information:
		Process ID:              	0x298
		Process Name:            	C:\Windows\System32\lsass.exe

	Network Information:
		Network Address:         	-
		Port:                    	-
	```
  - On Destination:
    - 4625
	```
		An account failed to log on.

	Subject:
		Security ID:             	S-1-0-0
		Account Name:            	-
		Account Domain:          	-
		Logon ID:                	0x0

	Logon Type:               	3

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	umbrella

	Failure Information:
		Failure Reason:          	Account currently disabled.
		Status:                  	0xC000006E
		Sub Status:              	0xC0000072

	Process Information:
		Caller Process ID:       	0x0
		Caller Process Name:     	-

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	-
		Source Port:             	-

	Detailed Authentication Information:
		Logon Process:           	NtLmSsp 
		Authentication Package:  	NTLM
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```
    - 4776 
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	charles
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000064
	```
  - On DC:
    - 4776 
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	charles
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000072
	```

## Local Login with Expired Account

### Unprivileged Domain User
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4625
	```
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	WIN10-01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	2

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	The specified user account has expired.
		Status:                  	0xC0000193
		Sub Status:              	0xC0000193

	Process Information:
		Caller Process ID:       	0x5b4
		Caller Process Name:     	C:\Windows\System32\svchost.exe

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	127.0.0.1
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```
  - On DC:
    - 4768 
	```
	A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:            	charles
		Supplied Realm Name:     	UMBRELLA
		User ID:                 	S-1-0-0

	Service Information:
		Service Name:            	krbtgt/UMBRELLA
		Service ID:              	S-1-0-0

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	50067

	Additional Information:
		Ticket Options:          	0x40810010
		Result Code:             	0x12
		Ticket Encryption Type:  	0xFFFFFFFF
		Pre-Authentication Type: 	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		
	```

### Unprivileged Domain User - NTLM Enforced
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4625
	```
		An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	WIN10-01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	2

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	The specified user account has expired.
		Status:                  	0xC0000193
		Sub Status:              	0x0

	Process Information:
		Caller Process ID:       	0x5b4
		Caller Process Name:     	C:\Windows\System32\svchost.exe

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	127.0.0.1
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```
  - On DC:
    - 4776 
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	charles
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000193
	```

## RDP Login with Expired Account

### Unprivileged Domain User and NLA
- User: umbrella\charles
- Source: Win10-01 (10.1.90.4)
- Destination: APP01 (10.2.1.150)
- Events recorded:
  - On Source:
    - 4648
	```
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA
		Logon ID:                	0x23FC7F
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:            	charles
		Account Domain:          	umbrella
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:      	APP01.umbrella.corp
		Additional Information:  	APP01.umbrella.corp

	Process Information:
		Process ID:              	0x298
		Process Name:            	C:\Windows\System32\lsass.exe

	Network Information:
		Network Address:         	-
		Port:                    	-
	```
  - On Destination:
    - 4625
	```
	An account failed to log on.

	Subject:
		Security ID:             	S-1-0-0
		Account Name:            	-
		Account Domain:          	-
		Logon ID:                	0x0

	Logon Type:               	3

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	umbrella

	Failure Information:
		Failure Reason:          	The specified user account has expired.
		Status:                  	0xC0000193
		Sub Status:              	0x0

	Process Information:
		Caller Process ID:       	0x0
		Caller Process Name:     	-

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	-
		Source Port:             	-

	Detailed Authentication Information:
		Logon Process:           	NtLmSsp 
		Authentication Package:  	NTLM
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0

	```
  - On DC:
    - 4768 
	```
	A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:            	charles
		Supplied Realm Name:     	umbrella
		User ID:                 	S-1-0-0

	Service Information:
		Service Name:            	krbtgt/umbrella
		Service ID:              	S-1-0-0

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	50147

	Additional Information:
		Ticket Options:          	0x40810010
		Result Code:             	0x12
		Ticket Encryption Type:  	0xFFFFFFFF
		Pre-Authentication Type: 	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		
	```
    -  4776 
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	charles
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000193
	```

### Unprivileged Domain User without NLA
- User: umbrella\charles
- Source: Win10-01 (10.1.90.4)
- Destination: APP01 (10.2.1.150)
- Events recorded:
  - On Destination:
    - 4625
	```
	An account failed to log on.

	Subject:
		Security ID:             	S-1-5-18
		Account Name:            	APP01$
		Account Domain:          	UMBRELLA
		Logon ID:                	0x3E7

	Logon Type:               	10

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	umbrella

	Failure Information:
		Failure Reason:          	The specified user account has expired.
		Status:                  	0xC0000193
		Sub Status:              	0xC0000193

	Process Information:
		Caller Process ID:       	0xe8
		Caller Process Name:     	C:\Windows\System32\winlogon.exe

	Network Information:
		Workstation Name:        	APP01
		Source Network Address:  	10.1.90.4
		Source Port:             	0

	Detailed Authentication Information:
		Logon Process:           	User32 
		Authentication Package:  	Negotiate
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0

	```
  - On DC:
    - 4768 
	```
	A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:            	charles
		Supplied Realm Name:     	umbrella
		User ID:                 	S-1-0-0

	Service Information:
		Service Name:            	krbtgt/umbrella
		Service ID:              	S-1-0-0

	Network Information:
		Client Address:          	::ffff:10.2.1.150
		Client Port:             	51083

	Additional Information:
		Ticket Options:          	0x40810010
		Result Code:             	0x12
		Ticket Encryption Type:  	0xFFFFFFFF
		Pre-Authentication Type: 	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		
	```

### Unprivileged Domain User and NLA - NTLM Enforced
- User: umbrella\charles
- Source: Win10-01 (10.1.90.4)
- Destination: APP01 (10.2.1.150)
- Events recorded:
  - On Source:
    - 4648
	```
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA
		Logon ID:                	0x23FC7F
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:            	charles
		Account Domain:          	umbrella
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:      	APP01.umbrella.corp
		Additional Information:  	APP01.umbrella.corp

	Process Information:
		Process ID:              	0x298
		Process Name:            	C:\Windows\System32\lsass.exe

	Network Information:
		Network Address:         	-
		Port:                    	-
	```
  - On Destination:
    - 4625
	```
	An account failed to log on.

	Subject:
		Security ID:             	S-1-0-0
		Account Name:            	-
		Account Domain:          	-
		Logon ID:                	0x0

	Logon Type:               	3

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles
		Account Domain:          	umbrella

	Failure Information:
		Failure Reason:          	The specified user account has expired.
		Status:                  	0xC0000193
		Sub Status:              	0x0

	Process Information:
		Caller Process ID:       	0x0
		Caller Process Name:     	-

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	-
		Source Port:             	-

	Detailed Authentication Information:
		Logon Process:           	NtLmSsp 
		Authentication Package:  	NTLM
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```
    - 4776 
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	charles
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000193
	```

## SMB Share Mount/Login with Invalid Credentials
- Source: Win10-01 (10.1.90.4)
- Destination: APP01 (10.2.1.150)

### Unprivileged Domain User
- User: umbrella\charles
- Events recorded:
  - On DC:
    - 4771 
	```	
	Kerberos pre-authentication failed.

	Account Information:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:            	charles

	Service Information:
		Service Name:            	krbtgt/UMBRELLA

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	52861

	Additional Information:
		Ticket Options:          	0x40810010
		Failure Code:            	0x18
		Pre-Authentication Type: 	2

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number: 	
		Certificate Thumbprint:		
	```	

### Local User
- User: app01\Administrator
- Events recorded:
  - On Source:
    - 4648
	```	
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA
		Logon ID:                	0xDBCAE3
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:            	Administrator
		Account Domain:          	APP01
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:      	APP01.umbrella.corp
		Additional Information:  	APP01.umbrella.corp

	Process Information:
		Process ID:              	0x4
		Process Name:		

	Network Information:
		Network Address:         	10.2.1.150
		Port:                    	445
	```	
  - On Destination:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-0-0
		Account Name:            	-
		Account Domain:          	-
		Logon ID:                	0x0

	Logon Type:               	3

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	Administrator
		Account Domain:          	APP01

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC000006A

	Process Information:
		Caller Process ID:       	0x0
		Caller Process Name:     	-

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	10.1.90.4
		Source Port:             	52881

	Detailed Authentication Information:
		Logon Process:           	NtLmSsp 
		Authentication Package:  	NTLM
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
    - 4776
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	Administrator
	Source Workstation:       	WIN10-01
	Error Code:               	0xC000006A
	```	
### Non-existing Domain User
- User: umbrella\john
- Events recorded:
  - On Source:
    - 4648
	```	
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA
		Logon ID:                	0xDBCAE3
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:            	john
		Account Domain:          	UMBRELLA
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:      	APP01.umbrella.corp
		Additional Information:  	APP01.umbrella.corp

	Process Information:
		Process ID:              	0x4
		Process Name:		

	Network Information:
		Network Address:         	10.2.1.150
		Port:                    	445
	```	
  - On Destination:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-0-0
		Account Name:            	-
		Account Domain:          	-
		Logon ID:                	0x0

	Logon Type:               	3

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	john
		Account Domain:          	UMBRELLA

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC0000064

	Process Information:
		Caller Process ID:       	0x0
		Caller Process Name:     	-

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	10.1.90.4
		Source Port:             	52893

	Detailed Authentication Information:
		Logon Process:           	NtLmSsp 
		Authentication Package:  	NTLM
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
  - On DC:
    - 4768 
	```	
	A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:            	john
		Supplied Realm Name:     	UMBRELLA
		User ID:                 	S-1-0-0

	Service Information:
		Service Name:            	krbtgt/UMBRELLA
		Service ID:              	S-1-0-0

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	52894

	Additional Information:
		Ticket Options:          	0x40810010
		Result Code:             	0x6
		Ticket Encryption Type:  	0xFFFFFFFF
		Pre-Authentication Type: 	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		

	```	
    - 4776 
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	john
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000064
	```	

### Non-existing Local User
- User: APP01\hacker
- Events recorded:
  - On Source:
    - 4648
	```	
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA
		Logon ID:                	0xDBCAE3
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:            	hacker
		Account Domain:          	APP01
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:      	APP01.umbrella.corp
		Additional Information:  	APP01.umbrella.corp

	Process Information:
		Process ID:              	0x4
		Process Name:		

	Network Information:
		Network Address:	10.2.1.150
		Port:			445

	```	
  -  On Destination:
    - 4625
	```	
	An account failed to log on.

	Subject:
		Security ID:             	S-1-0-0
		Account Name:            	-
		Account Domain:          	-
		Logon ID:                	0x0

	Logon Type:               	3

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	hacker
		Account Domain:          	APP01

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC0000064

	Process Information:
		Caller Process ID:       	0x0
		Caller Process Name:     	-

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	10.1.90.4
		Source Port:             	53004

	Detailed Authentication Information:
		Logon Process:           	NtLmSsp 
		Authentication Package:  	NTLM
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```	
    - 4776 
	```	
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	hacker
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000064
	```	

## Kerberos Brute Force with Rubeus
- User: UMBRELLA\charles
- Source: Win10-01 (10.1.90.4)
- [Rubeus](https://github.com/GhostPack/Rubeus)
- Command: 
  ```
  .\Rubeus.exe brute /passwords:.\pwlist.txt /user:charles /domain:umbrella.corp
  ```
- Events recorded:
  - On DC:
    - 4771 (multiple)
	```
	Kerberos pre-authentication failed.

	Account Information:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:            	charles

	Service Information:
		Service Name:            	krbtgt/umbrella.corp

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	54670

	Additional Information:
		Ticket Options:          	0x40800010
		Failure Code:            	0x18
		Pre-Authentication Type: 	2

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number: 	
		Certificate Thumbprint:		
	```

## LDAP Brute Force
- User: UMBRELLA\charles, UMBRELLA\charles1, UMBRELLA\charles2, UMBRELLA\charles3
- Source: Win10-01 (10.1.90.4)
- [adlogin.ps](https://github.com/InfosecMatter/Minimalistic-offensive-security-tools)
- Command:
  ```
  Import-Module .\localbrute.ps1
  adlogin users.txt umbrella.corp P@ssw0rd
  ```
- Events recorded(ID: 655730 -> 655756):
  - On Source:
    - 4648 
	```
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1104
		Account Name:            	redqueen
		Account Domain:          	UMBRELLA
		Logon ID:                	0xDBCAE3
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:            	charles1
		Account Domain:          	umbrella.corp
		Logon GUID:              	{00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:      	DC01.umbrella.corp
		Additional Information:  	DC01.umbrella.corp

	Process Information:
		Process ID:              	0xbc8
		Process Name:            	C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

	Network Information:
		Network Address:         	-
		Port:                    	-
	```
  - On DC:
    - 4625
	```
	An account failed to log on.

	Subject:
		Security ID:             	S-1-0-0
		Account Name:            	-
		Account Domain:          	-
		Logon ID:                	0x0

	Logon Type:               	3

	Account For Which Logon Failed:
		Security ID:             	S-1-0-0
		Account Name:            	charles1
		Account Domain:          	umbrella.corp

	Failure Information:
		Failure Reason:          	Unknown user name or bad password.
		Status:                  	0xC000006D
		Sub Status:              	0xC0000064

	Process Information:
		Caller Process ID:       	0x0
		Caller Process Name:     	-

	Network Information:
		Workstation Name:        	WIN10-01
		Source Network Address:  	10.1.90.4
		Source Port:             	54707

	Detailed Authentication Information:
		Logon Process:           	NtLmSsp 
		Authentication Package:  	NTLM
		Transited Services:      	-
		Package Name (NTLM only):	-
		Key Length:              	0
	```
    - 4768
	```
	A Kerberos authentication ticket (TGT) was requested.

	Account Information:
		Account Name:            	charles1
		Supplied Realm Name:     	umbrella.corp
		User ID:                 	S-1-0-0

	Service Information:
		Service Name:            	krbtgt/umbrella.corp
		Service ID:              	S-1-0-0

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	54708

	Additional Information:
		Ticket Options:          	0x40810010
		Result Code:             	0x6
		Ticket Encryption Type:  	0xFFFFFFFF
		Pre-Authentication Type: 	-

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number:	
		Certificate Thumbprint:		
	```
    - 4771 (only for existing user)
	```
	Kerberos pre-authentication failed.

	Account Information:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:            	charles

	Service Information:
		Service Name:            	krbtgt/umbrella.corp

	Network Information:
		Client Address:          	::ffff:10.1.90.4
		Client Port:             	54705

	Additional Information:
		Ticket Options:          	0x40810010
		Failure Code:            	0x18
		Pre-Authentication Type: 	2

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number: 	
		Certificate Thumbprint:		
	```
    - 4776 
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:   	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:            	charles1
	Source Workstation:       	WIN10-01
	Error Code:               	0xC0000064
	```

## TALON Brute Force utility
- Source: Kali (10.1.90.5)
- User: UMBRELLA\charles
- https://github.com/optiv/Talon
- Command: 
  ```
  .\Talon -H 10.2.1.101 -D UMBRELLA.CORP -Userfile users.txt -P "random" -sleep 1
  ```
- Events recorded(ID: 658926 -> 658931):
  - On DC:
    - 4771
	```
	Kerberos pre-authentication failed.

	Account Information:
		Security ID:             	S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:            	charles

	Service Information:
		Service Name:            	krbtgt/UMBRELLA.CORP

	Network Information:
		Client Address:          	10.1.90.5
		Client Port:             	40680

	Additional Information:
		Ticket Options:          	0x10
		Failure Code:            	0x18
		Pre-Authentication Type: 	2

	Certificate Information:
		Certificate Issuer Name:		
		Certificate Serial Number: 	
		Certificate Thumbprint:		
	```