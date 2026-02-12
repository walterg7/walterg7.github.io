---
title: "Baselining"
date: 2026-01-30 00:00:00 -0800
categories: [Cloud, Azure]
tags: [cloud, azure, soc]
description: 
---
Before implementing any controls, it is worth establishing a baseline to evaluate their effectiveness. In this case, we are measuring the number of failed RDP attempts before implementing IP-blocking automation.

This VM was running for approximately 13 hours (1/3/2026 11:00pm - 1/4/2026 12:00pm EST).

Running a simple KQL query, it tells us that we had exactly 18,344 failed login attempts over the 13 hour window.

>Even though the query is searching for results from the past 24 hours, it doesn't matter because this was the first time the VM was running, open to the Internet.

![1](/assets/img/azure-soc/baselining/1.png){: width="600" height="400" }

This query gives us an estimate of the number of failed login attempts per hour, which ended up being ~1,411 

At that rate, if I left the VM running for 24 hours, we could have 33,864 failed login attempts.

![2](/assets/img/azure-soc/baselining/2.png){: width="600" height="400" }

Next, we will get the top 10 attacking countries

![3](/assets/img/azure-soc/baselining/3.png){: width="600" height="400" }

More data

2 IPs are generating over 11,000 (~60%) of the alerts

5 IPs have over 1,000 failures

That is way too many failed login attempts for a single source.

![4](/assets/img/azure-soc/baselining/4.png){: width="600" height="400" }

## Next: [RDP Block Automation](/posts/rdp-block-automation)