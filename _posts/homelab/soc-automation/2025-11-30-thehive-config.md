---
title: "TheHive Configuration "
date: 2025-11-30 00:00:00 -0800
categories: [Cybersecurity Lab, SOC Automation]
tags: [cybersecurity, homelab, automation]
description: Configuring our TheHive VM
pin: false
---
### Cassandra
Run ```sudo nano /etc/cassandra/cassandra.yaml```

Change **cluster_name** to anything you'd like (I changed mine to ‘cyberlab’)

![1](/assets/img/cyberlab/soc-automation/thehive-config/1.png){: width="800" height="400" }

Look for “listen_address”

>You can use Ctrl + W to look for a specific string in the nano text editor
{: .prompt-info }

Change this value to the IP address of TheHive VM (**10.10.1.40** in my case)

![2](/assets/img/cyberlab/soc-automation/thehive-config/2.png){: width="800" height="400" }

Look for "rpc_address"

Change this value to the IP address of TheHive VM

![3](/assets/img/cyberlab/soc-automation/thehive-config/3.png){: width="800" height="400" }

Then, look for "seed_provider"

Under **seeds**, set the value to the IP address of the TheHive VM. Set the port to 7000

![4](/assets/img/cyberlab/soc-automation/thehive-config/4.png){: width="800" height="400" }

Save your changes (Ctrl + O) and stop the cassandra service

Run ```sudo systemctl stop cassandra.service```

Then, run ```sudo rm -rf /var/lib/cassandra/*```

Start the service again: ```sudo systemctl start cassandra.service```

Run ```sudo systemctl status cassandra.service``` to make sure it is running

![5](/assets/img/cyberlab/soc-automation/thehive-config/5.png){: width="800" height="400" }

### Elasticsearch

Run ```sudo nano /etc/elasticsearch/elasticsearch.yml```

Uncomment **cluster.name**, set a name (cyberlab)

Uncomment **node.name**, leave at default (node-1)

![6](/assets/img/cyberlab/soc-automation/thehive-config/6.png){: width="800" height="400" }

Set **network.host** to the IP of TheHive VM

Uncomment **http.port**, leave it at 9200

Uncomment **cluster.initial_master_nodes** and remove node-2 (only node-1 should be in the array)

![7](/assets/img/cyberlab/soc-automation/thehive-config/7.png){: width="800" height="400" }

Save and exit

Commands to run:

```sudo systemctl start elasticsearch```

```sudo systemctl enable elasticsearch```

```sudo systemctl status elasticsearch```

![8](/assets/img/cyberlab/soc-automation/thehive-config/8.png){: width="800" height="400" }

### TheHive

Run ```cd /opt/thp```

Change the ownership of this directory, it should be owned by the thehive user and group: ```sudo chown -R thehive:thehive /opt/thp```

Run ```ll``` to make sure the ownership changed

![9](/assets/img/cyberlab/soc-automation/thehive-config/9.png){: width="700" height="400" }

Then, run ```sudo nano /etc/thehive/application.conf```

In db.janusgraph, 

under **storage**, change the value of **hostname** to the IP of the VM 

under **cql**, change **clustername** to any name

under **index.search**, change the value of **hostname** to the IP of the VM

![10](/assets/img/cyberlab/soc-automation/thehive-config/10.png){: width="800" height="400" }

under service configuration, change **application.baseUrl** to "http://thehiveVMIP:9000”

![11](/assets/img/cyberlab/soc-automation/thehive-config/11.png){: width="600" height="400" }

Save and write changes

Commands to run:

```sudo systemctl start thehive```

```sudo systemctl enable thehive```

```sudo systemctl status thehive```

![12](/assets/img/cyberlab/soc-automation/thehive-config/12.png){: width="800" height="400" }

Access TheHive at http://10.10.1.40:9000 (or whatever the IP of your TheHive VM is)

![13](/assets/img/cyberlab/soc-automation/thehive-config/13.png){: width="800" height="400" }

If you see "Unable to connect", run this command: ```sudo ufw allow 9000```

TheHive listens on all network interfaces ( 0.0.0.0 ) at port 9000

![14](/assets/img/cyberlab/soc-automation/thehive-config/14.png){: width="600" height="400" }

Wait a few minutes and the page should load

Log in in using the default credentials

**Email**: admin@thehive.local

**Password**: secret

![15](/assets/img/cyberlab/soc-automation/thehive-config/15.png){: width="800" height="400" }

We should see this dashboard

>TheHive has a free trial period of 14 days (mine was 16 for some reason) so make sure to complete this project within the time frame.
{: .prompt-info }
![16](/assets/img/cyberlab/soc-automation/thehive-config/16.png){: width="800" height="400" }

Next: [Shuffle Installation via Docker](/posts/shuffle-docker)