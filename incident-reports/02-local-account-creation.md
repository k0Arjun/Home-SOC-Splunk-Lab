# Incident Investigation 02 — Local Account Creation

## 1. Incident Overview

| Field | Details |
|---|---|
| Incident Name | Local Account Creation |
| Host | ARJUN-LAB |
| Log Source | Windows Security Event Logs |
| Detection Event ID | 4720 — A User Account Was Created |
| Cleanup Event ID | 4726 — A User Account Was Deleted |
| Created Account | T1136.001_CMD |
| Creating Account | Administrator account |
| MITRE ATT&CK | T1136.001 — Create Account: Local Account |
| Severity | Medium (lab detection classification) |
| Investigation Status | Completed — Controlled Lab Simulation |

## 2. Alert Description

A Windows Security Event ID 4720 was detected, indicating that a new local user account had been created.

The account creation activity was generated using Atomic Red Team to simulate an attacker creating a local account for potential persistence.

Creating additional accounts can allow an attacker to maintain access to a compromised system.

## 3. Evidence Collected

| Evidence | Observation |
|---|---|
| Event ID 4720 | Local account creation detected |
| New account | T1136.001_CMD |
| Account creator | Administrator account |
| Event ID 4726 | Account deletion detected during cleanup |
| Test framework | Atomic Red Team |
| Atomic technique | T1136.001 |

## 4. Investigation and Analysis

1. Identified Event ID 4720 in Windows Security logs.
2. Examined the event to identify the newly created account.
3. Confirmed the account name as `T1136.001_CMD`.
4. Reviewed the account creator and identified the administrator account.
5. Correlated the creation event with Event ID 4726.
6. Confirmed that the test account was deleted during Atomic Red Team cleanup.

The test demonstrated how account creation and deletion events can be monitored to identify potentially suspicious account-management activity.

## 5. MITRE ATT&CK Mapping

| Technique | ID | Relevance |
|---|---|---|
| Create Account | T1136 | Creation of an additional account |
| Create Account: Local Account | T1136.001 | Creation of a local Windows user account |

## 6. Investigation Conclusion

The detection successfully identified the creation of a new local user account.

The account was created intentionally using Atomic Red Team as part of a controlled simulation. Event ID 4726 subsequently confirmed its deletion during cleanup.

There is no evidence from this exercise of unauthorized account creation or actual persistence.

**Final assessment:** Detection successfully identified simulated local account creation and the investigation established the account creation and deletion sequence.

## 7. Recommended SOC Analyst Actions

- Identify the account creator and the newly created account.
- Verify whether account creation was authorized through a change request or administrative activity.
- Review account group memberships and assigned privileges.
- Investigate unexpected account creation, especially on critical endpoints.
- Correlate account creation with subsequent logons and process activity.
- Escalate unexplained account creation to the incident response team.

## 8. Lessons Learned

- Event ID 4720 records user account creation.
- Event ID 4726 records user account deletion.
- Account creation can be abused for persistence.
- Correlating account creation with account deletion provides useful investigation context.
- Administrative account activity should be monitored and validated against authorized changes.