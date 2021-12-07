---
title: "E-Mail Security"
description: "How to protect your domain."
lead: "How to protect your domain."
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu: 
  dfir:
    parent: "guides"
weight: 200
toc: true
---

## SPF
SPF (Sender Policy Framework) defines which mail servers are authorized to send emails for the given domain.
To setup SPF create a TXT DNS entry starting with v=spf1. You can have several keys for different providers, each entry has a name, this name is attached as DKIM header in the mail.

```bash
# example
"v=spf1 ip4:192.0.2.0/24 ip4:198.51.100.123 a -all"
# or
"v=spf1 include:_spf.google.com -all"
```

## DKIM
DKM (Domain Keys Identified) uses public key authentication to sign the sending emails. The receiver can then validate, that nobody 
can spoof or tamper an email within transit. The sender singes all messages using a private key, while the receipient can check the 
validity of the message using public keys, which are provided as DNS TXT record starting with v=DKIM1.

```bash
# example
"v=DKIM1; k=rsa; t=s; p=MIGfMA....."
```

## DMARC
DMARC (Domain-based Message Authentication, Reporting, and Conformance) tells the receiver of an email how to interpret the SPF and DKIM check results. In addition, it allows collecting reports about the action taken on the mails. With that you get notified if someone tries to abuse your domain.

```bash
# example
"v=DMARC1;p=none;sp=quarantine;pct=100;rua=mailto:dmarcreports@example.com;"
```


