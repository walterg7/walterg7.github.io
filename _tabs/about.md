---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

# Welcome

I am Walter, a recent IT graduate with a strong foundation in networking, systems administration, web development, cloud computing, and a growing focus on cybersecurity. I am documenting my learning journey for everyone to see. This is my portfolio for my projects, feel free to follow along.

# Resume

### Skills

**Operating Systems:** Windows 10/11, Windows Server 2022/2025, Linux (Ubuntu 22.04/24.04 LTS)

**Cloud & Infrastructure:** Azure (Sentinel, VNets, NSGs, IAM, Logic Apps), GCP (VPC, Compute Engine, IAM, BigQuery), Active Directory (Users, GPO, RBAC, Domain Services), Virtualization (VirtualBox, Type 2 Hypervisors), RabbitMQ (AMQP Messaging)

**Security Engineering & Monitoring:** SIEM (Wazuh, Microsoft Sentinel), SOAR (Shuffle), Endpoint Telemetry (Sysmon), Vulnerability Management (OpenVAS, Nmap), Network Monitoring (Wireshark, Nagios), Firewalls (OPNsense)

**Scripting & Automation:** Python, PowerShell, Bash, KQL, SQL

### Projects

[**Cybersecurity & Systems Administration Home Lab**](/posts/main)
- Designed segmented lab network using OPNsense to simulate corporate, server, and administrative environments

- Deployed and managed Active Directory domain with users, RBAC, file shares, and GPOs to emulate enterprise identity management

- Configured Sysmon via GPO and centralized logs in Wazuh SIEM for Windows telemetry monitoring

- Developed custom Wazuh detection rules to identify credential dumping activity

[**SOC Automation Project**](/posts/soc-main)
- Built custom Wazuh detection rules and Active Response to automatically isolate compromised endpoints

- Developed Shuffle SOAR workflows for alert enrichment, orchestration, and automated containment

- Integrated VirusTotal API for IOC enrichment and threat intelligence automation

- Connected alerts to TheHive for structured case management and triage

- Automated high-severity alert notifications to reduce response time

[**Systems Integration Project - Backend Developer**](/posts/it490-main)
- Designed messaging architecture using RabbitMQ (exchanges, queues, consumers) for reliable inter-service communication

- Built secure authentication workflows with API validation and session management

- Developed push notification services to support real-time user updates

- Implemented centralized logging for distributed services to improve observability

- Integrated Nagios and OpenVAS for infrastructure monitoring and vulnerability scanning

[**Cloud Administration Projects**](/posts/cloud-main)
- Designed and deployed secure cloud environments in Azure and GCP simulating business applications

- Configured networking and access controls (VNet/VPC, NSGs, IAM, firewalls, private endpoints)

- Applied CIS Benchmark hardening standards across cloud resources

- Deployed and managed cloud services including:

    - Azure: VNets, NSGs, Private DNS, IAM, Firewall Rules

    - GCP: VPC, Compute Engine, Cloud Storage, Autoscaling, BigQuery

[**Azure SOC Project**](/posts/azure-soc-main/)
- Deployed intentionally exposed Windows VM to simulate external attack surface and generate real-world telemetry

- Configured Microsoft Sentinel for log ingestion, analytics, and alerting

- Built custom Sentinel workbook to visualize attacker IP geolocation and brute-force patterns

- Developed and automated RDP blocking playbook using Azure Logic Apps to reduce brute-force noise

- Created and tuned detection rule for persistence via scheduled tasks

### Certifications 

[**CompTIA Security+**](https://www.credly.com/badges/4a6a06ae-dcb2-44ae-9cff-d4307e54d291)

# Contact Me

- **Email:** [walter.guerrero7@outlook.com](mailto:walter.guerrero7@outlook.com)
- **LinkedIn:** [walterguerrero7](https://www.linkedin.com/in/walterguerrero7/)