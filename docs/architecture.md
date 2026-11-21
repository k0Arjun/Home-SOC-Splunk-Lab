# Home SOC Lab — Architecture

## 1. Project Overview

This document describes the architecture of the Home SOC Lab built using Splunk Enterprise.

The objective is to create a centralized security monitoring environment that collects endpoint security logs, detects suspicious activities, and supports incident investigation.

The lab is being developed in multiple phases.

## 2. Current Infrastructure

### SIEM Server

| Component | Configuration |
|---|---|
| VM Name | SOC-Splunk-Ubuntu |
| Hostname | splunk-server |
| Operating System | Ubuntu Server 26.04.1 LTS |
| SIEM | Splunk Enterprise 10.6.1.1 |
| Architecture | AMD64 |
| Virtual Disk | 40 GB |
| Root Filesystem | Approximately 38 GB |
| Host-only IP | 192.168.56.111 |
| Splunk Web | TCP 8000 |
| Splunk Management | TCP 8089 |

### Virtual Network Configuration

The Ubuntu VM uses two virtual network adapters.

**Adapter 1 — NAT**

Purpose:
- Provide outbound internet connectivity.
- Download software and security updates.

Observed address: `10.0.2.15`

**Adapter 2 — Host-only**

Purpose:
- Enable private communication between the Windows host and lab virtual machines.
- Support SSH administration and file transfers.

Observed Ubuntu address: `192.168.56.111`

## 3. Current Architecture Diagram

```mermaid
flowchart TB
    HOST["Windows 11 Host Machine"]

    subgraph VBOX["Oracle VirtualBox"]
        NAT["NAT Network"]
        PRIVATE["Host-only Network"]

        UBUNTU["Ubuntu Server VM<br/>Hostname: splunk-server"]

        SPLUNK["Splunk Enterprise 10.6.1.1<br/>TCP 8000 / 8089"]

        UBUNTU --> SPLUNK
        UBUNTU --- NAT
        UBUNTU --- PRIVATE
    end

    HOST -->|"Splunk Web / HTTP<br/>TCP 8000"| PRIVATE
    HOST -->|"SSH / SCP<br/>TCP 22"| PRIVATE
    NAT -->|"Outbound connectivity"| INTERNET["Internet"]
```

## 4. Planned SOC Lab Architecture

The following components will be introduced in upcoming phases.

| Component | Role | Planned Technology |
|---|---|---|
| SIEM Server | Centralized log collection and analysis | Splunk Enterprise |
| Windows Endpoint | Endpoint security event generation | Windows 11 |
| Endpoint Monitoring | Process and system activity monitoring | Sysmon |
| Log Forwarding | Send endpoint logs to SIEM | Splunk Universal Forwarder |
| Domain Controller | Centralized identity and authentication | Windows Server / Active Directory |
| Security Testing | Controlled attack simulation | Atomic Red Team |
| Detection Framework | Classify adversary techniques | MITRE ATT&CK |

## 5. Planned Data Flow

```mermaid
flowchart LR
    ENDPOINT["Windows 11 Endpoint"]
    SYSMON["Sysmon"]
    UF["Splunk Universal Forwarder"]
    SIEM["Splunk Enterprise"]
    DETECTION["SPL Detection Searches"]
    ANALYST["SOC Analyst Investigation"]

    ENDPOINT --> SYSMON
    SYSMON --> UF
    UF -->|"Forward Security Events"| SIEM
    SIEM --> DETECTION
    DETECTION --> ANALYST
```

## 6. Security Design Considerations

- Use Host-only networking for private lab communication.
- Use NAT for outbound updates when required.
- Avoid Bridged networking for the isolated SOC environment.
- Do not expose Splunk management interfaces directly to the public internet.
- Use a dedicated non-root account to run Splunk.
- Keep administrator credentials private.
- Take VM snapshots before major configuration changes.
- Use controlled, authorized security simulations.
- Never include real credentials, employer logs, sensitive hostnames, or private information in public documentation.

## 7. Current Implementation Status

| Component | Status |
|---|---|
| VirtualBox installation | Completed |
| Ubuntu Server deployment | Completed |
| NAT networking | Completed |
| Host-only networking | Completed |
| Splunk Enterprise installation | Completed |
| Splunk Web access | Verified |
| Windows 11 endpoint | Planned |
| Sysmon integration | Planned |
| Universal Forwarder | Planned |
| Windows log ingestion | Planned |
| Detection engineering | Planned |
| SOC dashboard | Planned |


