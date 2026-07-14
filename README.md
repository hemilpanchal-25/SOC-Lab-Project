# SOC Lab: SSH Brute Force Attack Detection using Wazuh SIEM

## 📌 Overview
This project is a hands-on SOC (Security Operations Center) lab that simulates and detects an **SSH brute force attack** using **Wazuh SIEM** in a controlled virtual lab environment. It demonstrates the full attack-to-detection lifecycle — from an attacker attempting unauthorized access via password guessing, to a SIEM platform logging, alerting, and helping analyze the incident.

The goal of this lab is to build practical, real-world SOC analyst skills: threat detection, log analysis, and incident investigation.

## 🎯 Objectives
- Understand how attackers perform brute force / password guessing attacks against SSH services.
- Deploy and configure a Wazuh agent on a monitored endpoint.
- Simulate an SSH brute force attack using Hydra.
- Detect, analyze, and investigate the attack using Wazuh's dashboards, alerts, and threat hunting module.

## 🖥️ Lab Environment

| Role | Machine | Details |
|------|---------|---------|
| Attacker | Kali Linux | Used to launch the brute force attack |
| Target / Victim | Ubuntu 24.04 LTS | Monitored endpoint with Wazuh Agent installed |
| Monitoring | Wazuh SIEM (v4.14.4) | Central manager for log collection, correlation, and alerting |

**Virtualization:** VirtualBox / VMware
**Networking:** Both VMs connected via NAT + Host-only/Bridged adapter on the same network

- Ubuntu (Victim) IP: `192.168.10.55`
- Kali Linux (Attacker) IP: `192.168.10.56`

## 🛠️ Tools & Technologies Used
- **Wazuh SIEM** – log monitoring, correlation, and alerting
- **Hydra** – brute force attack tool
- **OpenSSH** – remote login service (attack target)
- **Nmap** – network/service discovery and connectivity verification
- **VirtualBox / VMware** – virtual lab infrastructure

## 🔬 Lab Workflow

1. **Wazuh Manager Setup** – Installed and logged into the Wazuh dashboard.
2. **Agent Deployment** – Deployed a new agent from the Wazuh dashboard, generated the install command, and ran it on the Ubuntu victim machine.
3. **Agent Activation** – Started the `wazuh-agent` service via `systemctl` and confirmed the agent showed as **Active** in the Wazuh Endpoints dashboard.
4. **Environment Prep on Ubuntu**
   - Created test user accounts (`admin`, `test`, `web`) with weak passwords.
   - Installed and enabled the OpenSSH server (`sudo apt install openssh-server -y`).
5. **Network Connectivity Check** – Verified connectivity between attacker and victim using `ping` and an `nmap` scan (confirmed port 22/tcp open).
6. **Wordlist Creation** – Built simple `username.txt` and `password.txt` wordlists on Kali for the brute force attempt.
7. **Brute Force Attack** – Launched Hydra from Kali against the Ubuntu SSH service:
   ```
   hydra -L username.txt -P password.txt ssh://192.168.10.55/
   ```
   Result: 4 valid credential pairs discovered.
8. **Attacker Login** – Logged into the Ubuntu machine over SSH using the discovered credentials to confirm successful compromise.
9. **Detection & Analysis in Wazuh**
   - Reviewed the **Overview** dashboard for alert severity breakdown (critical/high/medium/low).
   - Used the **Threat Hunting** module filtered by agent to review authentication events.
   - Identified key rules triggered, including:
     - `sshd: authentication failed` (rule level 5, rule ID 5760)
     - `sshd: brute force trying to get access to the system. Authentication Failed` (rule level 10, rule ID 5763)
     - `PAM: User login failed` (rule ID 5503)
     - `sshd: Attempt to login using a non-existent user` (rule ID 5710)
     - `PAM: Login session opened` (rule level 3, rule ID 5501) — successful logins
   - Cross-referenced Wazuh alerts against raw SSH logs on the Ubuntu host for verification.

## 📊 Key Results
- **4,827** authentication failure events detected during the attack window.
- **48** authentication success events (successful logins following the brute force).
- **16** alerts at rule level 12 or above (high-severity brute force detections).
- Wazuh's **Vulnerability Detection** module also surfaced additional findings on the endpoint (CVE-tagged package vulnerabilities) and **Configuration Assessment** results against the CIS Ubuntu 24.04 LTS Benchmark.

## 📁 Repository Structure
```
SOC-LAB-PROJECT/
├── README.md
├── screenshots/          # Practical screenshots from the lab (agent setup, attack, detection)
└── docs/                 # (optional) Full write-up / report
```

## 🧠 Key Takeaways
- Demonstrates how weak/default credentials combined with an exposed SSH service create a real attack surface.
- Shows how a SIEM correlates raw authentication logs into actionable, severity-ranked alerts.
- Reinforces core SOC analyst workflows: monitoring → detection → log correlation → investigation.

## ⚠️ Disclaimer
This lab was performed entirely within an isolated virtual environment for educational purposes only. The techniques demonstrated (brute force attacks) should never be used against systems you do not own or have explicit authorization to test.

## 👤 Author
**Hemil Panchal**
Prepared as part of a personal/academic cybersecurity internship and SOC lab practice project.
