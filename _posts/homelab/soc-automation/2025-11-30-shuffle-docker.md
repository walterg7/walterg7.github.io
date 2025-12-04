---
title: "Shuffle Installation via Docker"
date: 2025-11-30 00:00:00 -0800
categories: [Cybersecurity Lab, SOC Automation]
tags: [cybersecurity, homelab, automation]
description: Installing Shuffle via Docker.
pin: false
---
Follow the [Ubuntu server VM setup](/posts/ubuntu-server-vm) again. This time, we will use this VM to run the Docker engine, which will run a Shuffle instance.

I gave this VM 8 GB RAM, 4 CPUs, 50GB storage.

This VM also has the IP address 10.10.1.41.

Once you set everything up, first install git: ```sudo apt install git```

### Docker Engine Install

The Docker Engine installation guide is available at: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

These are the commands I ran:
```bash
# Uninstall all conflicting packages:
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)

# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

# Install docker
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Optional sanity check
sudo systemctl status docker

sudo systemctl start docker

sudo docker run hello-world
```
After running all the commands, Docker should automatically be running

![1](/assets/img/cyberlab/soc-automation/shuffle-docker/1.png){: width="800" height="400" }

### Shuffle Installation

The installation guide is available at: [https://github.com/Shuffle/Shuffle/blob/main/.github/install-guide.md](https://github.com/Shuffle/Shuffle/blob/main/.github/install-guide.md)

Commands to run:

```bash
git clone https://github.com/Shuffle/Shuffle
cd Shuffle
sudo chown -R 1000:1000 shuffle-database    # IF you get an error using 'chown', add the user first with 'sudo useradd opensearch'

sudo swapoff -a                             # Disable swap
```

>Before running ```docker compose up -d```, change the **DOCKER_API_VERSION** to 1.44 in both **docker-compose.yml** and **.env** (both files are in root directory Shuffle/). At the time of this writing, the default Docker API version for the Shuffle repo is set to 1.40, however this is too low for the apps that we are going to use on Shuffle (VirusTotal and TheHive). The minimum API version must be set to at least 1.44. Otherwise, you will get this error when trying to activate any of the apps.
{: .prompt-danger }

![2](/assets/img/cyberlab/soc-automation/shuffle-docker/2.png){: width="800" height="400" }

**docker-compose.yml**

under environment, change **DOCKER_API_VERSION** to 1.44 

save and write changes

![3](/assets/img/cyberlab/soc-automation/shuffle-docker/3.png){: width="800" height="400" }

**.env**

change **DOCKER_API_VERSION** to 1.44 in .env as well

save and write changes

![4](/assets/img/cyberlab/soc-automation/shuffle-docker/4.png){: width="600" height="400" }

THEN run ```docker compose up -d```

Sanity check

![5](/assets/img/cyberlab/soc-automation/shuffle-docker/5.png){: width="800" height="400" }

Another sanity check

![6](/assets/img/cyberlab/soc-automation/shuffle-docker/6.png){: width="500" height="400" }

Then run ```sudo sysctl -w vm.max_map_count=262144```

After installation, head to the Shuffle dashboard (http://10.10.1.41:3001 in my case) and create an admin account

When it prompts you to log in, log in using the same credentials

![7](/assets/img/cyberlab/soc-automation/shuffle-docker/7.png){: width="800" height="400" }

We should be good to go

![8](/assets/img/cyberlab/soc-automation/shuffle-docker/8.png){: width="800" height="400" }

Next: [Shuffle Workflow](/posts/shuffle-workflow)