---
title: "Active Directory: Domain Controller Setup"
date: 2024-12-21 00:00:00 -0800
categories: [Cybersecurity Lab, Initial Setup]
tags: [cybersecurity, homelab]
description: Configuring a Domain Controller VM for our Active Directory environment.
---

### ISO Download & VM Setup

The Windows Server 2025 ISO can be downloaded here: [https://www.microsoft.com/en-us/evalcenter/download-windows-server-2025](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2025)

I chose the English 64-bit ISO Download

> Windows Server 2022 may be a better option since Windows 11 requires better hardware specs (more RAM, CPU, storage, etc.)
{: .prompt-info }

On VirtualBox, head to **Machine > New**

Name the VM

Give it 4 GB of RAM

2 CPUs

50GB Virtual Disk

Skip unattended installation

> I originally used 2 GB of RAM for this VM, but I kept getting crashes and black screens. 4 GB is more stable.
{: .prompt-warning }

![1](/assets/img/cyberlab/setup/domain_controller/1.png){: width="700" height="400" }

Attach Adapter 1 to Internal Network (cyberlab-servers)

![2](/assets/img/cyberlab/setup/domain_controller/2.png){: width="400" height="200" }

Launch the VM

Select Install Windows Server

![3](/assets/img/cyberlab/setup/domain_controller/3.png){: width="600" height="400" }

Select **Windows 2025 Standard Evaluation (Desktop Experience)**. This includes the GUI and makes the setup a lot easier and user friendly.

![4](/assets/img/cyberlab/setup/domain_controller/4.png){: width="600" height="400" }

Accept the license

> This license will expire after 90 days.
{: .prompt-info }

![5](/assets/img/cyberlab/setup/domain_controller/5.png){: width="600" height="400" }

Select Disk 0

![6](/assets/img/cyberlab/setup/domain_controller/6.png){: width="600" height="400" }

Install

![7](/assets/img/cyberlab/setup/domain_controller/7.png){: width="600" height="400" }

This should take around 15 minutes

![9](/assets/img/cyberlab/setup/domain_controller/9.png){: width="600" height="400" }

When the installation finishes and it prompts you to reboot, shut off the VM

Make sure the Hard Disk is first in the boot order

![10](/assets/img/cyberlab/setup/domain_controller/10.png){: width="600" height="400" }

Reboot the VM

![11](/assets/img/cyberlab/setup/domain_controller/11.png){: width="600" height="400" }

Enter a password and take note of it

Click on Finish

> This is for the built in Administrator account. We can create another Domain Controller admin account later.
{: .prompt-info }

![12](/assets/img/cyberlab/setup/domain_controller/12.png){: width="600" height="400" }

We should now see the login screen 

Go to **Input > Keyboard > Send Ctrl+Alt+Dlt** and log in

![13](/assets/img/cyberlab/setup/domain_controller/13.png){: width="600" height="400" }

### Guest Additions

Installing the Guest Additions will give us a smoother experience and allow us to adjust the resolution of the VM seamlessly.

On top of the VM, click on **Devices > Insert Guest Additions CD image**

Open File Explorer and head to **D:\VirtualBox Guest Additions**

Run the **VBoxWindowsAdditions.amd64** program

Install and reboot

![14](/assets/img/cyberlab/setup/domain_controller/14.png){: width="900" height="400" }

Install and reboot now

![15](/assets/img/cyberlab/setup/domain_controller/15.png){: width="600" height="400" }

### Computer Name & IP Configuration

Once we’re in, click on **Settings > System > Rename**

Rename the system (I recommend naming this computer **DC01**)

The VM will have to restart for the changes to take place

> The max length for a Windows NetBIOS name is 15 characters. Additionally, NetBIOS will capitalize the entire computer name. The name DomainController will be converted into DOMAINCONTROLLE for NetBIOS. This may pose problems with the Active Directory set up so I recommend using a name like DC01.
{: .prompt-warning }

![16](/assets/img/cyberlab/setup/domain_controller/16.png){: width="600" height="400" }

Go to **Settings > Network & internet > Ethernet**

Edit IP assignment

![17](/assets/img/cyberlab/setup/domain_controller/17.png){: width="600" height="400" }

Set the IP settings to Manual

IPv4 should be on

I gave this VM the IP address 10.10.1.10/24

The gateway and preferred DNS should be 10.10.1.1

Save and confirm settings

![18](/assets/img/cyberlab/setup/domain_controller/18.png){: width="600" height="400" }

Open command prompt and run ipconfig

We should be able to ping the gateway and DNS server

![19](/assets/img/cyberlab/setup/domain_controller/19.png){: width="900" height="400" }

### Configuring Active Directory Services

Open Server Manager

Click on **Manage > Add Roles and Features**

![20](/assets/img/cyberlab/setup/domain_controller/20.png){: width="500" height="300" }

Select Role-based or feature based installation

Next

![21](/assets/img/cyberlab/setup/domain_controller/21.png){: width="600" height="400" }

Make sure the correct computer name and IP address  appear in the server pool.

Next

![22](/assets/img/cyberlab/setup/domain_controller/22.png){: width="600" height="400" }

Click Add Features

![23](/assets/img/cyberlab/setup/domain_controller/23.png){: width="600" height="400" }

Select Active Directory Domain Services

Next

![24](/assets/img/cyberlab/setup/domain_controller/24.png){: width="600" height="400" }

.NET Framework 4.8 Features and Group Policy Management should be checked by default

Next

![25](/assets/img/cyberlab/setup/domain_controller/25.png){: width="600" height="400" }

Next

![26](/assets/img/cyberlab/setup/domain_controller/26.png){: width="600" height="400" }

Click Install

![27](/assets/img/cyberlab/setup/domain_controller/27.png){: width="600" height="400" }

When the installation is finished, click on the warning sign on the top right

Click on Promote this server to a domain controller

![28](/assets/img/cyberlab/setup/domain_controller/28.png){: width="600" height="300" }

Select Add a new forest

Add a domain name (cyber.lab in my case)

Next

![29](/assets/img/cyberlab/setup/domain_controller/29.png){: width="600" height="400" }

Leave the forest and domain functional levels at the default (Windows Server 2016)

Make sure DNS server and Global Catalog (GC) are checked

Set a DSRM password. Take note of it just in case

Next

![30](/assets/img/cyberlab/setup/domain_controller/30.png){: width="600" height="400" }

Don’t worry about this error message, click Next

![31](/assets/img/cyberlab/setup/domain_controller/31.png){: width="600" height="400" }

Set a NetBIOS domain name (CYBER)

Next

![32](/assets/img/cyberlab/setup/domain_controller/32.png){: width="600" height="400" }

Leave at default again

Next

![33](/assets/img/cyberlab/setup/domain_controller/33.png){: width="600" height="400" }

Next

![34](/assets/img/cyberlab/setup/domain_controller/34.png){: width="600" height="400" }

Click Install

![35](/assets/img/cyberlab/setup/domain_controller/35.png){: width="600" height="400" }

The VM may automatically restart

Log back in

![36](/assets/img/cyberlab/setup/domain_controller/36.png){: width="900" height="400" }

### Adding Activer Directory Users

When we’re back in the server manager, click on Tools > AD Users and Computers

![37](/assets/img/cyberlab/setup/domain_controller/37.png){: width="400" height="200" }

Right click the domain > New > Organizational Unit

![38](/assets/img/cyberlab/setup/domain_controller/38.png){: width="600" height="400" }

We will create the Admins OU

Name it accordingly and click OK

![39](/assets/img/cyberlab/setup/domain_controller/39.png){: width="400" height="200" }

Right click **Admins > New User**

Give the new admin a name (Trent)

![40](/assets/img/cyberlab/setup/domain_controller/40.png){: width="600" height="400" }

Set a password. Make sure you take note of these. Windows 10 and up have strong password requirements on by default and it could be difficult to remember them for all of your VMs

Set the password to never expire

Next

![41](/assets/img/cyberlab/setup/domain_controller/41.png){: width="600" height="400" }

You may add a Description to the new user (DC Admin)

![42](/assets/img/cyberlab/setup/domain_controller/42.png){: width="600" height="400" }

Right click on the new **Admin > Properties > Member Of**

Click Add

![43](/assets/img/cyberlab/setup/domain_controller/43.png){: width="400" height="250" }

Enter **Domain Admins**

Click Check Names

OK

![44](/assets/img/cyberlab/setup/domain_controller/44.png){: width="500" height="300" }

Apply

![45](/assets/img/cyberlab/setup/domain_controller/45.png){: width="500" height="300" }

We will now add our 2 clients, which should be regular users

Right Click on **Users > New > User**

![46](/assets/img/cyberlab/setup/domain_controller/46.png){: width="500" height="300" }

Enter a password and set it so that it never expires

Next

![47](/assets/img/cyberlab/setup/domain_controller/47.png){: width="500" height="300" }

I created 2 users, Bob and Alice

![48](/assets/img/cyberlab/setup/domain_controller/48.png){: width="600" height="400" }

### **Next**: [Active Directory: Client Setup](/posts/ad-client)