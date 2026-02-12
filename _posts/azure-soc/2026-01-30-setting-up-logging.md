---
title: "Setting Up Logging for Windows VM in Azure"
date: 2026-01-30 00:00:00 -0800
categories: [Cloud, Azure]
tags: [cloud, azure, soc]
description: Setting up logging for Windows VM in Azure via Log Analytics Workspace
---

By the end of this exercise, our Windows VM will be able to forward logs to Logs Analytics Workspace via the Azure Monitor Agent. We should be able to view all security related logs from the Setninel SIEM.

![1](/assets/img/azure-soc/logging-setup/1.png){: width="800" height="400" }

# Generating Windows Security Logs

If you are still RDP'd to your VM, exit the RDP session and generate a few failed log in attempts.

![2](/assets/img/azure-soc/logging-setup/2.png){: width="500" height="400" }

Then, log back into the VM and head to event viewer.

Notice our failed log in attempts. These events will have the code 4625 (Failed log in).

![3](/assets/img/azure-soc/logging-setup/3.png){: width="800" height="400" }

Additionally, it will show the workstation name and IP address of the device that tried to access the VM. This showed my information, which I blurred out for privacy. This is valuable data for investigations, so we want these logs in a centralized repository.

![4](/assets/img/azure-soc/logging-setup/4.png){: width="800" height="400" }

## Log Analytics Workspace Integration

The Log Analytics Workspace (LAW) will be our centralized data store for logs.

On the Azure portal, head to LAW and click create.

Set a name and make sure you use same RG and location as your subscription.

![5](/assets/img/azure-soc/logging-setup/5.png){: width="800" height="400" }

## Sentinel SIEM Setup

We will use Sentinel to query these logs.

Head to the Sentinel dashboard and click create new workspace.

Select the Log Analytics Workspace and click add.

Once finished, you should see this screen.

![6](/assets/img/azure-soc/logging-setup/6.png){: width="800" height="400" }

## Forwarding Windows Logs to Sentinel

On the Sentinel dashboard, head to **Content management > Content hub**

Look for Windows Security Events and install the plugin

![7](/assets/img/azure-soc/logging-setup/7.png){: width="800" height="400" }

Once it’s finished installing, click manage

![8](/assets/img/azure-soc/logging-setup/8.png){: width="400" height="400" }

Look for **Windows Security Events via AMA** and open click **Open connector page**

![9](/assets/img/azure-soc/logging-setup/9.png){: width="800" height="400" }

Click **Create data collection rule**

This rule will be used by our VM to forward logs to LAW, so we can view them in Sentinel 

![10](/assets/img/azure-soc/logging-setup/10.png){: width="800" height="400" }

Give the rule a name and make sure you are using the same subscription and RG

![11](/assets/img/azure-soc/logging-setup/11.png){: width="800" height="400" }

Select the VM from the hierarchy

![12](/assets/img/azure-soc/logging-setup/12.png){: width="800" height="400" }

Leave to **All security events**

![13](/assets/img/azure-soc/logging-setup/13.png){: width="600" height="400" }

Create

![14](/assets/img/azure-soc/logging-setup/14.png){: width="800" height="400" }

## VIewing Logs on Sentinel

Head to the VM dashboard

Head to **Settings > Extensions + Applications**

Under status for **AzureMonitorWindowsAgent**, it should show “Provisioning succeeded”

![15](/assets/img/azure-soc/logging-setup/15.png){: width="800" height="400" }  

Head to LAW

Go to the **Logs** tab

On the top right, change the mode to **KQL mode** 

Run a test query (`SecurityEvent | where EventID == 4625`)

At this point, generate a failed log in attempt log like we did before to make sure things are working, then leave your VM running overnight. We should see hundreds if not thousands of failed logins overnight

![16](/assets/img/azure-soc/logging-setup/16.png){: width="800" height="400" }

I left my VM running overnight on 1/4/26, from 12am to 12pm. 

I ran a query that looks for failed log in attempts (4625), and I got thousands of logs. Notice the different IP addresses.

Keep in mind the logs use UTC time. 11:47am in UTC is 6:47 AM EST, so it seems like the attacks stopped around that time.

![17](/assets/img/azure-soc/logging-setup/17.png){: width="800" height="400" }

## Next: [Sentinel Map](/posts/sentinel-map)