---
title: "Introduction / Frameworks"
description: "General Introduction into Incident Response and the process."
lead: "General Introduction into Incident Response and the process."
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu: 
  dfir:
    parent: "ir"
weight: 10
toc: true
---

Most organizations are failing to detect intrusions. Often intrusions are reported by third-party like law enforcement/intelligence agencies. According to FireEye, the median dwell time for incident detection was at 141 days when notified by an external party and 30 days when self-detected.[^1]

The adversaries can be roughly grouped into
- ATP
- Organized Crime
- Hacktivisty
- Insider
- Script Kiddies

## Incident Response Frameworks
Two of the most well known Incident Response Frameworks are from NIST[^2] and from SANS[^3].

The SANS Incident Response Process contains the following steps:

- **Preparation**

  Establish incident response capability, securing systems, applications, and networks.

- **Identification**

  Identify **ALL** systems compromised. Don't stop after the first few findings or directly start with containment. 

- **Containment / Intelligence Development**

  This step does not mean contain/quarantine all the compromised systems! The goal of this step is to heavily monitoring the adversary.
  Analysing the intrusion to get TTP's, identify patient zero, the ways of lateral movement, used malware and so on to build up your threat intelligence, develop IOC's and identify additional compromised systems. 

- **Eradication / Remediation**

  Contain compromised hosts, block IPs/black hole domain names, rebuild systems/network, password change etc. In the end verify all actions.
  This step should be completed in a coordinated way and short time frame!

- **Recovery**

  Move back to daily business. Start planning/implementing long-term solutions to increase your resilience/harden your infrastructure. For example Improve Enterprise Authentication Model, Enhance Network visibility, Vulnerability Management, Setup Logging(e.g. SIEM), Awareness Trainings etc.

- **Lessons Learned / Threat Intel Consumption**

  Follow up to the incident. Verify that the incident has been mitigated and the adversary has been removed.  Audit your network to check the new counter measures. Feed your gathered intelligence from the incident like IOC back to your SOC. 


The Incident Response process is intended to be a continuous loop. It is important to not skipping steps. For example, many organizations skipping directly from Identification to Remediation. **Don't** skip any steps! When you do this, you will probably miss infected systems, don't find out the initial attack vector or alarm the adversary so that the start for example with immediate data exfiltration. This is often called a "Whack a mole" response, where the adversary just move to another system and starts again. 

Also keep attention to emotional distances and be aware that the adversary can suddenly change his goals. Many teams are often vendor driven and think they can stop the intrusion at their perimeter. This is a illusion. We win, when we stop the adversaries from achieving their goals.


[^1]: [FireEye Mandiant M-Trends 2020 Report](https://investors.fireeye.com/news-releases/news-release-details/fireeye-mandiant-m-trends-2020-report-reveals-cyber-criminals)

[^2]: [NIST - Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

[^3]: [SANS - Incident Handler's Handbook](https://www.sans.org/reading-room/whitepapers/incident/incident-handlers-handbook-33901)