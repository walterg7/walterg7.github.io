---
title: "Cybersecurity Lab - Web App Services Setup"
date: 2026-01-09 00:00:00 -0800
categories: [Cybersecurity Lab]
tags: [cybersecurity, homelab]
description: WebServer, MySQL, RabbitMQ, and Data Processor VMs set up + port forwarding rule for the webserver.
---

By the end of this exercise, we will have:
- Wazuh VM set up
- Wazuh agents deployed on both Windows machines and the Ubuntu VM

## DMZ Network Setup

On VMware Workstation, click **Edit > Virtual Network Editor**

Click **Change Settings** (administrator privileges are required to change network settings)

Create the DMZ network. Give the network the address 10.10.4.0.

- **VMnet Information**: Host-only (connect VMs internally in a private network)
- Uncheck **Connect a host virtual adapter to this network** and uncheck **DHCP service**

![1](/assets/img/cyberlab2/setup/webapp-setup/1.png){: width="600" height="400" }

Shut off the OPNsense VM if it is running, and add a new network adapter to the OPNsense VM. Set it to the newly created DMZ network. This will be its 5th adapter, so we should see em4 on the web interface.

![2](/assets/img/cyberlab2/setup/webapp-setup/2.png){: width="800" height="400" }

Assign em4 to opt3. Rename this interface to DMZ and assign it the network address 10.10.4.0/24. Follow the steps from [Network Setup](/posts/network-setup/#network-interface-assignment) if you are unsure how to do this.

![3](/assets/img/cyberlab2/setup/webapp-setup/3.png){: width="800" height="400" }

### DMZ Firewall Rules

Head to **Firewall > Rules > DMZ**

I am purposefully using open firewall rules for the sake of testing.

These are the rules:

- DMZ net → Any (allow DMZ to access the open Internet)
- WAN net → DMZ net (my home network will be able to reach the DMZ via port forwarding, this will be configured later)
- DMZ net → Servers net (DMZ needs to communicate with MySQL, RabbitMQ, and data processor VMs)

![4](/assets/img/cyberlab2/setup/webapp-setup/4.png){: width="800" height="400" }

## App Services Setup

This is mainly for my own reference, but feel free to follow along.

If you are interested, this is the code for the app: [https://github.com/walterg7/systems-integration](https://github.com/walterg7/systems-integration)

For now, all the services will run on separate Ubuntu server VMs. [Ubuntu Server VM installation guide](/posts/ubuntu-server-vm)

### Web Server VM + Apache Setup

I gave this VM 2GB of RAM, 1 Processor and 25GB storage. This VM is only connected to the DMZ network.

![5](/assets/img/cyberlab2/setup/webapp-setup/5.png){: width="800" height="400" }

When first booting the Ubuntu server VM, it will not have an IP. Edit the network config file at `/etc/netplan/50-cloud-init.yaml`

![6](/assets/img/cyberlab2/setup/webapp-setup/6.png){: width="800" height="400" }

Follow this YAML format for the network configuration. I gave this VM the IP address 10.10.4.10. Run some ping tests to make sure connectivity works. 

![7](/assets/img/cyberlab2/setup/webapp-setup/7.png){: width="800" height="400" }

After giving the Web Server VM an IP, I SSH’d into it from my Ubuntu desktop VM. 

![8](/assets/img/cyberlab2/setup/webapp-setup/8.png){: width="800" height="400" }

The Web Server VM will need the following packages:

- apache2
- git
- php
- php-amqp
- php-amqplib
- composer

First, clone the repository. The repository directory will be `/home/prod/systems-integration`

Copy `WebServer/001-webserver.conf` into `/etc/apache2/sites-available`

![9](/assets/img/cyberlab2/setup/webapp-setup/9.png){: width="800" height="400" }

Copy the WebServer, RabbitMQ, Logger, and vendor directories into /var/www 

![10](/assets/img/cyberlab2/setup/webapp-setup/10.png){: width="600" height="400" }

I symlinked `/var/www/WebServer` with `/home/prod/systems-integration` .

![11](/assets/img/cyberlab2/setup/webapp-setup/11.png){: width="600" height="400" }

Head to `/etc/apache2/sites-enabled`. Delete the default config (000-default.conf) and add a symlink to our config (001-webserver.conf)

![12](/assets/img/cyberlab2/setup/webapp-setup/12.png){: width="800" height="400" }

Optional: edit `/etc/hosts` on the Ubuntu VM add an entry for `www.crypto.com`, mapping it to the IP address of the Web Server VM.

![13](/assets/img/cyberlab2/setup/webapp-setup/13.png){: width="600" height="400" }

Edit `/etc/apache2/apache2.conf`

Make sure **Directory** points to `/var/www/WebServer`

![14](/assets/img/cyberlab2/setup/webapp-setup/14.png){: width="800" height="400" }

Finally, run `sudo service apache2 reload` on the Web Server VM.

Notice that when I ping www.crypto.com, it resolves the domain name to the Web Server VM.

![15](/assets/img/cyberlab2/setup/webapp-setup/15.png){: width="800" height="400" }

Open Firefox and clear the cache and cookies.

We should now see our custom site. If not, we may have to restart our VM.

![16](/assets/img/cyberlab2/setup/webapp-setup/16.png){: width="800" height="400" }

### RabbitMQ VM

VM Settings:

![17](/assets/img/cyberlab2/setup/webapp-setup/17.png){: width="800" height="400" }

Once you created a new Ubuntu server VM and assigned it an IP address, install the following via apt

- git
- php
- php-amqp
- php-amqplib
- rabbitmq-server

Again, make sure to clone the repository.

Run `sudo rabbitmq-plugins enable rabbitmq_management`

#### RabbitMQ Setup

Create a new admin user by running the following commands:

`sudo rabbitmqctl add_user rbmq-admin admin`

`sudo rabbitmqctl set_user_tages rbmq-admin administrator`

`sudo rabbitmqctl set_permissions -p / rbmq-admin “.*” “.*” “.*”` 

These commands create an admin user **rbmq-admin** that has the password **admin**, with the administrator tags and permissions on all virtual hosts.

![18](/assets/img/cyberlab2/setup/webapp-setup/18.png){: width="800" height="400" }

Head to the RabbitMQ web interface at 10.10.1.20:15672

Log in with the new credentials.

Create a new virtual host.

![19](/assets/img/cyberlab2/setup/webapp-setup/19.png){: width="800" height="400" }

#### Exchanges Setup

Create an exchange

i am going to create the **logger** exchange first to test that the VMs can communicate with each other via RabbitMQ. Make sure to setup the **database** and **dmz** exchanges as well.

On the top right hand side drop down menu, change the vhost to RBMQ

Head to **Exchanges**

The logger exchange must be set to **fanout**

Leave durability to **durable**

Create

![20](/assets/img/cyberlab2/setup/webapp-setup/20.png){: width="800" height="400" }

Head to **Queues and Streams**

Create a queue. I set the type to **Classic** and durability to **Durable**.

![21](/assets/img/cyberlab2/setup/webapp-setup/21.png){: width="800" height="400" }

Click on the new queue

![22](/assets/img/cyberlab2/setup/webapp-setup/22.png){: width="800" height="400" }

Scroll down 

Under bindings, bind the queue to the **logger** exchange using the routing key *

![23](/assets/img/cyberlab2/setup/webapp-setup/23.png){: width="600" height="400" }

### MySQL VM

VM settings:

![24](/assets/img/cyberlab2/setup/webapp-setup/24.png){: width="800" height="400" }

Once you created a new Ubuntu server VM and assigned it an IP address, install the following via apt:

- git
- php
- php-amqp
- php-amqplib
- php-mysql
- mysql-server

Once again, make sure to clone the repository.

Use MySQL from the command line: `sudo mysql`

Run the following commands to create a new database and user:

```sql
--create db and user
create database cyberlab;

--user (WARNING: make sure you are using straight single quotes: ')
create user 'cryptoAdmin'@localhost identified by 'admin';

--giving privs
grant all privileges on cyberlab.* to 'cryptoAdmin'@localhost;

flush privileges;
```

Run the database schema setup command

![25](/assets/img/cyberlab2/setup/webapp-setup/25.png){: width="800" height="400" }

Log in as the new user and check out the new DB.

![26](/assets/img/cyberlab2/setup/webapp-setup/26.png){: width="800" height="400" }

The crypto table needs to be populated via the API.

![27](/assets/img/cyberlab2/setup/webapp-setup/27.png){: width="800" height="400" }

### Datasource Processor VM

VM Settings:

![28](/assets/img/cyberlab2/setup/webapp-setup/28.png){: width="800" height="400" }

Once you created a new Ubuntu server VM and assigned it an IP address, install the following via apt

- git
- php
- php-amqp
- php-amqplib
- composer

Once again, make sure to clone the repository.

Before running any scripts, make sure `RabbitMQ.ini` is using the correct configurations (on all VMs).

From the Data processor VM, run the `dmz_handler` script.

![29](/assets/img/cyberlab2/setup/webapp-setup/29.png){: width="600" height="400" }

On the MySQL VM, run the `getCryptoData` script.

Check the crypto table, it should now be populated.

![30](/assets/img/cyberlab2/setup/webapp-setup/30.png){: width="800" height="400" }

User Registration working

![31](/assets/img/cyberlab2/setup/webapp-setup/31.png){: width="800" height="400" }

User info added to DB

![32](/assets/img/cyberlab2/setup/webapp-setup/32.png){: width="800" height="400" }

Now the web app is fully functional.

![33](/assets/img/cyberlab2/setup/webapp-setup/33.png){: width="800" height="400" }

## Port Forwarding

Create a Kali Linux VM. Make sure it is sitting at the 192.168.1.0/24 network. [Kali Linux VM installation guide](/posts/kali)

![34](/assets/img/cyberlab2/setup/webapp-setup/34.png){: width="800" height="400" }

Keep in mind that devices on the home network can communicate with the OPNsense VM solely because it is placed on the same network (192.168.1.0/24). Hosts on the home network cannot directly reach the WebServer using its IP address 10.10.4.10, so we must redirect the traffic from OPNsense to the WebServer via port forwarding. 

Head to **Firewall > NAT > Port Forward**

Create new rule

**Interface**: WAN

**TCP/IP Version**: IPv4

**Protocol**: TCP

**Source**: 192.168.1.0/24

**Source port range**: any

**Destination**: WAN address

![38](/assets/img/cyberlab2/setup/webapp-setup/38.png){: width="800" height="400" }

**Destination port range**: HTTP

**Redirect target IP:** 10.10.4.10

**Redirect target port:** HTTP

![39](/assets/img/cyberlab2/setup/webapp-setup/39.png){: width="800" height="400" }

Save and apply

![40](/assets/img/cyberlab2/setup/webapp-setup/40.png){: width="800" height="400" }

Next, we need a rule that explicitly allows traffic originating from the home network to be passed to the DMZ.

On the OPNsense web interface, head to **Firewall > Rules > WAN**

Create a new rule

**Action**: Pass

**Interface**: WAN

**Direction**: in

**TCP/IP Verion**: IPv4

**Protocol**: TCP

**Source**: 192.168.1.0/24

![35](/assets/img/cyberlab2/setup/webapp-setup/35.png){: width="600" height="400" }

**Destination**: 10.10.4.10

**Destination port range**: HTTP

**Description**: Allow WAN to DMZ web traffic

![36](/assets/img/cyberlab2/setup/webapp-setup/36.png){: width="800" height="400" }

Save and apply the changes

![37](/assets/img/cyberlab2/setup/webapp-setup/37.png){: width="800" height="400" }

Finally, head to **Interfaces > WAN** 

Under generic configuration, uncheck **Block private networks** and **Block bogon networks**. 192.168.1.0/24 is indeed a private address range, however in my lab setup, we consider it an external network (from the internal VMs' perspective). We will need to uncheck this setting in order for devices on the home network to actually be able to reach the web server. Otherwise, all traffic originating from 192.168.1.0/24 will be blocked, no matter what firewall rules we have in place.

![41](/assets/img/cyberlab2/setup/webapp-setup/41.png){: width="800" height="400" }

Now from Kali Linux, when i enter 192.168.1.167 (OPNsense WAN IP), I am redirected to the web server landing page.

![42](/assets/img/cyberlab2/setup/webapp-setup/42.png){: width="800" height="400" }

I am even able to login and use the site.

![43](/assets/img/cyberlab2/setup/webapp-setup/43.png){: width="800" height="400" }

Testing: accessing other web pages. 

![44](/assets/img/cyberlab2/setup/webapp-setup/44.png){: width="800" height="400" }