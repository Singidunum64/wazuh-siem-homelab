# Windows Local Administrators Group Membership Change Detection

## Overview

This scenario demonstrates how Wazuh detects a change to the membership of the local Windows Administrators group. Adding an account to this group grants elevated local privileges and may indicate privilege escalation or unauthorized account manipulation.

A temporary local account was created specifically for this controlled lab test, added to the local Administrators group, and removed after the test.

## Lab Environment

| Component | Value |
|---|---|
| SIEM | Wazuh |
| Endpoint | Windows 10 |
| Agent name | `WindowsUser` |
| Agent ID | `003` |
| Endpoint IP | `192.168.56.101` |
| Windows Security Event ID | `4732` |
| Wazuh rule ID | `60154` |
| Wazuh alert level | `12` |

## Test Procedure

A temporary local account named `wazuh_lab_test` was created in an elevated PowerShell session.

The account was added to the local Administrators group:

```powershell
$password = ConvertTo-SecureString "LabTest-2026!Temp" -AsPlainText -Force

New-LocalUser -Name "wazuh_lab_test" `
    -Password $password `
    -Description "Temporary Wazuh privilege-change detection test"

Add-LocalGroupMember -Group "Administrators" -Member "wazuh_lab_test"
```

The test password above is a lab-only example. Do not reuse it for real accounts.

## Windows Event Evidence

Windows Security Event ID **4732** records the addition of a member to a security-enabled local group.

The event confirmed that a member was added to the local `Administrators` group. Relevant event fields included:

- `eventID`: `4732`
- `targetUserName`: `Administrators`
- `targetDomainName`: `Builtin`
- `targetSid`: `S-1-5-32-544`
- `subjectUserName`: `b82g1`

The event's `memberSid` identified the added account. The event message did not resolve the member SID to a username, so the SID is retained as the direct evidence.

## Wazuh Detection

Wazuh generated an alert using its built-in rule:

| Field | Observed value |
|---|---|
| Rule ID | `60154` |
| Rule description | `Administrators Group Changed` |
| Severity level | `12` |
| Event ID | `4732` |
| Agent | `WindowsUser` |
| Agent ID | `003` |
| MITRE ATT&CK mapping | `T1484` — as supplied by the observed Wazuh rule |

The alert was visible in the Wazuh Threat Hunting interface and contained the Windows security event details.

The ATT&CK mapping above reflects the rule's observed metadata; it should not be interpreted as proof that every local group membership change is a domain policy modification.

## Detection Workflow

1. A temporary local account was created on the Windows endpoint.
2. The account was added to the local Administrators group.
3. Windows generated Security Event ID `4732`.
4. The Wazuh agent collected and forwarded the event.
5. Wazuh rule `60154` generated a level-12 alert.
6. The alert was reviewed in Threat Hunting and supporting screenshots were saved.
7. The temporary account was removed from the Administrators group and deleted.

## Evidence

Screenshots are stored in `evidence/administrators-group-change/`.

| File | Description |
|---|---|
| `01-windows-event-4732.png` | Windows Security Event ID 4732 |
| `02-wazuh-alert.png` | Wazuh alert showing the detection |
| `03-event-details.png` | Alert fields and event details |
| `04-mitre-attack.png` | MITRE ATT&CK information displayed by Wazuh |

## Security Significance

Unexpected additions to the local Administrators group can indicate privilege escalation, persistence, or unauthorized changes to local access controls.

This detection is useful for identifying changes that warrant investigation. A single alert does not establish malicious intent; authorized administration and software installation can also change group membership.

Recommended investigation steps include:

- Identify the account that initiated the change.
- Verify the identity of the account added to the group.
- Confirm whether the change was authorized.
- Review nearby account creation, logon, and process execution events.
- Remove unauthorized membership and follow incident response procedures if compromise is suspected.

## Cleanup and Validation

After evidence collection, the temporary account `wazuh_lab_test` was removed from the local Administrators group and deleted.

The lab test demonstrated the end-to-end detection path from Windows Security auditing through Wazuh alert generation.



