---
title: "Tests: User Tracking - Successful Logins"
description: "Track successful user logins with windows event logs"
lead: ""
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu: 
  dfir:
    parent: "evtx"
weight: 30
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
| Test Scenarios | Event IDs: | 4624 | 4648 | 4768 | 4769 | 4776 |
|-|-|-|-|-|-|-|
| Local Login | Unprivileged Domain User - First Time on new Machine | Source, DC | Source | DC | DC |  |
|  | Unprivileged Domain User - Second Time | Source, DC | Source | DC | DC |  |
|  | Domain Administrator | Source, DC | Source | DC | DC |  |
|  | Unprivileged Domain User - NTLM Enforced | Source, DC | Source |  |  | DC |
|  | Unprivileged Domain User - No connection to DC/cached credentials | Source | Source |  |  |  |
|  | Local User | Source | Source |  |  | Source |
| Run as | Domain User | Source | Source | DC | DC |  |
| RDP Login | Unprivileged Domain User with NLA | Destination, DC | Source, Destination | DC | DC |  |
|  | Unprivileged Domain User with NLA - NTLM Enforced | Destiation, DC | Destination |  |  | DC |
|  | Unprivileged Local User with NLA | Destination | Source, Destination |  |  | Destination |
|  | Unprivileged Domain User without NLA | Destination, DC | Source | DC | DC |  |
|  | Local User without NLA | Destination | Destination |  |  | Destination |
| Scenario: SMB Share Mount/Login explicit credentials | Unprivileged Domain User | Destination | Source | DC | DC |  |
|  | Local User | Destination | Source |  |  | Destination |

## Local Login 

### Unprivileged Domain User - First Time on new Machine
- User: umbrella\charles
- Events recorded:
  - On Source:  
    - 4624  
    ```text
    An account was successfully logged on.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           WIN10-01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7

    Logon Information:
      Logon Type:             2
      Restricted Admin Mode:  -
      Virtual Account:        No
      Elevated Token:         No

    Impersonation Level:      Impersonation

    New Logon:
      Security ID:            S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:           charles
      Account Domain:         UMBRELLA
      Logon ID:               0xAEE29
      Linked Logon ID:        0x0
      Network Account Name:   -
      Network Account Domain: -
      Logon GUID:             {d24d7a54-4e3d-5cfc-55ce-50d667341ac1}

    Process Information:
      Process ID:               0x5c8
      Process Name:             C:\Windows\System32\svchost.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   127.0.0.1
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4648
    ```text
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           WIN10-01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA
      Logon GUID:             {d24d7a54-4e3d-5cfc-55ce-50d667341ac1}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0x5c8
      Process Name:           C:\Windows\System32\svchost.exe

    Network Information:
      Network Address:        127.0.0.1
      Port:                   0
    ```
  - On DC:
    - 4624 (5x)
    ```text
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x13D94908
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {5bf2bd8e-e769-f7e5-752c-1acd84f484b0}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   -
      Source Port:              -

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```text
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Delegation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x13D948DE
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {5bf2bd8e-e769-f7e5-752c-1acd84f484b0}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.1.90.4
      Source Port:              49680

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```text
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x13D948B9
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {075bd382-ce58-6dfa-478c-3fcf7eed6da9}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.1.90.4
      Source Port:              49804

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```text
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Delegation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x13D9475E
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {075bd382-ce58-6dfa-478c-3fcf7eed6da9}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.1.90.4
      Source Port:              49800

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```text
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x13D94736
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {075bd382-ce58-6dfa-478c-3fcf7eed6da9}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.1.90.4
      Source Port:              49798

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4768 
    ```text
    A Kerberos authentication ticket (TGT) was requested.

    Account Information:
      Account Name:             charles
      Supplied Realm Name:      UMBRELLA
      User ID:                  S-1-5-21-908899349-1681392183-178755882-1105

    Service Information:
      Service Name:             krbtgt
      Service ID:               S-1-5-21-908899349-1681392183-178755882-502

    Network Information:
      Client Address:           ::ffff:10.1.90.4
      Client Port:              49795

    Additional Information:
      Ticket Options:           0x40810010
      Result Code:              0x0
      Ticket Encryption Type:   0x12
      Pre-Authentication Type:  2

    Certificate Information:
      Certificate Issuer Name:		
      Certificate Serial Number:	
      Certificate Thumbprint:		
    ```
    - 4769 (6x)
    ```text
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {d24d7a54-4e3d-5cfc-55ce-50d667341ac1}

    Service Information:
      Service Name:           WIN10-01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1110

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            49796

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
    ```text
      A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {d24d7a54-4e3d-5cfc-55ce-50d667341ac1}

    Service Information:
      Service Name:           DC01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1000

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            49799

    Additional Information:
      Ticket Options:         0x40800000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
    ```text
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {d24d7a54-4e3d-5cfc-55ce-50d667341ac1}

    Service Information:
      Service Name:           DC01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1000

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            49801

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
    ```text
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {d24d7a54-4e3d-5cfc-55ce-50d667341ac1}

    Service Information:
      Service Name:           krbtgt
      Service ID:             S-1-5-21-908899349-1681392183-178755882-502

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            49802

    Additional Information:
      Ticket Options:         0x60810010
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
    ```text
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {09855e20-23cc-86ee-2121-d05336e166b3}

    Service Information:
      Service Name:           DC01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1000

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            49805

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
    ```text
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {09855e20-23cc-86ee-2121-d05336e166b3}

    Service Information:
      Service Name:           DC01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1000

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            49806

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
  
### Unprivileged Domain User - Second Time
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4624
    ```text
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             WIN10-01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Information:
      Logon Type:               2
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           No

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0x3B0EFE
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {de8a936c-bdb6-000e-2f3b-24102ede5bd8}

    Process Information:
      Process ID:               0x5c8
      Process Name:             C:\Windows\System32\svchost.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   127.0.0.1
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4648
    ```text
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           WIN10-01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA
      Logon GUID:             {de8a936c-bdb6-000e-2f3b-24102ede5bd8}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0x5c8
      Process Name:           C:\Windows\System32\svchost.exe

    Network Information:
      Network Address:        127.0.0.1
      Port:                   0
    ```
  - On DC:
    - 4624 (2x)
    ```text
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x13E04762
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {3db48572-8cad-e651-a8d6-192440157dea}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.1.90.4
      Source Port:              50063

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```text
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x13E047C7
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {3db48572-8cad-e651-a8d6-192440157dea}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.1.90.4
      Source Port:              50067

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4768 
    ```text
    A Kerberos authentication ticket (TGT) was requested.

    Account Information:
      Account Name:             charles
      Supplied Realm Name:      UMBRELLA
      User ID:                  S-1-5-21-908899349-1681392183-178755882-1105

    Service Information:
      Service Name:             krbtgt
      Service ID:               S-1-5-21-908899349-1681392183-178755882-502

    Network Information:
      Client Address:           ::ffff:10.1.90.4
      Client Port:              50060

    Additional Information:
      Ticket Options:           0x40810010
      Result Code:              0x0
      Ticket Encryption Type:   0x12
      Pre-Authentication Type:  2

    Certificate Information:
      Certificate Issuer Name:		
      Certificate Serial Number:	
      Certificate Thumbprint:		
    ```
    - 4769 (2x)
    ```text
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {de8a936c-bdb6-000e-2f3b-24102ede5bd8}

    Service Information:
      Service Name:           WIN10-01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1110

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            50061

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
    ```text
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {de8a936c-bdb6-000e-2f3b-24102ede5bd8}

    Service Information:
      Service Name:           DC01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1000

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            50064

    Additional Information:
      Ticket Options:         0x40800000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```

### Domain Administrator
- User: umbrella\redqueen
- Events recorded:
  - On Source:
    - 4624 (4x)
    ```text
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             WIN10-01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Information:
      Logon Type:               11
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:             redqueen
      Account Domain:           UMBRELLA
      Logon ID:                 0x50C48
      Linked Logon ID:          0x50C68
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x57c
      Process Name:             C:\Windows\System32\svchost.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   127.0.0.1
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             WIN10-01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Information:
      Logon Type:               11
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           No

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:             redqueen
      Account Domain:           UMBRELLA
      Logon ID:                 0x50C68
      Linked Logon ID:          0x50C48
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x57c
      Process Name:             C:\Windows\System32\svchost.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   127.0.0.1
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0

    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             WIN10-01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Information:
      Logon Type:               7
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:             redqueen
      Account Domain:           UMBRELLA
      Logon ID:                 0x50CC6
      Linked Logon ID:          0x50DE2
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {1566892b-be13-5280-4ea0-7712e556c9d4}

    Process Information:
      Process ID:               0x288
      Process Name:             C:\Windows\System32\lsass.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   -
      Source Port:            -

    Detailed Authentication Information:
      Logon Process:            Negotiat
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             WIN10-01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Information:
      Logon Type:               7
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           No

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:             redqueen
      Account Domain:           UMBRELLA
      Logon ID:                 0x50DE2
      Linked Logon ID:          0x50CC6
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x288
      Process Name:             C:\Windows\System32\lsass.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   -
      Source Port:              -

    Detailed Authentication Information:
      Logon Process:            Negotiat
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4648 
    ```text
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             WIN10-01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:             redqueen
      Account Domain:           UMBRELLA
      Logon GUID:               {1566892b-be13-5280-4ea0-7712e556c9d4}

    Target Server:
      Target Server Name:       localhost
      Additional Information:   localhost

    Process Information:
      Process ID:               0x288
      Process Name:             C:\Windows\System32\lsass.exe

    Network Information:
      Network Address:          -
      Port:                     -
    ```
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           WIN10-01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           redqueen
      Account Domain:         UMBRELLA
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0x57c
      Process Name:           C:\Windows\System32\svchost.exe

    Network Information:
      Network Address:        127.0.0.1
      Port:                   0

    ```
    - 4672
    ```
    Special privileges assigned to new logon.

    Subject:
      Security ID:    S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:   redqueen
      Account Domain: UMBRELLA
      Logon ID:       0x50CC6

    Privileges:       SeSecurityPrivilege
                      SeTakeOwnershipPrivilege
                      SeLoadDriverPrivilege
                      SeBackupPrivilege
                      SeRestorePrivilege
                      SeDebugPrivilege
                      SeSystemEnvironmentPrivilege
                      SeImpersonatePrivilege
                      SeDelegateSessionUserImpersonatePrivilege
    ```

    ```
    Special privileges assigned to new logon.

    Subject:
      Security ID:   S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:  redqueen
      Account Domain:UMBRELLA
      Logon ID:      0x50C48

    Privileges:      SeSecurityPrivilege
                     SeTakeOwnershipPrivilege
                     SeLoadDriverPrivilege
                     SeBackupPrivilege
                     SeRestorePrivilege
                     SeDebugPrivilege
                     SeSystemEnvironmentPrivilege
                     SeImpersonatePrivilege
                     SeDelegateSessionUserImpersonatePrivilege
    ```
  - On DC:
    - 4624 (2x) 
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:             redqueen
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x13FBF07E
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {5dda9868-b8d8-4093-98f1-92e7f419077d}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.1.90.4
      Source Port:              49757

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:             S-1-0-0
      Account Name:            -
      Account Domain:          -
      Logon ID:                0x0

    Logon Information:
      Logon Type:              3
      Restricted Admin Mode:   -
      Virtual Account:         No
      Elevated Token:          Yes

    Impersonation Level:       Impersonation

    New Logon:
      Security ID:             S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:            redqueen
      Account Domain:          UMBRELLA.CORP
      Logon ID:                0x13FBF0D9
      Linked Logon ID:         0x0
      Network Account Name:    -
      Network Account Domain:  -
      Logon GUID:              {5dda9868-b8d8-4093-98f1-92e7f419077d}

    Process Information:
      Process ID:              0x0
      Process Name:            -

    Network Information:
      Workstation Name:        -
      Source Network Address:  10.1.90.4
      Source Port:             49760

    Detailed Authentication Information:
      Logon Process:           Kerberos
      Authentication Package:  Kerberos
      Transited Services:      -
      Package Name (NTLM only):-
      Key Length:              0

    ```
    - 4672 (2x)
    ```
    Special privileges assigned to new logon.

    Subject:
      Security ID:    S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:   redqueen
      Account Domain: UMBRELLA
      Logon ID:       0x13FBF07E

    Privileges:       SeSecurityPrivilege
                      SeBackupPrivilege
                      SeRestorePrivilege
                      SeTakeOwnershipPrivilege
                      SeDebugPrivilege
                      SeSystemEnvironmentPrivilege
                      SeLoadDriverPrivilege
                      SeImpersonatePrivilege
                      SeDelegateSessionUserImpersonatePrivilege
                      SeEnableDelegationPrivilege
    ```
    ```
    Special privileges assigned to new logon.

    Subject:
      Security ID:    S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:   redqueen
      Account Domain: UMBRELLA
      Logon ID:       0x13FBF0D9

    Privileges:       SeSecurityPrivilege
                      SeBackupPrivilege
                      SeRestorePrivilege
                      SeTakeOwnershipPrivilege
                      SeDebugPrivilege
                      SeSystemEnvironmentPrivilege
                      SeLoadDriverPrivilege
                      SeImpersonatePrivilege
                      SeDelegateSessionUserImpersonatePrivilege
                      SeEnableDelegationPrivilege
    ```
    - 4768 (2x)
    ```
    A Kerberos authentication ticket (TGT) was requested.

    Account Information:
      Account Name:             redqueen
      Supplied Realm Name:      UMBRELLA
      User ID:                  S-1-5-21-908899349-1681392183-178755882-1104

    Service Information:
      Service Name:             krbtgt
      Service ID:               S-1-5-21-908899349-1681392183-178755882-502

    Network Information:
      Client Address:           ::ffff:10.1.90.4
      Client Port:              49752

    Additional Information:
      Ticket Options:           0x40810010
      Result Code:              0x0
      Ticket Encryption Type:   0x12
      Pre-Authentication Type:  2

    Certificate Information:
      Certificate Issuer Name:		
      Certificate Serial Number:	
      Certificate Thumbprint:		
    ```
    ```
    A Kerberos authentication ticket (TGT) was requested.

    Account Information:
      Account Name:             redqueen
      Supplied Realm Name:      UMBRELLA.CORP
      User ID:                  S-1-5-21-908899349-1681392183-178755882-1104

    Service Information:
      Service Name:             krbtgt
      Service ID:               S-1-5-21-908899349-1681392183-178755882-502

    Network Information:
      Client Address:           ::ffff:10.1.90.4
      Client Port:              49755

    Additional Information:
      Ticket Options:           0x40810010
      Result Code:              0x0
      Ticket Encryption Type:   0x12
      Pre-Authentication Type:  2

    Certificate Information:
      Certificate Issuer Name:		
      Certificate Serial Number:	
      Certificate Thumbprint:		
    ```
    - 4769 (2x) 
    ```
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           redqueen@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {1566892b-be13-5280-4ea0-7712e556c9d4}

    Service Information:
      Service Name:           WIN10-01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1110

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            49753

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
    ```
      A Kerberos service ticket was requested.

    Account Information:
      Account Name:           redqueen@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {1566892b-be13-5280-4ea0-7712e556c9d4}

    Service Information:
      Service Name:           DC01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1000

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            49758

    Additional Information:
      Ticket Options:         0x40800000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```

### Unprivileged Domain User - NTLM Enforced
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4624
    ```
	 An account was successfully logged on.

	  Subject:
		  Security ID:		S-1-5-18
		  Account Name:		WIN10-01$
		  Account Domain:		UMBRELLA
		  Logon ID:		0x3E7

    Logon Information:
      Logon Type:		11
      Restricted Admin Mode:	-
      Virtual Account:		No
      Elevated Token:		No

    Impersonation Level:		Impersonation

    New Logon:
      Security ID:		S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:		charles
      Account Domain:		UMBRELLA
      Logon ID:		0x17AFB3
      Linked Logon ID:		0x0
      Network Account Name:	-
      Network Account Domain:	-
      Logon GUID:		{00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:		0x684
      Process Name:		C:\Windows\System32\svchost.exe

    Network Information:
      Workstation Name:	WIN10-01
      Source Network Address:	127.0.0.1
      Source Port:		0

    Detailed Authentication Information:
      Logon Process:		User32 
      Authentication Package:	Negotiate
      Transited Services:	-
      Package Name (NTLM only):	-
      Key Length:		0
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:		S-1-5-18
      Account Name:		WIN10-01$
      Account Domain:		UMBRELLA
      Logon ID:		0x3E7

    Logon Information:
      Logon Type:		7
      Restricted Admin Mode:	-
      Virtual Account:		No
      Elevated Token:		No

    Impersonation Level:		Impersonation

    New Logon:
      Security ID:		S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:		charles
      Account Domain:		UMBRELLA
      Logon ID:		0x181ECD
      Linked Logon ID:		0x0
      Network Account Name:	-
      Network Account Domain:	-
      Logon GUID:		{00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:		0x298
      Process Name:		C:\Windows\System32\lsass.exe

    Network Information:
      Workstation Name:	WIN10-01
      Source Network Address:	-
      Source Port:		-

    Detailed Authentication Information:
      Logon Process:		Negotiat
      Authentication Package:	Negotiate
      Transited Services:	-
      Package Name (NTLM only):	-
      Key Length:		0
    ```
    - 4648
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:		S-1-5-18
      Account Name:		WIN10-01$
      Account Domain:		UMBRELLA
      Logon ID:		0x3E7
      Logon GUID:		{00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:		charles
      Account Domain:		UMBRELLA
      Logon GUID:		{00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:	localhost
      Additional Information:	localhost

    Process Information:
      Process ID:		0x298
      Process Name:		C:\Windows\System32\lsass.exe

    Network Information:
      Network Address:	-
      Port:			-
    ```
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:		S-1-5-18
      Account Name:		WIN10-01$
      Account Domain:		UMBRELLA
      Logon ID:		0x3E7
      Logon GUID:		{00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:		charles
      Account Domain:		UMBRELLA
      Logon GUID:		{00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:	localhost
      Additional Information:	localhost

    Process Information:
      Process ID:		0x684
      Process Name:		C:\Windows\System32\svchost.exe

    Network Information:
      Network Address:	127.0.0.1
      Port:			0
    ```
  - On DC:
    - 4624
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0x7751F98
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   10.1.90.4
      Source Port:              49839

    Detailed Authentication Information:
      Logon Process:            NtLmSsp 
      Authentication Package:   NTLM
      Transited Services:       -
      Package Name (NTLM only): NTLM V2
      Key Length:               128
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0x774FD2D
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   10.1.90.4
      Source Port:              49793

    Detailed Authentication Information:
      Logon Process:            NtLmSsp 
      Authentication Package:   NTLM
      Transited Services:       -
      Package Name (NTLM only): NTLM V2
      Key Length:               128
    ```
	  - 4776
    ```
    The computer attempted to validate the credentials for an account.

    Authentication Package:     MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Logon Account:              charles
    Source Workstation:         WIN10-01
    Error Code:                 0x0
    ```
    ```
    The computer attempted to validate the credentials for an account.

    Authentication Package:	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Logon Account:	charles
    Source Workstation:	WIN10-01
    Error Code:	0x0
    ```
    ```
    The computer attempted to validate the credentials for an account.

    Authentication Package:	MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Logon Account:	charles
    Source Workstation:	WIN10-01
    Error Code:	0x0
    ```

### Unprivileged Domain User - No connection to DC/cached credentials
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4624 (2x)
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             WIN10-01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Information:
      Logon Type:               11
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           No

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0x3C37D7
      Linked Logon ID:		0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x57c
      Process Name:             C:\Windows\System32\svchost.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   127.0.0.1
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             WIN10-01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Information:
      Logon Type:               7
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           No

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0x3D6EFD
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x288
      Process Name:             C:\Windows\System32\lsass.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Netwo              k Address: -
      Source Port:              -

    Detailed Authentication Information:
      Logon Process:            Negotiat
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0

    ```
    - 4648 (2x)
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           WIN10-01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0x57c
      Process Name:           C:\Windows\System32\svchost.exe

    Network Information:
      Network Address:        127.0.0.1
      Port:                   0

    ```
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           WIN10-01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0x288
      Process Name:           C:\Windows\System32\lsass.exe

    Network Information:
      Network Address:        -
      Port:                   -
    ```

### Local User
- User: win10-01\Localadmin
- Events recorded:
  - On Client:
    - 4624 (2x)
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             WIN10-01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Information:
      Logon Type:               2
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-2430966876-3932776285-3037176140-1001
      Account Name:             Localadmin
      Account Domain:           WIN10-01
      Logon ID:                 0x41A1F0
      Linked Logon ID:          0x41A21E
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x57c
      Process Name:             C:\Windows\System32\svchost.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   127.0.0.1
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             WIN10-01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Information:
      Logon Type:               2
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           No 
    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-2430966876-3932776285-3037176140-1001
      Account Name:             Localadmin
      Account Domain:           WIN10-01
      Logon ID:                 0x41A21E
      Linked Logon ID:          0x41A1F0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x57c
      Process Name:             C:\Windows\System32\svchost.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   127.0.0.1
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-21-2430966876-3932776285-3037176140-1001
      Account Name:             Localadmin
      Account Domain:           WIN10-01
      Logon ID:                 0x41A21E

    Logon Information:
      Logon Type:               2
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           No

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0x50BD86
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {09baf998-4917-36e4-a37d-50c167f4aae6}

    Process Information:
      Process ID:               0x1a14
      Process Name:             C:\Windows\System32\svchost.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   ::1
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            seclogo
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4648 
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           WIN10-01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           Localadmin
      Account Domain:         WIN10-01
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0x57c
      Process Name:           C:\Windows\System32\svchost.exe

    Network Information:
      Network Address:        127.0.0.1
      Port:                   0
    ```
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-21-2430966876-3932776285-3037176140-1001
      Account Name:           Localadmin
      Account Domain:         WIN10-01
      Logon ID:               0x41A21E
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA
      Logon GUID:             {09baf998-4917-36e4-a37d-50c167f4aae6}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0x1a14
      Process Name:           C:\Windows\System32\svchost.exe

    Network Information:
      Network Address:        ::1
      Port:                   0
    ```
    - 4672 
    ```
    Special privileges assigned to new logon.

    Subject:
      Security ID:    S-1-5-21-2430966876-3932776285-3037176140-1001
      Account Name:   Localadmin
      Account Domain: WIN10-01
      Logon ID:       0x41A1F0

    Privileges:       SeSecurityPrivilege
                      SeTakeOwnershipPrivilege
                      SeLoadDriverPrivilege
                      SeBackupPrivilege
                      SeRestorePrivilege
                      SeDebugPrivilege
                      SeSystemEnvironmentPrivilege
                      SeImpersonatePrivilege
                      SeDelegateSessionUserImpersonatePrivilege
    ```
    - 4776
    ```
    The computer attempted to validate the credentials for an account.

    Authentication Package: MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Logon Account:          Localadmin
    Source Workstation:     WIN10-01
    Error Code:             0x0
    ```

## Run as

### Domain User
- Current User: win10-01\Localadmin
- Runas User: umbrella\charles
- Events recorded:
  - On Source:
    - 4624
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-21-2430966876-3932776285-3037176140-1001
      Account Name:             Localadmin
      Account Domain:           WIN10-01
      Logon ID:                 0x41A21E

    Logon Information:
      Logon Type:               2
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           No

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0x50BD86
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {09baf998-4917-36e4-a37d-50c167f4aae6}

    Process Information:
      Process ID:               0x1a14
      Process Name:             C:\Windows\System32\svchost.exe

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   ::1
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            seclogo
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4648
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-21-2430966876-3932776285-3037176140-1001
      Account Name:           Localadmin
      Account Domain:         WIN10-01
      Logon ID:               0x41A21E
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA
      Logon GUID:             {09baf998-4917-36e4-a37d-50c167f4aae6}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0x1a14
      Process Name:           C:\Windows\System32\svchost.exe

    Network Information:
      Network Address:        ::1
      Port:                   0
    ```
  - On DC:
    - 4768 
    ```
      A Kerberos authentication ticket (TGT) was requested.

    Account Information:
      Account Name:             charles
      Supplied Realm Name:      UMBRELLA
      User ID:                  S-1-5-21-908899349-1681392183-178755882-1105

    Service Information:
      Service Name:             krbtgt
      Service ID:               S-1-5-21-908899349-1681392183-178755882-502

    Network Information:
      Client Address:           ::ffff:10.1.90.4
      Client Port:              50458

    Additional Information:
      Ticket Options:           0x40810010
      Result Code:              0x0
      Ticket Encryption Type:   0x12
      Pre-Authentication Type:  2

    Certificate Information:
      Certificate Issuer Name:		
      Certificate Serial Number:	
      Certificate Thumbprint:		
    ```
    - 4769
    ```
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {09baf998-4917-36e4-a37d-50c167f4aae6}

    Service Information:
      Service Name:           WIN10-01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1110

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            50459

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```

## RDP Login
- Source: Win10-01 (10.1.90.4)
- Destination: APP01 (10.2.1.150)

### Unprivileged Domain User with NLA
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4648 (2x)
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:           redqueen
      Account Domain:         UMBRELLA
      Logon ID:               0x542833
      Logon GUID:		{00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {08e6f871-241e-55cb-bbc9-c8351fb50a0f}

    Target Server:
      Target Server Name:     APP01
      Additional Information: TERMSRV/APP01

    Process Information:
      Process ID:             0x288
      Process Name:           C:\Windows\System32\lsass.exe

    Network Information:
      Network Address:        -
      Port:                   -
    ```
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:           redqueen
      Account Domain:         UMBRELLA
      Logon ID:               0x542833
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {08e6f871-241e-55cb-bbc9-c8351fb50a0f}

    Target Server:
      Target Server Name:     APP01
      Additional Information: TERMSRV/APP01

    Process Information:
      Process ID:             0x288
      Process Name:           C:\Windows\System32\lsass.exe

    Network Information:
      Network Address:        -
      Port:                   -
    ```
  - On Destination:
    -  4624 (2x)
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Type:                 3

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0x1182184
      Logon GUID:               {6BF1BED0-0816-9028-29E0-44935EF890A8}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   -
      Source Port:              -

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0

    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             APP01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Type:                 10

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0x1184646
      Logon GUID:               {08E6F871-241E-55CB-BBC9-C8351FB50A0F}

    Process Information:
      Process ID:               0xdd8
      Process Name:             C:\Windows\System32\winlogon.exe

    Network Information:
      Workstation Name:         APP01
      Source Network Address:   10.1.90.4
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0

    ```
    - 4648
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           APP01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA
      Logon GUID:             {08E6F871-241E-55CB-BBC9-C8351FB50A0F}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0xdd8
      Process Name:           C:\Windows\System32\winlogon.exe

    Network Information:
      Network Address:        10.1.90.4
      Port:                   0

    ```
  - On DC:
    - 4624 (3x)
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x1408A575
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {87a5dc53-9ec2-9942-16c9-097507a4b745}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.2.1.150
      Source Port:              53698

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```
      An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x1408A62E
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {87a5dc53-9ec2-9942-16c9-097507a4b745}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.2.1.150
      Source Port:              53702

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0

    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x1408A6B4
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {7b8fc3c4-8664-b856-503c-0171fb1e6d5b}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.2.1.150
      Source Port:              53703

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4768 (2x)
    ```
    A Kerberos authentication ticket (TGT) was requested.

    Account Information:
      Account Name:             charles
      Supplied Realm Name:      UMBRELLA
      User ID:                  S-1-5-21-908899349-1681392183-178755882-1105

    Service Information:
      Service Name:             krbtgt
      Service ID:               S-1-5-21-908899349-1681392183-178755882-502

    Network Information:
      Client Address:           ::ffff:10.1.90.4
      Client Port:              50519

    Additional Information:
      Ticket Options:           0x40810010
      Result Code:              0x0
      Ticket Encryption Type:   0x12
      Pre-Authentication Type:  2

    Certificate Information:
      Certificate Issuer Name:		
      Certificate Serial Number:	
      Certificate Thumbprint:		
    ```
    ```
    A Kerberos authentication ticket (TGT) was requested.

    Account Information:
      Account Name:             charles
      Supplied Realm Name:      UMBRELLA.CORP
      User ID:                  S-1-5-21-908899349-1681392183-178755882-1105

    Service Information:
      Service Name:             krbtgt
      Service ID:               S-1-5-21-908899349-1681392183-178755882-502

    Network Information:
      Client Address:           ::ffff:10.2.1.150
      Client Port:              53691

    Additional Information:
      Ticket Options:           0x40810010
      Result Code:              0x0
      Ticket Encryption Type:   0x12
      Pre-Authentication Type:  2

    Certificate Information:
      Certificate Issuer Name:		
      Certificate Serial Number:	
      Certificate Thumbprint:		

    ```
    - 4769 (5x)
    ```
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {08e6f871-241e-55cb-bbc9-c8351fb50a0f}

    Service Information:
      Service Name:           APP01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1107

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            50520

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
    ```
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {08e6f871-241e-55cb-bbc9-c8351fb50a0f}

    Service Information:
      Service Name:           APP01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1107

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            50521

    Additional Information:
      Ticket Options:         0x40810008
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -

    ```
    ```
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {08e6f871-241e-55cb-bbc9-c8351fb50a0f}

    Service Information:
      Service Name:           APP01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1107

    Network Information:
      Client Address:         ::ffff:10.2.1.150
      Client Port:            53692

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
    ```
      A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {ace88e5f-dcc2-a2e1-5984-027621e0dd94}

    Service Information:
      Service Name:           DC01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1000

    Network Information:
      Client Address:         ::ffff:10.2.1.150
      Client Port:            53699

    Additional Information:
      Ticket Options:         0x40800000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -

    ```
    ```
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {be8e313c-7a29-3de3-8059-5017584eb791}

    Service Information:
      Service Name:           DC01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1000

    Network Information:
      Client Address:         ::ffff:10.2.1.150
      Client Port:            53704

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -

    ```

### Unprivileged Domain User with NLA - NTLM enforced
- User: umbrella\charles
- Events recorded:
  - On Destination:
	- 4624:
	```
	An account was successfully logged on.

	Subject:
		Security ID:              S-1-5-18
		Account Name:             APP01$
		Account Domain:           UMBRELLA
		Logon ID:                 0x3E7

	Logon Type:                 10

	Impersonation Level:        Impersonation

	New Logon:
		Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:             charles
		Account Domain:           UMBRELLA
		Logon ID:                 0x27DEA17
		Logon GUID:               {00000000-0000-0000-0000-000000000000}

	Process Information:
		Process ID:               0x5ec
		Process Name:             C:\Windows\System32\winlogon.exe

	Network Information:
		Workstation Name:         APP01
		Source Network Address:   10.1.90.4
		Source Port:              0

	Detailed Authentication Information:
		Logon Process:            User32 
		Authentication Package:   Negotiate
		Transited Services:       -
		Package Name (NTLM only): -
		Key Length:               0
	```
	```
	An account was successfully logged on.

	Subject:
		Security ID:              S-1-0-0
		Account Name:             -
		Account Domain:           -
		Logon ID:                 0x0

	Logon Type:                 3

	Impersonation Level:        Impersonation

	New Logon:
		Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:             charles
		Account Domain:           UMBRELLA
		Logon ID:                 0x27DC8A0
		Logon GUID:               {00000000-0000-0000-0000-000000000000}

	Process Information:
		Process ID:               0x0
		Process Name:             -

	Network Information:
		Workstation Name:         WIN10-01
		Source Network Address:   -
		Source Port:              -

	Detailed Authentication Information:
		Logon Process:            NtLmSsp 
		Authentication Package:   NTLM
		Transited Services:       -
		Package Name (NTLM only): NTLM V2
		Key Length:               128
	```
	```
	An account was successfully logged on.

	Subject:
		Security ID:              S-1-0-0
		Account Name:             -
		Account Domain:           -
		Logon ID:                 0x0

	Logon Type:                 3

	Impersonation Level:        Impersonation

	New Logon:
		Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:             charles
		Account Domain:           UMBRELLA
		Logon ID:                 0x27DC811
		Logon GUID:               {00000000-0000-0000-0000-000000000000}

	Process Information:
		Process ID:               0x0
		Process Name:             -

	Network Information:
		Workstation Name:         WIN10-01
		Source Network Address:   -
		Source Port:              -

	Detailed Authentication Information:
		Logon Process:            NtLmSsp 
		Authentication Package:   NTLM
		Transited Services:       -
		Package Name (NTLM only): NTLM V2
		Key Length:               128
	```
	- 4648:
	```
	A logon was attempted using explicit credentials.

	Subject:
		Security ID:              S-1-5-18
		Account Name:             APP01$
		Account Domain:           UMBRELLA
		Logon ID:                 0x3E7
		Logon GUID:               {00000000-0000-0000-0000-000000000000}

	Account Whose Credentials Were Used:
		Account Name:             charles
		Account Domain:           UMBRELLA
		Logon GUID:               {00000000-0000-0000-0000-000000000000}

	Target Server:
		Target Server Name:       localhost
		Additional Information:   localhost

	Process Information:
		Process ID:               0x5ec
		Process Name:             C:\Windows\System32\winlogon.exe

	Network Information:
		Network Address:          10.1.90.4
		Port:                     0
	```
  - On DC:
    - 4624
	```
	An account was successfully logged on.

	Subject:
		Security ID:              S-1-0-0
		Account Name:             -
		Account Domain:           -
		Logon ID:                 0x0

	Logon Information:
		Logon Type:               3
		Restricted Admin Mode:    -
		Virtual Account:          No
		Elevated Token:           Yes

	Impersonation Level:        Impersonation

	New Logon:
		Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:             charles
		Account Domain:           UMBRELLA
		Logon ID:                 0x76FFBAE
		Linked Logon ID:          0x0
		Network Account Name:     -
		Network Account Domain:   -
		Logon GUID:               {00000000-0000-0000-0000-000000000000}

	Process Information:
		Process ID:               0x0
		Process Name:             -

	Network Information:
		Workstation Name:         APP01
		Source Network Address:   10.2.1.150
		Source Port:              51793

	Detailed Authentication Information:
		Logon Process:            NtLmSsp 
		Authentication Package:   NTLM
		Transited Services:       -
		Package Name (NTLM only): NTLM V2
		Key Length:               128
	```
	```
	An account was successfully logged on.

	Subject:
		Security ID:              S-1-0-0
		Account Name:             -
		Account Domain:           -
		Logon ID:                 0x0

	Logon Information:
		Logon Type:               3
		Restricted Admin Mode:    -
		Virtual Account:          No
		Elevated Token:           Yes

	Impersonation Level:        Impersonation

	New Logon:
		Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:             charles
		Account Domain:           UMBRELLA
		Logon ID:                 0x76FFB87
		Linked Logon ID:          0x0
		Network Account Name:     -
		Network Account Domain:   -
		Logon GUID:               {00000000-0000-0000-0000-000000000000}

	Process Information:
		Process ID:               0x0
		Process Name:             -

	Network Information:
		Workstation Name:         APP01
		Source Network Address:   10.2.1.150
		Source Port:		51792

	Detailed Authentication Information:
		Logon Process:            NtLmSsp 
		Authentication Package:   NTLM
		Transited Services:       -
		Package Name (NTLM only): NTLM V2
		Key Length:               128
	```
	```
	An account was successfully logged on.

	Subject:
		Security ID:              S-1-0-0
		Account Name:             -
		Account Domain:           -
		Logon ID:                 0x0

	Logon Information:
		Logon Type:               3
		Restricted Admin Mode:    -
		Virtual Account:          No
		Elevated Token:           Yes

	Impersonation Level:        Impersonation

	New Logon:
		Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
		Account Name:             charles
		Account Domain:           UMBRELLA
		Logon ID:                 0x76FFAF8
		Linked Logon ID:          0x0
		Network Account Name:     -
		Network Account Domain:   -
		Logon GUID:               {00000000-0000-0000-0000-000000000000}

	Process Information:
		Process ID:               0x0
		Process Name:             -

	Network Information:
		Workstation Name:         APP01
		Source Network Address:   10.2.1.150
		Source Port:              51790

	Detailed Authentication Information:
		Logon Process:            NtLmSsp 
		Authentication Package:   NTLM
		Transited Services:       -
		Package Name (NTLM only): NTLM V2
		Key Length:               128
	```
	- 4776:
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:     MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:              charles
	Source Workstation:         APP01
	Error Code:                 0x0
	```
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:     MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:              charles
	Source Workstation:         APP01
	Error Code:                 0x0
	```
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:     MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:              charles
	Source Workstation:         APP01
	Error Code:                 0x0
	```
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:     MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:              charles
	Source Workstation:         APP01
	Error Code:                 0x0
	```
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:     MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:              charles
	Source Workstation:         WIN10-01
	Error Code:                 0x0
	```
	```
	The computer attempted to validate the credentials for an account.

	Authentication Package:     MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
	Logon Account:              charles
	Source Workstation:         WIN10-01
	Error Code:                 0x0
	```

### Unprivileged Local User with NLA
- User: APP01\Administrator
- Events recorded:
  - On Source:
    - 4648
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:           redqueen
      Account Domain:         UMBRELLA
      Logon ID:               0x542833
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           Administrator
      Account Domain:         APP01
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:     APP01.umbrella.corp
      Additional Information: APP01.umbrella.corp

    Process Information:
      Process ID:             0x288
      Process Name:           C:\Windows\System32\lsass.exe

    Network Information:
      Network Address:        -
      Port:                   -
    ```
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:           redqueen
      Account Domain:         UMBRELLA
      Logon ID:               0x542833
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           Administrator
      Account Domain:         APP01
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:     APP01.umbrella.corp
      Additional Information: APP01.umbrella.corp

    Process Information:
      Process ID:             0x288
      Process Name:           C:\Windows\System32\lsass.exe

    Network Information:
      Network Address:        -
      Port:                   -
    ```
  - On Destination:
    - 4624 (3x)
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Type:                 3

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-711647069-3496584513-2683447958-500
      Account Name:             Administrator
      Account Domain:           APP01
      Logon ID:                 0x11B0201
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   -
      Source Port:              -

    Detailed Authentication Information:
      Logon Process:            NtLmSsp 
      Authentication Package:   NTLM
      Transited Services:       -
      Package Name (NTLM only): NTLM V2
      Key Length:               128
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Type:                 3

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-711647069-3496584513-2683447958-500
      Account Name:             Administrator
      Account Domain:           APP01
      Logon ID:                 0x11B0282
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   -
      Source Port:              -

    Detailed Authentication Information:
      Logon Process:            NtLmSsp 
      Authentication Package:   NTLM
      Transited Services:       -
      Package Name (NTLM only): NTLM V2
      Key Length:               128
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             APP01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Type:                 10

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-711647069-3496584513-2683447958-500
      Account Name:             Administrator
      Account Do                ain: APP01
      Logon ID:                 0x11B26A0
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0xe0c
      Process Name:             C:\Windows\System32\winlogon.exe

    Network Information:
      Workstation Name:         APP01
      Source Network Address:   10.1.90.4
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4648
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           APP01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           Administrator
      Account Domain:         APP01
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0xe0c
      Process Name:           C:\Windows\System32\winlogon.exe

    Network Information:
      Network Address:        10.1.90.4
      Port:                   0

    ```
    - 4672
    ```
    Special privileges assigned to new logon.

    Subject:
      Security ID:    S-1-5-21-711647069-3496584513-2683447958-500
      Account Name:   Administrator
      Account Domain: APP01
      Logon ID:       0x11B0282

    Privileges:       SeSecurityPrivilege
                      SeBackupPrivilege
                      SeRestorePrivilege
                      SeTakeOwnershipPrivilege
                      SeDebugPrivilege
                      SeSystemEnvironmentPrivilege
                      SeLoadDriverPrivilege
                      SeImpersonatePrivilege
    ```
    ```
    Special privileges assigned to new logon.

    Subject:
      Security ID:    S-1-5-21-711647069-3496584513-2683447958-500
      Account Name:   Administrator
      Account Domain: APP01
      Logon ID:       0x11B26A0

    Privileges:       SeSecurityPrivilege
                      SeTakeOwnershipPrivilege
                      SeLoadDriverPrivilege
                      SeBackupPrivilege
                      SeRestorePrivilege
                      SeDebugPrivilege
                      SeSystemEnvironmentPrivilege
                      SeImpersonatePrivilege
    ```
    ```
    Special privileges assigned to new logon.

    Subject:
      Security ID:    S-1-5-21-711647069-3496584513-2683447958-500
      Account Name:   Administrator
      Account Domain: APP01
      Logon ID:       0x11B0201

    Privileges:       SeSecurityPrivilege
                      SeBackupPrivilege
                      SeRestorePrivilege
                      SeTakeOwnershipPrivilege
                      SeDebugPrivilege
                      SeSystemEnvironmentPrivilege
                      SeLoadDriverPrivilege
                      SeImpersonatePrivilege
    ```
    - 4776 (3x)
    ```
    The computer attempted to validate the credentials for an account.

    Authentication Package: MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Logon Account:          Administrator
    Source Workstation:     WIN10-01
    Error Code:             0x0
    ```
    ```
    The computer attempted to validate the credentials for an account.

    Authentication Package: MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Logon Account:          Administrator
    Source Workstation:     WIN10-01
    Error Code:             0x0
    ```
    ```
    The computer attempted to validate the credentials for an account.

    Authentication Package: MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Logon Account:          Administrator
    Source Workstation:     APP01
    Error Code:             0x0
    ```

### Unprivileged Domain User without NLA
- User: umbrella\charles
- Events recorded:
  - On Destination:
    -  4624 
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             APP01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Type:                 10

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0x58779
      Logon GUID:               {01949FBC-A6C9-8F3C-8CC5-B8060F3141FD}

    Process Information:
      Process ID:               0x928
      Process Name:             C:\Windows\System32\winlogon.exe

    Network Information:
      Workstation Name:         APP01
      Source Network Address:   10.1.90.4
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4648
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           APP01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA
      Logon GUID:             {01949FBC-A6C9-8F3C-8CC5-B8060F3141FD}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0x928
      Process Name:           C:\Windows\System32\winlogon.exe

    Network Information:
      Network Address:        10.1.90.4
      Port:                   0
    ```
  - On DC:
    - 4624 (3x)
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x140EF548
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {e8c6ac79-cf51-384e-f8b2-6ec1a03685a0}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.2.1.150
      Source Port:              49163

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:                 0x140EF57E
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {e8c6ac79-cf51-384e-f8b2-6ec1a03685a0}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.2.1.150
      Source Port:              49233

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0

    ```
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Information:
      Logon Type:               3
      Restricted Admin Mode:    -
      Virtual Account:          No
      Elevated Token:           Yes

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA.CORP
      Logon ID:		0x140EF5BF
      Linked Logon ID:          0x0
      Network Account Name:     -
      Network Account Domain:   -
      Logon GUID:               {1e8cd504-d5c6-21c1-e9e0-3a8e86cddd84}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.2.1.150
      Source Port:              49234

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4768 
    ```
    A Kerberos authentication ticket (TGT) was requested.

    Account Information:
      Account Name:             charles
      Supplied Realm Name:      umbrella
      User ID:                  S-1-5-21-908899349-1681392183-178755882-1105

    Service Information:
      Service Name:             krbtgt
      Service ID:               S-1-5-21-908899349-1681392183-178755882-502

    Network Information:
      Client Address:           ::ffff:10.2.1.150
      Client Port:              49223

    Additional Information:
      Ticket Options:           0x40810010
      Result Code:              0x0
      Ticket Encryption Type:   0x12
      Pre-Authentication Type:  2

    Certificate Information:
      Certificate Issuer Name:		
      Certificate Serial Number:	
      Certificate Thumbprint:		
    ```
    - 4769 (3x)
    ```
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {01949fbc-a6c9-8f3c-8cc5-b8060f3141fd}

    Service Information:
      Service Name:           APP01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1107

    Network Information:
      Client Address:         ::ffff:10.2.1.150
      Client Port:            49224

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -

    ```
    ```
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {604be338-b9ac-8bce-e1b4-73f3ca73e114}

    Service Information:
      Service Name:           DC01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1000

    Network Information:
      Client Address:         ::ffff:10.2.1.150
      Client Port:            49231

    Additional Information:
      Ticket Options:         0x40800000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```
    ```
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {63456a9c-2285-caaf-2867-3c5f1aa14f8d}

    Service Information:
      Service Name:           DC01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1000

    Network Information:
      Client Address:         ::ffff:10.2.1.150
      Client Port:            49235

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```

### Local User without NLA
- User: APP01\Administrator
- Events recorded:
  - On Destination:
    -  4624 
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-5-18
      Account Name:             APP01$
      Account Domain:           UMBRELLA
      Logon ID:                 0x3E7

    Logon Type:                 10

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-711647069-3496584513-2683447958-500
      Account Name:             Administrator
      Account Domain:           APP01
      Logon ID:                 0x74DCA
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x8fc
      Process Name:             C:\Windows\System32\winlogon.exe

    Network Information:
      Workstation Name:         APP01
      Source Network Address:   10.1.90.4
      Source Port:              0

    Detailed Authentication Information:
      Logon Process:            User32 
      Authentication Package:   Negotiate
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
    - 4648
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-18
      Account Name:           APP01$
      Account Domain:         UMBRELLA
      Logon ID:               0x3E7
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           Administrator
      Account Domain:         APP01
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:     localhost
      Additional Information: localhost

    Process Information:
      Process ID:             0x8fc
      Process Name:           C:\Windows\System32\winlogon.exe

    Network Information:
      Network Address:        10.1.90.4
      Port:                   0

    ```
    - 4672 
    ```
    Special privileges assigned to new logon.

    Subject:
      Security ID:    S-1-5-21-711647069-3496584513-2683447958-500
      Account Name:   Administrator
      Account Domain: APP01
      Logon ID:       0x74DCA

    Privileges:       SeSecurityPrivilege
                      SeTakeOwnershipPrivilege
                      SeLoadDriverPrivilege
                      SeBackupPrivilege
                      SeRestorePrivilege
                      SeDebugPrivilege
                      SeSystemEnvironmentPrivilege
                      SeImpersonatePrivilege
    ```
    - 4776 
    ```
    The computer attempted to validate the credentials for an account.

    Authentication Package: MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Logon Account:          Administrator
    Source Workstation:     APP01
    Error Code:             0x0
    ```

## SMB Share Mount/Login explicit credentials
- Source: Win10-01 (10.1.90.4)
- Destination: APP01 (10.2.1.150)

### Unprivileged Domain User
- User: umbrella\charles
- Events recorded:
  - On Source:
    - 4648
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:           redqueen
      Account Domain:         UMBRELLA
      Logon ID:               0x542833
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           charles
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {559696ad-f93c-1579-4b25-bb3148c043c0}

    Target Server:
      Target Server Name:     APP01
      Additional Information: cifs/APP01

    Process Information:
      Process ID:             0x4
      Process Name:		

    Network Information:
      Network Address:        10.2.1.150
      Port:                   445
    ```
  - On Destination:
    -  4624 
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Type:                 3

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-908899349-1681392183-178755882-1105
      Account Name:             charles
      Account Domain:           UMBRELLA
      Logon ID:                 0xDB58D
      Logon GUID:               {34EA631A-1369-33E9-DBCD-87221F157AF5}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         -
      Source Network Address:   10.1.90.4
      Source Port:              50855

    Detailed Authentication Information:
      Logon Process:            Kerberos
      Authentication Package:   Kerberos
      Transited Services:       -
      Package Name (NTLM only): -
      Key Length:               0
    ```
  - On DC:
    - 4768 
    ```
    A Kerberos authentication ticket (TGT) was requested.

    Account Information:
      Account Name:             charles
      Supplied Realm Name:      UMBRELLA
      User ID:                  S-1-5-21-908899349-1681392183-178755882-1105

    Service Information:
      Service Name:             krbtgt
      Service ID:               S-1-5-21-908899349-1681392183-178755882-502

    Network Information:
      Client Address:           ::ffff:10.1.90.4
      Client Port:              50857

    Additional Information:
      Ticket Options:           0x40810010
      Result Code:              0x0
      Ticket Encryption Type:   0x12
      Pre-Authentication Type:  2

    Certificate Information:
      Certificate Issuer Name:		
      Certificate Serial Number:	
      Certificate Thumbprint:		
    ```
    - 4769 
    ```
    A Kerberos service ticket was requested.

    Account Information:
      Account Name:           charles@UMBRELLA.CORP
      Account Domain:         UMBRELLA.CORP
      Logon GUID:             {559696ad-f93c-1579-4b25-bb3148c043c0}

    Service Information:
      Service Name:           APP01$
      Service ID:             S-1-5-21-908899349-1681392183-178755882-1107

    Network Information:
      Client Address:         ::ffff:10.1.90.4
      Client Port:            50858

    Additional Information:
      Ticket Options:         0x40810000
      Ticket Encryption Type: 0x12
      Failure Code:           0x0
      Transited Services:     -
    ```

### Local User
- User: app01\Administrator
- Events recorded:
  - On Source:
    - 4648
    ```
    A logon was attempted using explicit credentials.

    Subject:
      Security ID:            S-1-5-21-908899349-1681392183-178755882-1104
      Account Name:           redqueen
      Account Domain:         UMBRELLA
      Logon ID:               0x542833
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Account Whose Credentials Were Used:
      Account Name:           Administrator
      Account Domain:         APP01
      Logon GUID:             {00000000-0000-0000-0000-000000000000}

    Target Server:
      Target Server Name:     APP01.umbrella.corp
      Additional Information: APP01.umbrella.corp

    Process Information:
      Process ID:             0x4
      Process Name:		

    Network Information:
      Network Address:        10.2.1.150
      Port:                   445
    ```
  - On Destination:
     -  4624 
    ```
    An account was successfully logged on.

    Subject:
      Security ID:              S-1-0-0
      Account Name:             -
      Account Domain:           -
      Logon ID:                 0x0

    Logon Type:                 3

    Impersonation Level:        Impersonation

    New Logon:
      Security ID:              S-1-5-21-711647069-3496584513-2683447958-500
      Account Name:             Administrator
      Account Domain:           APP01
      Logon ID:                 0xE3F98
      Logon GUID:               {00000000-0000-0000-0000-000000000000}

    Process Information:
      Process ID:               0x0
      Process Name:             -

    Network Information:
      Workstation Name:         WIN10-01
      Source Network Address:   10.1.90.4
      Source Port:              50877

    Detailed Authentication Information:
      Logon Process:            NtLmSsp 
      Authentication Package:   NTLM
      Transited Services:       -
      Package Name (NTLM only): NTLM V2
      Key Length:               128
    ```
    - 4672 
    ```
      Special privileges assigned to new logon.

    Subject:
      Security ID:    S-1-5-21-711647069-3496584513-2683447958-500
      Account Name:   Administrator
      Account Domain: APP01
      Logon ID:       0xE3F98

    Privileges:       SeSecurityPrivilege
                      SeBackupPrivilege
                      SeRestorePrivilege
                      SeTakeOwnershipPrivilege
                      SeDebugPrivilege
                      SeSystemEnvironmentPrivilege
                      SeLoadDriverPrivilege
                      SeImpersonatePrivilege
    ```
    - 4776
    ```
    The computer attempted to validate the credentials for an account.

    Authentication Package: MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Logon Account:          Administrator
    Source Workstation:     WIN10-01
    Error Code:             0x0
    ```