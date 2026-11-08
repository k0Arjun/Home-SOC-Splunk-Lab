# Windows Endpoint Log Collection and Field Extraction

## 1. Objective

Integrate a Windows 11 endpoint into the Home SOC Lab and forward Windows Security and Sysmon events to Splunk Enterprise.

The objective was to:
- Install and configure Sysmon.
- Install Splunk Universal Forwarder.
- Forward Windows event logs to Splunk Enterprise.
- Verify event ingestion.
- Configure reusable search-time field extractions.
- Troubleshoot app-level configuration visibility in Splunk.

## 2. Lab Architecture

| Component | Configuration |
|---|---|
| SIEM server | Ubuntu Server 26.04.1 LTS |
| SIEM platform | Splunk Enterprise 10.6.1.1 |
| Endpoint | Windows 11 VM |
| Endpoint monitoring | Microsoft Sysmon |
| Log collector | Splunk Universal Forwarder |
| Receiving port | TCP 9997 |
| Event index | `main` |
| Sysmon sourcetype | `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` |
| Security sourcetype | `XmlWinEventLog:Security` |

The Windows endpoint and Splunk server communicate over the lab's host-only virtual network. NAT is used for internet access where required.

## 3. Sysmon Installation and Verification

Installed Microsoft Sysmon on the Windows 11 endpoint.

Verified the Sysmon service and event channel:

- Service: `Sysmon64`
- Event channel: `Microsoft-Windows-Sysmon/Operational`

Generated test activity using:

```powershell
whoami
```

Verified that process-related activity appeared in Event Viewer.

## 4. Splunk Universal Forwarder Configuration

Installed Splunk Universal Forwarder on Windows 11.

### 4.1 Forwarding configuration

File:

`C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf`

```ini
[tcpout]
defaultGroup = splunk_lab_receiver

[tcpout:splunk_lab_receiver]
server = <SPLUNK_SERVER_IP>:9997
```

The placeholder represents the current host-only IP of the Splunk server.

### 4.2 Windows event inputs

File:

`C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

```ini
[WinEventLog://Security]
disabled = 0
index = main
renderXml = true

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = main
renderXml = true
```

### 4.3 Windows service permission troubleshooting

Initially, Security events were being forwarded, but Sysmon events were not.

The Universal Forwarder service was running as:

`NT SERVICE\SplunkForwarder`

The Splunk forwarder log showed an access-denied error when subscribing to:

`Microsoft-Windows-Sysmon/Operational`

Changed the `SplunkForwarder` Windows service Log On account to `Local System` and restarted the service.

After this change, Sysmon events began appearing in Splunk.

**Security note:** Local System has extensive privileges. This was a lab troubleshooting workaround, not a recommended production configuration. Production deployments should use an appropriately configured least-privilege service identity.

## 5. Verification of Event Ingestion

Confirmed that both sourcetypes were available in Splunk:

```text
XmlWinEventLog:Security

XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

Verified the Sysmon source and sourcetype using:

```spl
index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| head 5
| table host source sourcetype
```

The endpoint host and expected sourcetype were displayed.

## 6. Field Extraction Problem

Although Sysmon events were successfully ingested, fields such as `EventCode`, `Image`, `CommandLine`, `User`, and `ParentImage` were not automatically available in search results.

The raw event contained XML similar to:

```xml
<EventID>5</EventID>
<Data Name='Image'>C:\Program Files\SplunkUniversalForwarder\bin\splunk-MonitorNoHandle.exe</Data>
<Data Name='User'>NT AUTHORITY\SYSTEM</Data>
```

The data existed in `_raw`, but the desired fields were not populated automatically.

### 6.1 Manual extraction test

Tested the XML using SPL `rex`:

```spl
index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID>(?<EventCode>\d+)</EventID>"
| rex field=_raw "Data Name='Image'>(?<Image>[^<]*)</Data>"
| rex field=_raw "Data Name='CommandLine'>(?<CommandLine>[^<]*)</Data>"
| rex field=_raw "Data Name='User'>(?<User>[^<]*)</Data>"
| rex field=_raw "Data Name='ParentImage'>(?<ParentImage>[^<]*)</Data>"
| table _time host EventCode Image CommandLine User ParentImage
| head 10
```

**Result:** Manual field extraction worked. This confirmed that the raw XML structure and regex expressions were suitable.

### 6.2 Persistent extraction configuration

Created the custom Splunk app directory:

```bash
sudo mkdir -p /opt/splunk/etc/apps/soc-lab-parsing/local
```

Created:

`/opt/splunk/etc/apps/soc-lab-parsing/local/props.conf`

```ini
[XmlWinEventLog:Microsoft-Windows-Sysmon/Operational]

EXTRACT-eventcode = <EventID>(?<EventCode>\d+)</EventID>
EXTRACT-image = <Data Name='Image'>(?<Image>[^<]*)</Data>
EXTRACT-commandline = <Data Name='CommandLine'>(?<CommandLine>[^<]*)</Data>
EXTRACT-user = <Data Name='User'>(?<User>[^<]*)</Data>
EXTRACT-parentimage = <Data Name='ParentImage'>(?<ParentImage>[^<]*)</Data>
```

Verified that Splunk loaded the extraction settings using:

```bash
sudo -u splunk /opt/splunk/bin/splunk btool props list --debug | grep -E 'EXTRACT-(eventcode|image|commandline|user|parentimage)'
```

All five extraction rules appeared in the output.

However, the fields were still empty when searching from Search & Reporting.

### 6.3 Root cause: App-level configuration visibility

The custom extraction rules were stored inside the `soc-lab-parsing` app, while searches were being executed in the Search & Reporting app.

Splunk uses app context when evaluating search-time configurations. Configuration in another app may not be available unless it is exported globally.

Created the metadata directory:

```bash
sudo mkdir -p /opt/splunk/etc/apps/soc-lab-parsing/metadata
```

Created:

`/opt/splunk/etc/apps/soc-lab-parsing/metadata/default.meta`

```ini
[props]
export = system
```

This exports the app's `props` configuration category globally, making the extraction configuration available in other apps.

Restarted Splunk Enterprise:

```bash
sudo systemctl restart Splunkd
```

Verified service status:

```bash
sudo systemctl is-active Splunkd
```

### 6.4 Final validation

Executed the following search without any manual `rex` commands:

```spl
index=main sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| table _time host EventCode Image CommandLine User ParentImage
| head 20
```

**Result: SUCCESS**

The fields were automatically extracted and displayed in the search results.

## 7. Important Troubleshooting Lessons

| Problem | Investigation | Resolution |
|---|---|---|
| Sysmon events not arriving | Forwarder service lacked access to Sysmon channel | Changed service account to Local System for the lab |
| Automatic fields empty | Tested raw XML with `rex` | Confirmed regex expressions worked |
| Extraction rules not visible in searches | Inspected `btool` output and app context | Added global export in `default.meta` |
| Empty columns for some event types | Inspected the event's XML | Recognized that not every event contains every field |

### Key technical lessons

- Successful event ingestion does not guarantee successful field extraction.
- `source` and `sourcetype` are important when troubleshooting event data.
- `props.conf` can define reusable search-time extractions.
- `EXTRACT-` uses named regular-expression capture groups.
- `default.meta` controls configuration sharing and app visibility.
- `btool` helps inspect effective configuration and its origin.
- Fields such as `CommandLine` and `ParentImage` may legitimately be absent from certain Sysmon event types.

## 8. Current Project Status

| Task | Status |
|---|---|
| Ubuntu Splunk server deployment | Completed |
| Splunk receiving port 9997 | Verified |
| Windows 11 endpoint deployment | Completed |
| Sysmon installation | Verified |
| Universal Forwarder installation | Completed |
| Windows Security event forwarding | Verified |
| Sysmon event forwarding | Verified |
| Persistent Sysmon field extraction | Verified |
| Cross-app configuration visibility | Fixed and verified |

## 9. Next Steps

1. Validate additional Sysmon event types and their extracted fields.
2. Configure and validate Windows Security event field extractions.
3. Generate controlled test activity for process creation and failed logons.
4. Begin detection engineering using SPL.
5. Map validated detections to MITRE ATT&CK.
6. Create investigation reports and a SOC dashboard.

## 10. References

- [Splunk: Configure custom fields at search time](https://help.splunk.com/en/splunk-enterprise/manage-knowledge-objects/knowledge-management-manual/10.2/use-the-configuration-files-to-configure-field-extractions/configure-custom-fields-at-search-time)
- [Splunk: Configuration file precedence](https://help.splunk.com/en/splunk-enterprise/administer/admin-manual/10.2/administer-splunk-enterprise-with-configuration-files/configuration-file-precedence)
- [Splunk: default.meta.conf](https://help.splunk.com/en/splunk-enterprise/administer/admin-manual/9.4/configuration-file-reference/9.4.12-configuration-file-reference/default.meta.conf)
- [Splunk: Use btool to troubleshoot configurations](https://help.splunk.com/en/splunk-enterprise/administer/troubleshoot/10.6/first-steps/use-btool-to-troubleshoot-configurations)

---

**Project:** Home SOC Lab — Security Monitoring and Detection with Splunk Enterprise

**Milestone:** Windows Endpoint Log Collection and Field Extraction

**Status:** Completed and validated in the lab.