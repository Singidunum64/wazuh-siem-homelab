# Suspicious PowerShell Execution — Encoded Command

## Overview

This scenario validates Wazuh's detection of a PowerShell process launched with the `-EncodedCommand` parameter. The test uses a harmless command that prints a unique marker to the console. It does not modify the system.


## Lab Environment

| Component | Details |
|---|---|
| Monitored endpoint | Windows 10 VM |
| Wazuh agent name | `WindowsUser` |
| Wazuh agent ID | `003` |
| Endpoint IP | `192.168.56.101` |
| Telemetry source | Sysmon Operational log |
| Windows event ID | `1` — Process Create |

## Test Procedure

The following commands construct a Base64-encoded PowerShell command and execute it. The decoded command only prints a test string.

```powershell
$command = "Write-Output 'WAZUH_ENCODED_POWERSHELL_TEST'"
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($command))
powershell.exe -NoProfile -EncodedCommand $encoded
```

Expected console output:

```text
WAZUH_ENCODED_POWERSHELL_TEST
```

## Detection Results

Wazuh generated a live alert for the process-creation event.

| Field | Observed value |
|---|---|
| Wazuh rule ID | `92057` |
| Alert level | `12` |
| Rule description | PowerShell.exe spawned a powershell process which executed a base64 encoded command |
| Agent | `WindowsUser` |
| Agent ID | `003` |
| Agent IP | `192.168.56.101` |
| Windows event ID | `1` |
| Event source | `Microsoft-Windows-Sysmon/Operational` |
| MITRE ATT&CK technique | `T1059.001` — PowerShell |
| MITRE tactic | Execution |

The detection was observed in the `wazuh-alerts` index and was searchable in Wazuh Discover. The event included the PowerShell image, command line, process details, user, and endpoint information.

## Validation

The following Discover query locates the alert:

```text
rule.id: "92057" AND agent.name: "WindowsUser"
```

To locate Sysmon process-creation events more generally:

```text
data.win.system.eventID: "1"
```

To inspect the encoded-command test specifically:

```text
data.win.system.eventID: "1" AND data.win.eventdata.commandLine: *EncodedCommand*
```

## MITRE ATT&CK Mapping

- **Tactic:** Execution
- **Technique:** [T1059.001 — Command and Scripting Interpreter: PowerShell](https://attack.mitre.org/techniques/T1059/001/)

## Evidence

Screenshots for this scenario are stored in:

`evidence/powershell-encoded-command/`

The evidence includes:

1. PowerShell test execution.
2. Wazuh alert in Discover.
3. Expanded event details.
4. MITRE ATT&CK mapping.

## Detection Limitations

- This scenario validates the existing Wazuh built-in rule `92057`. It does not implement a custom detection rule.
- Encoded PowerShell commands are not inherently malicious. Legitimate administrative scripts may also use encoded commands.
- The alert demonstrates detection of this specific behavior. It does not prove malicious intent or guarantee detection of every suspicious PowerShell technique.
- The test command is intentionally harmless and was executed in an isolated lab environment.



