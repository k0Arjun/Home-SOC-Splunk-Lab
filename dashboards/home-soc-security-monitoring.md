# Home SOC Lab — Security Monitoring Dashboard

## 1. Overview

The Home SOC Lab Security Monitoring dashboard was developed using Splunk Enterprise to provide centralized visibility into Windows endpoint security events.

It brings together authentication activity, account management events, suspicious process execution and process discovery activity.

The dashboard uses Windows Security logs and Sysmon telemetry collected from the Windows 11 endpoint.

## 2. Dashboard Details

| Attribute | Details |
|---|---|
| Platform | Splunk Enterprise 10.6.1.1 |
| Dashboard Type | Dashboard Studio |
| Dashboard Name | Home SOC Lab — Security Monitoring |
| Dashboard ID | home_soc_security_monitoring |
| Data Index | main |
| Endpoint | Windows 11 VM |
| Log Sources | Windows Security, Sysmon Operational |

## 3. Dashboard Components

### A. KPI Panels

| Panel | Purpose | Event Source |
|---|---|---|
| Total Security Events | Displays the number of collected Windows Security events | Security logs |
| Failed Logons | Displays failed authentication attempts | Event ID 4625 |
| Local Account Creations | Displays newly created user accounts | Event ID 4720 |
| Process Discovery Events | Displays PowerShell Get-Process activity | Sysmon Event ID 1 |

### B. Detection Trend Charts

The dashboard includes time-based visualizations for:

1. Failed logon activity.
2. Local account creation.
3. Script-host identity discovery.
4. PowerShell process discovery.

These charts help identify changes in event frequency over the selected time range.

### C. Recent Detection Events

A consolidated event table displays:

- Event timestamp
- Hostname
- Detection name
- Severity
- Event ID
- User information
- Process image and parent process
- Source IP address

This table provides a centralized view of events matching the four detection use cases.

## 4. Detection Coverage

| Detection | Event ID | Severity |
|---|---|---|
| Repeated Failed Logons | 4625 | Medium |
| Local Account Creation | 4720 | Medium |
| Script Host Identity Discovery | Sysmon 1 | Medium |
| PowerShell Process Discovery | Sysmon 1 | Low |

## 5. Dashboard Capabilities

- Centralized monitoring of Windows endpoint security events.
- KPI-based visibility into important security activity.
- Time-based detection trend analysis.
- Consolidated view of detection events.
- Global time range selection for investigating different periods.

## 6. Limitations

- Detection results depend on the events collected and the configured SPL logic.
- Event counts do not independently establish malicious activity.
- Legitimate administrative actions may trigger detections.
- This dashboard uses Splunk Enterprise, not the separately licensed Splunk Enterprise Security application.
- The lab currently monitors a controlled Windows endpoint rather than a production enterprise environment.

## 7. Evidence

Dashboard screenshots are stored in:

`dashboards/screenshots/`

- `dashboard_kpi.png`
- `Dashboard_Detection_charts.png`
- `Dashboard_recent_Events.png`

## 8. Project Outcome

The dashboard provides a centralized monitoring interface for the four implemented SOC detection use cases.

It demonstrates the practical use of Splunk Dashboard Studio, SPL searches, Windows Security Event Logs and Sysmon telemetry in a small-scale SOC monitoring environment.