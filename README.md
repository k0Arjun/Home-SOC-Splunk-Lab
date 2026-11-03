# Home SOC Lab — Security Monitoring and Detection with Splunk Enterprise

**Project Status:** In Progress 🚧

## 1. Project Overview

This project focuses on building a small Security Operations Center (SOC) home lab using Splunk Enterprise to practice security monitoring, log analysis, threat detection, and incident investigation.

The lab simulates a small enterprise Windows environment where security events are generated, collected, forwarded to Splunk, and investigated using Search Processing Language (SPL).

The objective is to develop practical SOC Analyst skills through controlled security simulations and hands-on detection engineering.

## 2. Project Objectives

- Deploy and configure Splunk Enterprise.
- Establish centralized Windows event log collection.
- Configure Sysmon for endpoint activity monitoring.
- Integrate Windows Security and PowerShell logs.
- Develop custom SPL-based detection searches.
- Simulate benign adversary techniques using Atomic Red Team.
- Investigate suspicious activities and document findings.
- Build security monitoring dashboards.
- Map detection use cases to MITRE ATT&CK.

## 3. Lab Environment

### Host Machine

| Component | Specification |
|---|---|
| Processor | AMD Ryzen 5 7000 Series |
| RAM | 16 GB |
| Host OS | Windows 11 |
| Hypervisor | Oracle VirtualBox |

### Planned Virtual Machines

| Virtual Machine | Operating System | Purpose |
|---|---|---|
| SOC-SIEM | Ubuntu Server | Splunk Enterprise |
| SOC-WIN11 | Windows 11 | Endpoint monitoring and Sysmon |
| SOC-DC01 | Windows Server | Active Directory and domain authentication |

The lab will be deployed in phases to accommodate available hardware resources.

## 4. Technology Stack

- **SIEM:** Splunk Enterprise
- **Log Collection:** Splunk Universal Forwarder
- **Endpoint Telemetry:** Microsoft Sysmon
- **Operating Systems:** Ubuntu Server, Windows 11, Windows Server
- **Virtualization:** Oracle VirtualBox
- **Security Testing:** Atomic Red Team
- **Detection Framework:** MITRE ATT&CK
- **Query Language:** Splunk Search Processing Language (SPL)

## 5. Planned Detection Use Cases

| Use Case | Windows Event / Telemetry | MITRE ATT&CK |
|---|---|---|
| Suspicious command execution | Sysmon Event ID 1 | T1059 |
| Local account creation | Security Event ID 4720 | T1136.001 |
| Repeated failed logons | Security Event ID 4625 | T1110 |
| PowerShell activity | PowerShell logs and Sysmon | T1059.001 |

These are planned use cases. Detection logic and event collection will be validated during implementation.

## 6. Project Architecture

*Architecture diagram will be added after the virtual lab network and data collection pipeline are configured and tested.*

## 7. Repository Structure

```text
home-soc-splunk-lab/
│
├── README.md
├── docs/
├── architecture/
├── detections/
├── dashboards/
├── incident-reports/
├── screenshots/
└── test-results/
```

## 8. Expected Deliverables

- Working Splunk-based SOC lab.
- Centralized Windows security event collection.
- Custom SPL detection searches.
- Validated security alerts.
- Security monitoring dashboard.
- MITRE ATT&CK-mapped detection use cases.
- Incident investigation reports.
- Technical implementation documentation.

## 9. Learning Outcomes

This project is intended to develop practical experience in:

- SIEM deployment and administration.
- Windows event log analysis.
- Endpoint monitoring.
- SPL-based security investigations.
- Detection engineering.
- Security alert triage.
- Incident investigation and documentation.

## 10. Disclaimer

This is an independent educational cybersecurity project conducted in an isolated virtual environment.

All security simulations will be performed on systems owned and controlled by the project author. No unauthorized systems or production environments will be targeted.

---

**Project developed for hands-on SOC Analyst learning and cybersecurity portfolio development.**






## Implementation Progress

### Milestone 1: Splunk Server Deployment — Completed

**Objective:** Deploy a centralized SIEM server using Splunk Enterprise on Ubuntu Server.

**Environment:**
- Operating System: Ubuntu Server 26.04.1 LTS
- SIEM Platform: Splunk Enterprise 10.6.1.1
- Virtualization: Oracle VirtualBox
- Architecture: AMD64

**Implementation:**
- Created and configured an Ubuntu Server virtual machine.
- Configured NAT networking for internet connectivity.
- Configured Host-only networking for private lab communication.
- Expanded the Ubuntu root logical volume using LVM.
- Installed Splunk Enterprise using the Debian package.
- Configured a dedicated non-root `splunk` service account.
- Accessed Splunk Web from the Windows host machine.

**Verification:**
- Confirmed Splunk daemon is running.
- Verified Splunk Web on TCP port 8000.
- Verified Splunk Management Interface on TCP port 8089.
- Confirmed Splunk Enterprise version 10.6.1.1.

**Status:** Completed

