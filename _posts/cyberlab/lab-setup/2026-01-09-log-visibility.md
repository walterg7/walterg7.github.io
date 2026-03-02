---
title: "Security Onion Log Integrations"
date: 2026-01-09 00:00:00 -0800
categories: [Cybersecurity Lab]
tags: [cybersecurity, homelab]
description: Ingesting OPNsense and NetFlow logs for Security Onion
---
Out of the box, Security Onion has solid monitoring capabilities, however we can take advantage of its integrations (specifcally, the pfSense/OPNsense and NetFlow integrations) for better network visiblity.
Keep in my mind that in my setup, Wazuh is used to collect endpoint-based telemetry. We will not use Security Onion's Elastic Agents. 

By the end of this exercise, we will have:
- OPNsense log ingestion (traffic traversing the internal network)
- NetFlow log ingestion (IP packet metadata)

## OPNsense Log Ingestion

Head to the Security Onion web interface and click **Elastic Fleet** from the lefthand menu

Click the **Agent policies** tab

Click **so-grid-nodes_general**

![1](/assets/img/cyberlab2/setup/log-visibility/1.png){: width="800" height="400" }

Click add integration

Look for **pfSense** (the name says pfSense, but this works with OPNsense as well)

![2](/assets/img/cyberlab2/setup/log-visibility/2.png){: width="800" height="400" }

Click **Add pfSense**

![3](/assets/img/cyberlab2/setup/log-visibility/3.png){: width="800" height="400" }

Change **sysloghost** to 0.0.0.0 so we can listen on all interfaces, leave everything else at default.

Save and continue

![4](/assets/img/cyberlab2/setup/log-visibility/4.png){: width="600" height="400" }

On the Security Onion web interface, head to **Administration > Configuration**

Head to **Firewall > hostgroups > customhostgroup0**

Under current grid value, enter the IP of OPNsense (10.10.3.1)

Click the green checkmark

![5](/assets/img/cyberlab2/setup/log-visibility/5.png){: width="800" height="400" }

Next, head to **Firewall > portgroups > customportgroup0 > udp** 

Set the grid value to 9001

Click the green checkmark

![6](/assets/img/cyberlab2/setup/log-visibility/6.png){: width="800" height="400" }

Next, head to **firewall > role > standalone > chain > INPUT > hostgroups > customhostgroup0 > portgroups** 

Under grid value, enter **customportgroup0**

Click the green checkmark

![7](/assets/img/cyberlab2/setup/log-visibility/7.png){: width="800" height="400" }

Click synchronize firewall then synchronize grid

![8](/assets/img/cyberlab2/setup/log-visibility/8.png){: width="800" height="400" }

Next, head to the OPNsense web interface.

Head to **System > Settings > Logging**. Click on the **Remote** tab. Click Add

**Transport**: UDP(4)

**Applications:** select all for sake of testing

**Levels**: select all for sake of testing

**Facilities**: locally used (0)

**Hostname**: 10.10.3.3

**Port**: 9001

**rfc5424**: Checked

Save

![9](/assets/img/cyberlab2/setup/log-visibility/9.png){: width="800" height="400" }

After a few minutes, head back to the Security Onion web interface and head to **Hunt**

We should see a pfSense dataset

**Left click > Include** to see all pfsense logs

![10](/assets/img/cyberlab2/setup/log-visibility/10.png){: width="800" height="400" }

We should now see firewall traffic. Here, it looks like our VMs are reaching the Wazuh VM (presumably the agent connection)

![11](/assets/img/cyberlab2/setup/log-visibility/11.png){: width="800" height="400" }

More logs

![12](/assets/img/cyberlab2/setup/log-visibility/12.png){: width="800" height="400" }

I adjusted the logging rule so we do not get too much data

![13](/assets/img/cyberlab2/setup/log-visibility/13.png){: width="800" height="400" }

I also enabled logging for the port forwarding rule (notice the blue i icon)

![14](/assets/img/cyberlab2/setup/log-visibility/14.png){: width="800" height="400" }

I disabled IPv6 on all interfaces so we don’t get IPv6 traffic or logs.

![15](/assets/img/cyberlab2/setup/log-visibility/15.png){: width="800" height="400" }

## NetFlow Log Ingestion

On OPNsense, head to **Reporting > NetFlow**

This is my configuration. It is basically listening on all interfaces and reporting to the Security Onion VM at port 2055. However, we still need to configure Security Onion’s side to receive the data.

![16](/assets/img/cyberlab2/setup/log-visibility/16.png){: width="600" height="400" }

This process is very similar to the OPNsense integration.

Click Elastic Fleet

Click the **Agent policie**s tab

click **so-grid-nodes_general**

Click add integration

Look for NetFlow Records

![17](/assets/img/cyberlab2/setup/log-visibility/17.png){: width="800" height="400" }

Listen on all interfaces (0.0.0.0)

Make sure the UDP port is 2055

![18](/assets/img/cyberlab2/setup/log-visibility/18.png){: width="600" height="400" }

Leave at default

![19](/assets/img/cyberlab2/setup/log-visibility/19.png){: width="600" height="400" }

Make sure show advanced options checked

![20](/assets/img/cyberlab2/setup/log-visibility/20.png){: width="600" height="400" }

On the Security Onion web interface, head to **Administration > Configuration**

Head to **Firewall > portgroups> customhostgroup0 > udp**

Under current grid value, add 2055. Since we have multiple ports, separate them by comma (no spaces)

![21](/assets/img/cyberlab2/setup/log-visibility/21.png){: width="600" height="400" }

Head to Hunt, and filter for the netflow dataset. We should see lots of logs.

![22](/assets/img/cyberlab2/setup/log-visibility/22.png){: width="800" height="400" }

## Monitoring Interface Verification (Zeek)

If for some reason you did not set up a monitor interface on Security Onion, follow these steps:

1. Shut down the Security Onion VM and attach a new network adapter for the network to be monitored. In this case, I attached it to DMZ network. Reboot the VM.
2. Run `ip a`.  The DMZ adapter is named `ens192`.
3. Run `sudo so-monitor-add ens192` to assign this interface as the monitoring interface. Notice that the adapter is set to promiscuous mode by default. This adapter should NOT have an IP address.

![23](/assets/img/cyberlab2/setup/log-visibility/23.png){: width="800" height="400" }

Generate some traffic on the web server VM, then head to Hunt on Security Onion and filter for the zeek dataset.

We should see some logs.

![24](/assets/img/cyberlab2/setup/log-visibility/24.png){: width="800" height="400" }

## Fixing Unbound DNS

In my setup, there was tens of thousands of zeek.dns logs, pointing to `ntp.ubuntu.com`

### Explanation

- WebServer (10.10.4.10) is trying to sync time
- Ubuntu resolves `ntp.ubuntu.com`
- OPNsense (10.10.4.1) is acting as DNS forwarder
- DNS replies are failing (`SERVFAIL`)
- Zeek logs every attempt

Sample log

```
orig_h:10.10.4.10   (WebServer)
resp_h:10.10.4.1    (OPNsense)
query: ntp.ubuntu.com
rcode: SERVFAIL
rejected:true
```

This is unnecessary noise that’s happening because:

- No upstream DNS properly configured
- No NTP server reachable
- VMs retry aggressively
- Zeek logs everything by design

To fix this, open the OPNsense web interface and head to **Services > Unbound DNS > Query Forwarding** and click **Add**

This is my configuration. Click save

![25](/assets/img/cyberlab2/setup/log-visibility/25.png){: width="800" height="400" }

Now, we should see a lot less [`ntp.ubuntu.com`](http://ntp.ubuntu.com) logs.

![26](/assets/img/cyberlab2/setup/log-visibility/26.png){: width="800" height="400" }