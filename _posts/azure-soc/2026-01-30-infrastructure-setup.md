---
title: "Infrastructure Setup"
date: 2026-01-30 00:00:00 -0800
categories: [Cloud, Azure]
tags: [cloud, azure, soc]
description: Simple lab environment setup for Azure
---

This is a very simple envrionment, so I am not going to go step-by-step for creating the resource group (RG) and VM, as these are very straightforward processes.

By the end of this exercise, we will have a resource group, virtual network, a network security group (NSG), and a VM within that virtual network. These resources are all under the same subscription. The VM will be open to ALL traffic, so any endpoint should be able to RDP into it. 

![1](/assets/img/azure-soc/infrastructure-setup/1.png){: width="800" height="400" }

## Checkpoint 1 - Resource Group Created

I have the region for this RG set to US East 2, so all other resources must be in the same region.

![2](/assets/img/azure-soc/infrastructure-setup/2.png){: width="800" height="400" }

## Checkpoint 2 - Virtual Network Created

I left everything at default settings.

Make sure to select the appropriate RG (*SOC-Lab*), Azure subscription and region (East US 2)

![3](/assets/img/azure-soc/infrastructure-setup/3.png){: width="800" height="400" }

## Checkpoint 3 - VM Created

I chose a Windows 10 Enterprise VM.

Again, make sure to select the appropriate RG (*SOC-Lab*), Azure subscription and region (East US 2).

>When creating the user, set the password to something strong. We do not want full compromise; we do not want attackers actually getting access to the VM. The purpose of this lab is to generate logs/telemetry. Keep note of this password somewhere safe.
{: .prompt-danger }

![4](/assets/img/azure-soc/infrastructure-setup/4.png){: width="800" height="400" }

Additional VM details for reference. 

![5](/assets/img/azure-soc/infrastructure-setup/5.png){: width="800" height="400" }

After creating the VM, a public IP, NIC, NSG and disk resources will automatically be created.

![6](/assets/img/azure-soc/infrastructure-setup/6.png){: width="800" height="400" }

## Network Security Group (NGS)

We will create a rule that allows ALL ingress traffic

Rule configuration:

![7](/assets/img/azure-soc/infrastructure-setup/7.png){: width="600" height="400" }

Our inbound rules should now look like this

![8](/assets/img/azure-soc/infrastructure-setup/8.png){: width="800" height="400" }

## RDP Test

RDP from your host PC, using the VM’s public IP address.

![9](/assets/img/azure-soc/infrastructure-setup/9.png){: width="800" height="400" }

Turn off the firewall for all profiles (domain, private, public)

![10](/assets/img/azure-soc/infrastructure-setup/10.png){: width="800" height="400" }

## Sanity Check

Afterwards, make sure you can ping the VM (from your host PC).

If you can ping it, then every single device on the Internet will be able to as well.

![11](/assets/img/azure-soc/infrastructure-setup/11.png){: width="600" height="400" }

## Next: [Setting Up Logging](/posts/setting-up-logging)