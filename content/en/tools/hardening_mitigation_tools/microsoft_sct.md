---
title: "Microsoft Security Compliance Toolkit"
description: "Analyze and compare GPO against baselines."
lead: "Analyze and compare GPO against baselines."
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu: 
  tools:
    parent: "hardening_mitigation_tools"
toc: true
---

Microsoft Security Compliance Toolkit[^1] allows to compare your GPO against a baseline. There are several recommended policies available. 
To start check out the DoD guidelines [https://public.cyber.mil/stigs/gpo/](https://public.cyber.mil/stigs/gpo/).

## Installation
Download the PolicyAnalyzer from [https://www.microsoft.com/en-us/download/details.aspx?id=55319](https://www.microsoft.com/en-us/download/details.aspx?id=55319).

## Usage
1. Execute the PolicyAnalyzer.exe.
2. Add Baseline GPO's like from the DoD
3. Select Imported Policy and click "View / Compare"
4. Check the results, under View you can also filter for only Differences or Conflicts.

[^1]: [Microsoft SCT](https://www.microsoft.com/en-us/download/details.aspx?id=55319)