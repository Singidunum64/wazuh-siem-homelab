# Windows Network Authentication Brute Force

## Overview

This test generated repeated failed SMB authentication attempts from Kali Linux against a Windows 10 endpoint. Windows recorded the failures as Security Event ID `4625`, which were collected and analyzed by Wazuh.

The test verified that a custom Wazuh rule could correlate repeated network logon failures from the same source IP within a defined time window.

## Lab Systems

| System | Role | IP address |
|---|---|---|
| Kali Linux | Attack system | `192.168.56.102` |
| Windows 10 | Target endpoint | `192.168.56.101` |
| Linux Mint | Wazuh server | `192.168.56.1` |

The test was performed on the isolated VirtualBox Host-Only network.

## Test Procedure

The target's SMB service was checked from Kali, and repeated authentication attempts were made using NetExec with an incorrect password.

```bash
netexec smb 192.168.56.101 -u b82g1 -p 'WrongPassword123!'
```

Five failed attempts were then generated with a short delay between requests. NetExec returned `STATUS_LOGON_FAILURE` for the attempts.

## Windows Telemetry

The Windows Security log recorded failed network logons with Event ID `4625`.

Relevant fields included:

- Logon Type: `3` (network logon)
- Source IP: `192.168.56.102`
- Target account: `b82g1`
- Authentication package: `NTLM`
- Failure status: `0xC000006D`
- Substatus: `0xC000006A`

The substatus indicates an incorrect password for the account.

## Wazuh Detection

Wazuh's built-in rule `60122` identifies a failed logon caused by an unknown user or incorrect password.

The custom rule `100100` correlates five matching failures from the same source IP within 60 seconds, requiring Logon Type `3`.

| Rule | Purpose |
|---|---|
| `60122` | Detects an individual Windows logon failure |
| `100100` | Correlates repeated network authentication failures |

The custom rule generates a level 10 alert and maps the activity to MITRE ATT&CK technique `T1110` (Brute Force).

Although the simulated attempts used SMB, the custom rule detects repeated Windows network authentication failures rather than SMB exclusively.

## MITRE ATT&CK

- **Tactic:** Credential Access
- **Technique:** [T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)

## Evidence

Screenshots for this test are stored in [`../evidence/brute-force/`](../evidence/brute-force/).

1. `01-kali-attack.png` - NetExec authentication attempts
2. `02-wazuh-correlation.png` - Wazuh detection results
3. `03-mitre-attack.png` - MITRE ATT&CK mapping
4. `04-alert-json.png` - Wazuh alert details and correlated events
5. `05-windows-events.png` - Windows-side authentication evidence

## Result

The test successfully generated Windows network logon failures, and Wazuh produced alerts from the built-in detection rule and the custom correlation rule.

This confirms that the lab can collect Windows authentication telemetry and correlate repeated failures into a higher-severity alert.

The rule is a lab detection and should be validated against legitimate authentication patterns before use in production.
