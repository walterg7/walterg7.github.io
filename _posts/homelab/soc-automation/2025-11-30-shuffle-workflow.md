---
title: "Shuffle Workflow Setup"
date: 2025-11-30 00:00:00 -0800
categories: [Cybersecurity Lab, SOC Automation]
tags: [cybersecurity, homelab, automation]
description: Creating the Shuffle workflow
pin: false
---
Before we get started, make sure your Wazuh, TheHive, Windows and Ubuntu VMs are all running.

### Creating the Workflow

On the left hand side, select **Automate > Workflows > Create Workflow**

Give it a name and description then click **Create from scratch**

![1](/assets/img/cyberlab/soc-automation/shuffle-workflow/1.png){: width="500" height="400" }

Under triggers (left hand side), drag the webhook (1st icon) onto the work area

Click on it and copy the webhook URI

![2](/assets/img/cyberlab/soc-automation/shuffle-workflow/2.png){: width="800" height="400" }

SSH into the Wazuh VM

run ```nano /var/ossec/etc/ossec.conf```

In between the "global" and "alerts" tag, paste the following:
```
  <integration>
    <name>shuffle</name>
    <hook_url>http://your_webhook_uri</hook_url>
    <rule_id>100002</rule_id>
    <alert_format>json</alert_format>
  </integration>
```

Make sure "hook_url" includes your unique webhook URI (the one you copied from Shuffle)

Save and write changes, then run ```systemctl restart wazuh-manager.service``` to apply the changes

![3](/assets/img/cyberlab/soc-automation/shuffle-workflow/3.png){: width="800" height="400" }

On Shuffle, click start on the webhook

![4](/assets/img/cyberlab/soc-automation/shuffle-workflow/4.png){: width="800" height="400" }

On the Windows VM, trigger the alert by executing Mimikatz

My path to the Mimikatz executable is **\Downloads\mimikatz_trunk\Win32**

![5](/assets/img/cyberlab/soc-automation/shuffle-workflow/5.png){: width="600" height="400" }

Head back to Shuffle. Click **Explore all runs** (right side)

Our Mimikatz alert should pop up

![6](/assets/img/cyberlab/soc-automation/shuffle-workflow/6.png){: width="800" height="400" }

Delete the "Change me" item

Drag the “repeat back to me” item (orange box with recycle symbol) onto the work area and change “Repeat back to me” to “Regex capture group”

Under regex, enter: **SHA256=([0-9A-Fa-f]{64})**

This regex basically retrieves the SHA256 file hash from the Wazuh alert.

![7](/assets/img/cyberlab/soc-automation/shuffle-workflow/7.png){: width="800" height="400" }

Attach the webhook to the Shuffle tool (the arrow should point to Shuffle Tools 1)

Rename **Shuffle Tools 1** to **SHA256-Hash**

Under input data, click **runtime argument > hashes**

My runtime argument ended up being **$exec.all_fields.full_log.win.eventdata.hashes**, yours may be different

![8](/assets/img/cyberlab/soc-automation/shuffle-workflow/8.png){: width="800" height="400" }

### VirusTotal Integration

If you haven’t already, register for a VirusTotal account to get a free API key: [https://www.virustotal.com/gui/home/upload](https://www.virustotal.com/gui/home/upload)

Log in, click on the **top right > API key**

Copy your API key

Look for the VirusTotal app on Shuffle and drag it onto the work area

Click on **VirusTotal > Authenticate to VirusTotal**

Change the label name and paste your API key

Click **Submit**

![9](/assets/img/cyberlab/soc-automation/shuffle-workflow/9.png){: width="700" height="400" }

Click on VirusTotal 

For **find actions**, select **Get a hash report**

For **id**, select **SHA256-Hash** and click list

![10](/assets/img/cyberlab/soc-automation/shuffle-workflow/10.png){: width="500" height="400" }

The **id** field should be: **$SHA256-Hash.group_0.#**

![11](/assets/img/cyberlab/soc-automation/shuffle-workflow/11.png){: width="600" height="400" }

Make sure the webhook is still running

![12](/assets/img/cyberlab/soc-automation/shuffle-workflow/12.png){: width="800" height="400" }

Head to **Explore all runs** > **Rerun workflow**

We should get an HTTP 200 code. The **id** field should match the hash from our Wazuh log

![13](/assets/img/cyberlab/soc-automation/shuffle-workflow/13.png){: width="700" height="400" }

Some of the VirusTotal data

![14](/assets/img/cyberlab/soc-automation/shuffle-workflow/14.png){: width="500" height="400" }

### TheHive Integration

Drag TheHive onto the work area (do not attach it to anything yet)

![15](/assets/img/cyberlab/soc-automation/shuffle-workflow/15.png){: width="800" height="400" }

Head to TheHive dashboard (http://10.10.1.40:9000 in my case)

Log in and on the top left, click on **Add Organization**

Add a name and description and confirm (leave **tasks sharing rule** and **observables sharing rule** at default)

![16](/assets/img/cyberlab/soc-automation/shuffle-workflow/16.png){: width="600" height="400" }

Click on the new org and add a new user (top left)

Set a login and name

Set the profile to analyst

Click **Save and add another**

![17](/assets/img/cyberlab/soc-automation/shuffle-workflow/17.png){: width="600" height="400" }

This will be a service account

I set the login to shuffle@test.com and the name to SOAR

Set the profile to analyst

![18](/assets/img/cyberlab/soc-automation/shuffle-workflow/18.png){: width="600" height="400" }

Click on the user account (eyeball icon)

My user account is joe@cyberlab.com

Give this user a password

![19](/assets/img/cyberlab/soc-automation/shuffle-workflow/19.png){: width="600" height="400" }

Next, click on SOAR (eyeball icon)

Create an API key for this account

Copy the key

![20](/assets/img/cyberlab/soc-automation/shuffle-workflow/20.png){: width="600" height="400" }

Head back to Shuffle

Click on **TheHive > Authenticate TheHive**

Set a label name and paste the API key

For the URL, put in the IP of TheHive VM and set the port to 9000

![21](/assets/img/cyberlab/soc-automation/shuffle-workflow/21.png){: width="700" height="400" }

Connect the VirusTotal item to TheHive (make sure the arrow points to TheHive)

Click TheHive

For **Find Actions**, set it to **create alert**

Click on advanced

![22](/assets/img/cyberlab/soc-automation/shuffle-workflow/22.png){: width="700" height="400" }

Paste this JSON for the body (your runtime arguments may be different, but these are mine)

```json
{
  "description": "$exec.title",
  "externallink": "${externallink}",
  "flag": false,
  "pap": 2,
  "severity": "3",
  "source": "$exec.pretext",
  "sourceRef": "$exec.rule_id",
  "status": "New",
  "summary": "Mimikatz activity detected on host $exec.all_fields.data.win.system.computer",
  "tags": ["T1003"],
  "title": "$exec.title",
  "tlp": 2,
  "type": "Internal"
}
```

This will basically create a case on TheHive using information from the Wazuh log. You can customize the data to your liking and make it as descriptive as you want, but for now we just want this to be a proof of concept to make sure everything works.

Optional VirusTotal arguments to include in the case.

![23](/assets/img/cyberlab/soc-automation/shuffle-workflow/23.png){: width="600" height="400" }

Save and rerun the workflow

We should get a an HTTP 201 Created code

![24](/assets/img/cyberlab/soc-automation/shuffle-workflow/24.png){: width="700" height="400" }

Head back to TheHive dashboard

Log in as the analyst user (joe@cyberlab.com in my case)

On the left hand side, notice the alert

Click on the alert, and we should see the Mimikatz case created

![25](/assets/img/cyberlab/soc-automation/shuffle-workflow/25.png){: width="800" height="400" }

Nothing fancy, but our automated workflow is working so far

We may add extra data like the process ID, time of execution, etc.

![26](/assets/img/cyberlab/soc-automation/shuffle-workflow/26.png){: width="800" height="400" }

### Email Alert

Head back to Shuffle

Drag Email onto the work area

![27](/assets/img/cyberlab/soc-automation/shuffle-workflow/27.png){: width="800" height="400" }

Connect VirusTotal to the Email icon

Click on Email

Leave **Find Actions** to **Send email shuffle**

Paste your Shuffle API key (free with your Shuffle account)

>You will need to register for an actual Shuffle account to get your free API key - [https://shuffler.io/register](https://shuffler.io/register)
{: .prompt-info}

Under recipients, put your own email

For the subject and body, you can set this to anything you want.

We can add runtime arguments for the body, but again, this is mainly a proof of concept so we don’t want anything too crazy yet.

![28](/assets/img/cyberlab/soc-automation/shuffle-workflow/28.png){: width="800" height="400" }

Save and rerun the workflow

When it is finished executing, we should see a success message for the email

![29](/assets/img/cyberlab/soc-automation/shuffle-workflow/29.png){: width="800" height="400" }

Head to your email

We should see the alert

![30](/assets/img/cyberlab/soc-automation/shuffle-workflow/30.png){: width="400" height="400" }

Notice the timestamps of the Wazuh log and the time when the email was sent; it is almost instantaneous.

![31](/assets/img/cyberlab/soc-automation/shuffle-workflow/31.png){: width="800" height="400" }

Next, we will create a Wazuh Active Response rule to quarantine the endpoint and update our email alert.

Next: [Wazuh Active Response](/posts/wazuh-ar)