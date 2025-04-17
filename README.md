# Building a SOC + Honeynet in Azure (Live Traffic with NIST 800-53 Alignment)

![Cloud Honeynet / SOC](https://i.imgur.com/ZWxe03e.jpg)

## Introduction

In this project, I built a mini honeynet in Azure and ingested log sources from various resources into a Log Analytics workspace, which was then integrated with Microsoft Sentinel to build attack maps, trigger alerts, and generate incidents. The environment was left intentionally exposed for 24 hours to simulate real-world attacks and collect baseline data.

After the initial exposure, I applied hardening controls and aligned the environment with key controls from NIST SP 800-53 (Rev. 5) using Azure Policy, Defender for Cloud, and built-in compliance initiatives. I then re-measured the metrics to demonstrate the effectiveness of these controls.

The metrics I tracked were:

- `SecurityEvent` (Windows Event Logs)
- `Syslog` (Linux Event Logs)
- `SecurityAlert` (Sentinel alerts triggered)
- `SecurityIncident` (Incidents created by Sentinel)
- `AzureNetworkAnalytics_CL` (Malicious network flows into honeynet)

## Architecture Before Hardening / Security Controls

![Architecture Diagram](https://i.imgur.com/aBDwnKb.jpg)

## Architecture After Hardening + NIST 800-53 Controls Applied

![Architecture Diagram](https://i.imgur.com/YQNa9Pp.jpg)

The post-hardening architecture included:

- VNet segmentation with isolated subnets
- NSG rules hardened to allow traffic only from my admin IP
- Azure Storage, Key Vault, and Log Analytics now accessed only via Private Endpoints
- Azure Firewall and custom route tables to control egress traffic
- Azure Policy initiative for NIST SP 800-53 assigned at the subscription level

## NIST 800-53 Implementation in Azure

To align with NIST SP 800-53 Rev. 5, I used the following controls and services:

### Step 1: Assigned Built-in NIST Compliance Policy

- Policy Definition: `NIST SP 800-53 Rev. 5`
- Assigned At: Subscription level
- Mode: All resource types
- Scope: Resource group `SOC-Honeynet-RG` and above

This allowed Azure Defender for Cloud to continuously assess compliance against the framework.

### Step 2: Hardened NSGs and VM Access

- Only one trusted IP allowed inbound access (SSH/RDP)
- All other inbound traffic blocked

Control alignment:
- AC-4: Information Flow Enforcement
- SC-7: Boundary Protection

### Step 3: Defender for Cloud + Recommendations

- Enabled Defender for:
  - Virtual Machines
  - Storage Accounts
  - Key Vaults
- Fixed high-severity security misconfigurations flagged by Defender

Control alignment:
- SI-2: Flaw Remediation
- CM-2: Baseline Configuration

### Step 4: Private Endpoints for PaaS Services

- Converted public access to:
  - Azure Storage Account → Private Endpoint
  - Log Analytics Workspace → Private Link
  - Azure Key Vault → Private Endpoint

Control alignment:
- SC-12: Cryptographic Key Establishment
- AC-17: Remote Access

### Step 5: Logging + Sentinel Integration

- Connected VMs to Log Analytics via the MMA Agent
- Enabled built-in Sentinel analytics rules
- Created custom alert rule for “suspicious authentication failures”

Control alignment:
- AU-2: Auditable Events
- SI-4: Information System Monitoring

## Attack Maps Before Security Controls

<img width="1369" alt="Before-NSG-Malicious-Allowed" src="https://github.com/user-attachments/assets/cc900544-dac5-4fc5-8655-0c659e36a1e3" />
<br>
<img width="1375" alt="Before-Linux-SSH-auth-fail" src="https://github.com/user-attachments/assets/98e0d95e-c874-410d-8bc9-05287c8b4c1e" />
<br>
<img width="1375" alt="Before-Windows-RDP-auth-fail" src="https://github.com/user-attachments/assets/91cc4ffb-35c6-4bbc-881e-6225c684f9e3" />
<br>

## Metrics Before Hardening / Security Controls

Start Time: 2025-03-20 13:22:07  
Stop Time: 2025-03-21 13:22:07

| Metric                   | Count  |
|--------------------------|--------|
| SecurityEvent            | 89,984 |
| Syslog                   | 22,650 |
| SecurityAlert            | 10     |
| SecurityIncident         | 234    |
| AzureNetworkAnalytics_CL | 743    |

## Metrics After NIST Hardening Controls Applied

```All attack map queries returned no results due to no malicious activity post-hardening.```

Start Time: 2025-04-03 23:15:14  
Stop Time: 2025-04-04 23:15:14

| Metric                   | Count  |
|--------------------------|--------|
| SecurityEvent            | 32,947 |
| Syslog                   | 0      |
| SecurityAlert            | 0      |
| SecurityIncident         | 0      |
| AzureNetworkAnalytics_CL | 0      |

## Conclusion

In this project, I built and exposed a honeynet in Azure to capture malicious behavior, then remediated the environment using Azure-native tools aligned with NIST SP 800-53 Rev. 5.

Key takeaways:

- Without controls, even small Azure environments are heavily targeted
- Hardened environments drastically reduce attack surface and incident count
- Azure Policy and Defender for Cloud make it feasible to enforce compliance at scale
- Microsoft Sentinel provides real-time visibility into attack paths and logs

This project demonstrates how to use Azure cloud tools to simulate, detect, defend, and report threats using NIST-aligned best practices.
