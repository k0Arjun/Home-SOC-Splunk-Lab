# Incident Investigation 01 — Repeated Failed Logons

## 1. Incident Overview

| Field | Details |
|---|---|
| Incident Name | Repeated Failed Logons |
| Host | ARJUN-LAB |
| Target Account | soc_testuser |
| Log Source | Windows Security Event Logs |
| Relevant Event IDs | 4625 — Failed Logon; 4624 — Successful Logon |
| MITRE ATT&CK | T1110 — Brute Force; T1110.001 — Password Guessing |
| Severity | Medium (lab detection classification) |
| Investigation Status | Completed — Controlled Lab Simulation |

## 2. Alert Description

Multiple failed authentication attempts were observed against the local account `soc_testuser`.

A successful authentication was also observed immediately following failed attempts. The failed and successful authentication activity involved the same account and source IP address.

This behavior can be consistent with password guessing followed by successful authentication.

## 3. Evidence Collected

| Evidence | Observation |
|---|---|
| Failed authentication events | 17 Event ID 4625 events |
| Successful authentication events | 33 Event ID 4624 events |
| Target account | soc_testuser |
| Source IP observed | ::1 (IPv6 loopback) |
| Logon Type | 3 — Network |
| Authentication package | NTLM observed in failed-logon events |
| Authentication sequence | Successful logon observed immediately after failed attempts |

**Count interpretation:** The 17 failed and 33 successful events are aggregate counts from the selected search period. They do not establish that every successful event followed a failed attempt.

## 4. Investigation and Analysis

1. Reviewed Windows Security Event ID 4625 to identify failed authentication attempts.
2. Reviewed Event ID 4624 to identify successful authentication activity.
3. Compared the target account and source IP across the failed and successful events.
4. Confirmed that the same account and source IP were involved.
5. Observed Logon Type 3, indicating network authentication.
6. Identified `::1` as the IPv6 loopback address, representing the local machine.

The authentication activity was generated during controlled testing using the Windows `net use` command against the local IPC$ share with intentionally incorrect passwords.

## 5. MITRE ATT&CK Mapping

| Technique | ID | Relevance |
|---|---|---|
| Brute Force | T1110 | Repeated authentication failures |
| Password Guessing | T1110.001 | Multiple incorrect password attempts |

## 6. Investigation Conclusion

The observed authentication pattern resembled password guessing followed by successful authentication.

However, the source IP `::1` identifies loopback activity, and the events were generated intentionally as part of the controlled SOC lab exercise.

There is no evidence from this test alone of an external attacker or actual account compromise.

**Final assessment:** Detection successfully identified the simulated authentication pattern. The investigation demonstrated how failed and successful logons can be correlated using Windows Security Event Logs.

## 7. Recommended SOC Analyst Actions

- Correlate failed and successful authentication events by account, source IP and timestamp.
- Investigate unusual source addresses and unexpected network logons.
- Review subsequent activity performed by the successfully authenticated account.
- Escalate when authentication behavior is unexplained or associated with additional suspicious activity.
- Tune thresholds based on normal authentication patterns in the monitored environment.

## 8. Lessons Learned

- Event ID 4625 identifies failed logons.
- Event ID 4624 identifies successful logons.
- Logon Type 3 represents network authentication.
- The IPv6 loopback address `::1` represents the local system.
- A successful logon after repeated failures warrants investigation but does not independently prove compromise.