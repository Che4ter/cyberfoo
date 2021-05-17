---
title: "Cheat Sheet"
description: "Cheatsheet of usefull Event Logs"
lead: ""
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu: 
  dfir:
    parent: "evtx"
weight: 20
toc: true
---

The following is a cheat sheet of useful Windows Event logs.

## 4624: An account was successfully logged on 
- **Log Source**: Client - This event is generated on the computer that was accessed.
- **Policy**: Logon/Logoff audit
- **Type**: Success
- **Links**:
    - [Microsoft Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
    - [Ultimate Windows Security](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4624)
    - [ManageEngine](https://www.manageengine.com/products/active-directory-audit/kb/windows-security-log-event-id-4624.html)
- **Relevant Fields**:
  - Subject: _(Who is doing the logon)_
    - Account Name
    - Account Domain
  - Logon Type[^1]
  - New Logon:
    - Account Name
    - Account Domain
    - Logon ID
  - Network Information: _(Values can be blank, most of the time, this means local login)_
    - Workstation Name
    - Source Network Address
- **Known Limitations**: N/A
- **Additional Recommendations**: N/A

## 4625: An account failed to log on
- **Log Source**: Client - This event is generated on the computer from where the logon attempt was made.
- **Policy**: Logon/Logoff audit
- **Type**: Failure
- **Links**: 
    - [Microsoft Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625) 
    - [Ultimate Windows Security](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4625) 
    - [ManageEngine](https://www.manageengine.com/products/active-directory-audit/kb/windows-security-log-event-id-4625.html) 
- **Relevant Fields**:
  - Subject: _(Who is doing the logon)_
    - Account Name
    - Account Domain
  - Logon Type[^1]
  - Account For Which Logon Failed:
    - Account Name
    - Account Domain
    - Logon ID
  - Network Information: (Values can be blank, most of the time, this means local login)_
    - Workstation Name
    - Source Network Address
- **Relevant Failure Codes**: 
    - 0xC0000064 _(User logon with misspelled or bad user account)_ 
    - 0xC000006A _(User logon with misspelled or bad password)_
    - 0XC000006D _(The cause is either a bad username or authentication information)_
- **Known Limitations**: Does not cover all type of logins
- **Additional Recommendations**: Even if Client logs are not available, still collect the log on the domain controller

## 4648: A logon was attempted using explicit credentials.
This event is generated when a process attempts an account logon by explicitly specifying that account’s credentials. Regardless of whether the attempt was successful or not.. This most commonly occurs in batch-type configurations such as scheduled tasks, or when using the “RUNAS” command.
- **Log Source**: Client - This event is generated on the computer from where the logon attempt was made.
- **Policy**: Logon/Logoff audit - Logon
- **Type**: Success
- **Links**: 
    - [Microsoft Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4648) 
    - [Ultimate Windows Security](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4648) 
- **Relevant Fields**:
  - Subject: _(Who is doing the logon)_
    - Account Name
    - Account Domain
  - Account For Which Logon Failed:
    - Account Name
    - Account Domain
    - Logon ID
  - Target Server
    - Target Server Name _(For example when using SharePoint, blank or localhost means local computer)_
  - Network Information: 
    - Network Address _(Source of the logon request, most of the time blank, filled out when using rdp)_
- **Known Limitations**: N/A
- **Additional Recommendations**: Hugh amount of these can indicating brute force attempts.

## 4672: Special privileges assigned to new logon
This event is generated when a logon for an account with sensitive privileges occurs. 
- **Log Source**: DC, Client
- **Policy**: Logon/Logoff audit -> Special Logon
- **Type**: Success
- **Links**: 
    - [Microsoft Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
    - [Ultimate Windows Security](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4672)
- **Relevant Fields**:
  - Subject: _(Who is doing the logon)_
    - Account Name
    - Account Domain
    - Logon ID
  - Privileges
- **Known Limitations**: N/A
- **Additional Recommendations**: N/A
  
## 4768: A Kerberos authentication ticket (TGT) was requested
Issued when a client requests a new TGT. Helps tracking initial logins with kerberos.
- **Log Source**: Domain Controller
- **Policy**: Account Logon -> Kerberos Service Ticket Operations
- **Type**: Success, Failure
- **Links**:
    - [Microsoft Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768)
    - [Ultimate Windows Security](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4768)
    - [ManageEngine](https://www.manageengine.com/products/active-directory-audit/kb/windows-security-log-event-id-4768.html)
- **Relevant Fields**:
  - Account Information: 
    - Account Name _(Computer account name ends with a $)_
    - Supplied Real Name _(Domain of the account)_
  - Network Information:
    - Client Address
  - Additional Information
    - Result Code _(0x0 = Success, else failure)_
    - Ticket Encryption Type
- **Success Status Code**:
  - 0x0
- **Relevant Failure Codes**: 
  - 0x6 _(Client not found in Kerberos database - The username doesn’t exist.)_ 
  - 0x12 _(Client’s credentials have been revoked - For example: account disabled, expired, or locked out.)_
  - 0x18 _(Pre-authentication information was invalid - The wrong password was provided.)_
- **Known Limitations**: N/A
- **Additional Recommendations**: 
  - This also logs request by computer accounts (account name ending with a $).
  - For Anomaly Based "Machine Learning" further fields like Encryption or Pre-Authentication Type could be useful.

## 4769: A Kerberos service ticket was requested
Issued when a client requests a new TGS. Allows tracking of service usage.
- **Log Source**: Domain Controller
- **Policy**: Account Logon -> Kerberos Authentication Service
- **Type**: Success, Failure
- **Links**:
    - [Microsoft Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4769)
    - [Ultimate Windows Security](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4769)
    - [ManageEngine](https://www.manageengine.com/products/active-directory-audit/kb/windows-security-log-event-id-4769.html)
- **Relevant Fields**:
  - Account Information: 
    - Account Name _(Computer account name ends with a $)_
    - Account Domain _(Domain of the account)_
  - Service Information
    - Service Name
  - Network Information:
    - Client Address
  - Additional Information
    - Result Code _(0x0 = Success, else failure)
    - Ticket Encryption Type
- **Success Status Code**:
  - 0x0
- **Relevant Failure Codes**: 
  - 0x20: TGS expired
- **Known Limitations**: N/A
- **Additional Recommendations**: 
  - This also logs request by computer accounts (account name ending with a $).
  - For Anomaly Based "Machine Learning" further options like Encryption Type could be useful.

## 4770: A Kerberos service ticket was renewed
Issued when a client renews a TGS
- **Log Source**: Domain Controller
- **Policy**: Account Logon -> Kerberos Service Ticket Operations
- **Type**: Success
- **Links**:
    - [Microsoft Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4770)
    - [Ultimate Windows Security](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4770)
- **Relevant Fields**:
  - Account Information: 
    - Account Name _ _(Computer account name ends with a $)_
    - Supplied Realm _(Domain of the account)_
  - Service Information
    - Service Name
  - Network Information:
    - Client Address 
  - Additional Information
    - Ticket Encryption Type
- **Known Limitations**: N/A
- **Additional Recommendations**: 
  - -Not sure if really needed

## 4771: Kerberos pre-authentication failed.
Issued when the username or password used for the TGT request is invalid
- **Log Source**: Domain Controller
- **Policy**: Account Logon -> Kerberos Authentication Service
- **Type**: Failure
- **Links**:
    - [Microsoft Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4771)
    - [Ultimate Windows Security](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4771)
    - [ManageEngine](https://www.manageengine.com/products/active-directory-audit/kb/windows-security-log-event-id-4771.html)
- **Relevant Fields**:
  - Account Information: 
    - Security ID
    - Account Name _(Computer account name ends with a $)_
  - Network Information  _(Values can be blank, most of the time, this means local login)_:
    - Client Address
  - Additional Information
    - Result Code 
    - Ticket Encryption Type
- **Relevant Result(Failure) Codes**: 
  - 0x18 _(Pre-authentication information was invalid - The wrong password was provided.)_
- **Known Limitations**: N/A
- **Additional Recommendations**: N/A

## 4776: The computer attempted to validate the credentials for an account 
This event generates every time that a credential validation occurs using NTLM authentication.
- **Log Source**: Domain Controller, Client
- **Policy**: Account Logon -> Credential Validation
- **Type**: Success, Failure
- **Links**:
    - [Microsoft Documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4776)
    - [Ultimate Windows Security](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4776)
    - [ManageEngine](https://www.manageengine.com/products/active-directory-audit/kb/windows-security-log-event-id-4776.html)
- **Relevant Fields**:
  - Logon Account
  - Source Workstation
  - Error Code
- **Success Code**: 0x0
- **Relevant Result(Failure) Codes**: 
  - C0000064 _(user name does not exist)_
  - C000006A _(user name is correct but the password is wrong)_
- **Known Limitations**: N/A
- **Additional Recommendations**: Old name was _The **domain** controller attempted to validate the credentials for an account_

## Footnotes
[^1]: [Microsoft - Logon Types](https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/reference-tools-logon-types)