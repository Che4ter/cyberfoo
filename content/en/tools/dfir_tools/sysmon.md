---
title: "Sysmon"
description: "Supercharge your windows event logging."
lead: "Supercharge your windows event logging."
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu: 
  tools:
    parent: "dfir_tools"
toc: true
---

System Monitor (Sysmon) is a part of the Sysinternals Tools from Microsoft. It extends the Windows Event Logging Service and allows the logging of events like process creates, network connections, file changes, driver loads and many more. 

## Installation
Download the latest release from [https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)).

```bash
#provide your own config.xml
sysmon -accepteula -i c:\windows\config.xml
```

### Create your sysmon config
The most important part is to create, tune and maintain your sysmon config.
Some good example configs can be found under:
- [SwiftOnSecurity](https://github.com/SwiftOnSecurity/sysmon-config)
- [Sysmon-Modular](https://github.com/olafhartong/sysmon-modular)

It's recommended to use a different config at least for your client, servers and dc.