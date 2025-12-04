---
title: "Ubuntu Server VM Setup"
date: 2025-11-30 00:00:00 -0800
categories: [Cybersecurity Lab, SOC Automation]
tags: [cybersecurity, homelab, automation]
description: Ubuntu Server VM installation and setup.
pin: false
---
I am using Ubuntu Server (24.04.03 LTS): [https://ubuntu.com/download/server](https://ubuntu.com/download/server)

On VirtualBox, create a new VM

This one is for TheHive, so name it accordingly

Select the Ubuntu server ISO

![1](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/1.png){: width="800" height="400" }

As per documentation, TheHive recommends at least 16GB and 6 CPUs for stable performance - [https://docs.strangebee.com/thehive/installation/system-requirements/](https://docs.strangebee.com/thehive/installation/system-requirements/)

>TheHive relies on Cassandra and Elasticsearch which runs on JVM, thus the high memory requirement
{: .prompt-info }

![2](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/2.png){: width="800" height="400" }

I set the storage to 100 GB

![3](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/3.png){: width="800" height="400" }

Launch the VM (do not change any network configurations yet)

Select a language

![4](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/4.png){: width="800" height="400" }

Leave Ubuntu server checked

![5](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/5.png){: width="800" height="400" }

Leave this at the default setting 

![6](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/6.png){: width="800" height="400" }

Leave blank, next

![7](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/7.png){: width="800" height="400" }

Wait for the mirror to pass the tests, then go next

![8](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/8.png){: width="800" height="400" }

Leave at default settings, next

![9](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/9.png){: width="800" height="400" }

Select done, and continue

![10](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/10.png){: width="800" height="400" }

Give the server a name and create a user 

We will use this user’s credentials to SSH into the VM

![11](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/11.png){: width="800" height="400" }

Continue

![12](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/12.png){: width="800" height="400" }

Make sure to check “Install OpenSSH server” before proceeding

![13](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/13.png){: width="800" height="400" }

Don't select anything, next

![14](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/14.png){: width="800" height="400" }

Click reboot now and shut down the VM

![15](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/15.png){: width="800" height="400" }

Right click the **VM > Settings**

Under **System**, make sure hard disk is at the top of the boot order

![16](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/16.png){: width="650" height="400" }

Under **Network**, change Adapter 1 to internal network (cyberlab-servers)

![17](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/17.png){: width="650" height="400" }

Launch the VM

Log in as the user you created and run ```sudo nano /etc/netplan/50-cloud-init.yaml```

![18](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/18.png){: width="400" height="400" }

These are the network configurations for my TheHive VM

I gave the VM the IP address: 10.10.1.40

Save and write your changes (Ctrl + O) and exit (Ctrl + X)

![19](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/19.png){: width="800" height="400" }

Run ```sudo netplan apply```

Run ```ip a``` to confirm the changes went through and run a few pings to make sure the VM can communicate with other VMs in our network

![20](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/20.png){: width="800" height="400" }

SSH into TheHive VM from our Ubuntu desktop VM

>SSH is completely optional, I just find the Ubuntu desktop environment more pleasant
{: .prompt-info }

This is a bit confusing since i named the user accounts “joe” for both the Ubuntu VM and TheHive VM, however note that my shell is now “joe@thehive”.

![21](/assets/img/cyberlab/soc-automation/ubuntu-server-vm/21.png){: width="650" height="400" }

Next: [Installing TheHive Dependencies](/posts/thehive-install)