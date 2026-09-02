
# Splunk SOC Home Lab

A self-built Security Operations Center lab simulating real-world attack detection and alerting, using Splunk Enterprise, a Universal Forwarder, and a Kali Linux attacker machine — all on an isolated virtual network.

## Overview

This project simulates a small enterprise SOC environment where common attacker techniques are executed, detected, and alerted on in real time. It covers the full detection engineering lifecycle: attack simulation → log ingestion → SPL detection logic → alerting → triage.

Built to demonstrate practical SOC analyst skills: log analysis, SIEM configuration, detection engineering, and MITRE ATT&CK-aligned incident response.

## Architecture

| Component | Role |
|---|---|
| Windows Host — Splunk Enterprise | Central SIEM, log indexing, alerting |
| Ubuntu Server — Splunk Universal Forwarder | Monitored endpoint, forwards logs to Splunk |
| Kali Linux | Attacker machine simulating malicious activity |

**Network:** VirtualBox Host-Only Adapter, `192.168.56.0/24` — isolated from the internet, all traffic contained within the virtual network.

## Tools & Technologies

| Category | Tool |
|---|---|
| Virtualization | VirtualBox (Host-Only networking) |
| SIEM | Splunk Enterprise |
| Log Shipping | Splunk Universal Forwarder |
| Victim OS | Ubuntu Server |
| Attacker OS | Kali Linux |
| Attack Tools | Hydra (SSH brute force), native Linux commands (`useradd`, `sudo`) |
| Search Language | SPL (Search Processing Language) |
| Alerting | Splunk real-time alerts, email notifications (SMTP) |
| Framework | MITRE ATT&CK |
| Documentation | Markdown, Microsoft Word |

## Setup & Configuration

1. **Provision VMs in VirtualBox**
   - Windows host: install Splunk Enterprise
   - Ubuntu Server: install Splunk Universal Forwarder
   - Kali Linux: default install, no additional Splunk components
2. **Configure networking**
   - Create a Host-Only Adapter (`192.168.56.0/24`) in VirtualBox
   - Assign static IPs to each VM within that range

3. **Verify forwarding** in Splunk Enterprise
   - Settings → Forwarder Management → confirm the Ubuntu host shows as connected
4. **Build SPL detection searches**
   - Write and test each search (`ssh-bruteforce.spl`, `new-user-account.spl`, `sudo-abuse.spl`) against live incoming data
5. **Create real-time alerts**
   - Per-result trigger, throttle enabled, field-based suppression (`src_ip` / `sudo_user`), severity assigned per MITRE-mapped risk level




## Attack Simulations

| Scenario | Tool / Command (on Kali) | Target |
|---|---|---|
| SSH Brute Force | Hydra against SSH service | Ubuntu Server |
| New User Account Created | Manual `useradd` / `adduser` execution | Ubuntu Server (via SSH session) |
| Sudo Command Abuse | `sudo` execution of sensitive commands (`passwd`, `visudo`, `chmod 777`, `/bin/bash`) | Ubuntu Server (via SSH session) |

Each simulation was run directly against the Ubuntu Server from the Kali attacker VM, generating log events that the Universal Forwarder shipped to Splunk for detection and alerting. Screenshots of each attack command and the resulting raw log entry are in [`screenshots/attacks/`](screenshots/attacks/).

## Detection Scenarios

| MITRE Technique ID | Technique | Detection Logic | Alert | Severity |
|---|---|---|---|---|
| [T1110.001](https://attack.mitre.org/techniques/T1110/001/) | Brute Force: Password Guessing | SSH failed login threshold (≥5 attempts) from same source IP within 5 minutes | SSH Brute Force Detected | Medium |
| [T1136.001](https://attack.mitre.org/techniques/T1136/001/) | Create Account: Local Account | New Linux user account creation via `useradd` / `adduser` | New User Account Created | High |
| [T1548.003](https://attack.mitre.org/techniques/T1548/003/) | Abuse Elevation Control Mechanism: Sudo and Sudo Caching | Sudo execution of sensitive commands (`passwd`, `visudo`, `chmod 777`, `/bin/bash`, `useradd`) | Sudo Abuse by Non-Admin User | Critical |



## How It Works

1. **Attack simulated** on Kali against the Ubuntu Server (e.g., Hydra SSH brute force, manual `useradd`, sudo command abuse)
2. **Universal Forwarder** ships `/var/log/auth.log` events to Splunk Enterprise in real time
3. **SPL search** parses and extracts key fields (`src_ip`, `user`, `sudo_user`, `command`) from raw log data
4. **Real-time alert** fires per-result when the search matches, with throttling to prevent alert fatigue
5. **Email notification** delivers alert details (user, command, host, timestamp) directly to inbox
6. **Triage** via Splunk's Triggered Alerts page, drilling into the raw event for investigation

## Repository Structure

```
splunk-soc-home-lab/
├── README.md
├── detection_rules/
│   ├── ssh_brute_force.spl
│   ├── new_user_account.spl
│   └── sudo_abuse.spl
└── screenshots/
    ├── attacks/                     # Kali attack commands + raw log ingestion
    ├── alerts/                      # Alert configs, triggered alerts, email notifications
    └── dashboard/                   # SOC dashboard panels
```

## Skills Demonstrated

- SIEM configuration and log source onboarding (Splunk Universal Forwarder)
- Detection engineering with SPL (search, `rex`, `stats`, field extraction)
- Alert tuning: throttling, suppression, severity classification
- MITRE ATT&CK framework mapping
- Incident triage workflow
- Technical documentation
