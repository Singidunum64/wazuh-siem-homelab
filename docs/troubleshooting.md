# Troubleshooting Guide

## 1. Purpose

This guide documents practical checks for common connectivity, agent, telemetry, and detection problems in the Wazuh SIEM homelab. The examples are based on issues encountered while building this lab.

## 2. Windows agent is disconnected

### Symptoms

- The Windows endpoint appears disconnected in the Wazuh Dashboard.
- New Windows events are not appearing in Wazuh.

### Checks

Check the Manager:

```bash
sudo systemctl status wazuh-manager
```

Check whether the agent ports are listening:

```bash
sudo ss -lntp | grep -E '1514|1515'
```

On Windows PowerShell, test connectivity to the Host-Only manager address:

```powershell
Test-NetConnection 192.168.56.1 -Port 1514
Test-NetConnection 192.168.56.1 -Port 1515
```

If the TCP tests fail, check the Host-Only interface state and UFW rules before changing the agent configuration.

## 3. Agent enrollment fails

### Symptoms

The Wazuh Agent service is running, but the agent log reports that it cannot connect to the enrollment service at `192.168.56.1:1515`.

### Checks

1. Confirm that the Windows agent's configured manager address is `192.168.56.1`.
2. Confirm that the Wazuh Manager is running.
3. Confirm that TCP port 1515 is listening.
4. Confirm that UFW permits TCP 1515 from `192.168.56.0/24`.
5. Confirm that the VirtualBox Host-Only adapter is up.

Review the Windows agent log:

`C:\Program Files (x86)\ossec-agent\ossec.log`

## 4. Host-Only network is unavailable

### Symptoms

- Windows can reach the host, but the reverse direction fails.
- Windows or Kali has an unexpected `169.254.x.x` address.
- The `vboxnet0` interface is shown as down.

### Checks on the host

```bash
VBoxManage list hostonlyifs
ip addr show vboxnet0
```

If `vboxnet0` exists but is down:

```bash
sudo ip link set vboxnet0 up
```

Then verify that the Windows and Kali VMs have their Host-Only adapters attached and that their IP addresses belong to `192.168.56.0/24`.

## 5. Wazuh Manager is inactive

### Symptoms

- The dashboard does not show new endpoint events.
- Ports 1514 and 1515 are not listening.
- The Manager service is inactive.

### Checks

```bash
sudo systemctl status wazuh-manager
```

If the service is stopped, inspect the service output and Wazuh logs before attempting a restart:

```bash
sudo journalctl -u wazuh-manager -n 100 --no-pager
```

After resolving the cause, start or restart the service as appropriate:

```bash
sudo systemctl start wazuh-manager
```

Then verify the service state and listening ports again.

## 6. Sysmon events exist locally but do not appear in Wazuh

### Symptoms

Sysmon events are visible in Windows Event Viewer or PowerShell, but no corresponding events appear in Wazuh Discover.

### Checks

Confirm that the Sysmon channel contains events:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10
```

Then check:

1. The Wazuh Agent configuration includes the Sysmon operational channel.
2. The agent service is running and connected.
3. The agent log does not show event subscription errors.
4. The Discover query is using the correct event fields.

For this project, the event ID field is:

```text
data.win.system.eventID
```

## 7. Windows Security events do not appear in Wazuh

### Symptoms

An event is visible in the local Windows Security log but cannot be found in Discover.

### Checks

Inspect recent local Security events:

```powershell
Get-WinEvent -LogName Security -MaxEvents 20
```

Confirm that the Wazuh Agent is configured to collect the Security channel.

## 8. Custom rule 100100 does not fire

### Symptoms

Failed SMB logons are visible in Discover, but the expected custom alert is absent.

### Checks

1. Confirm that the underlying failed logon events are arriving.
2. Inspect their fields and confirm they match the conditions in `detections/local_rules.xml`.
3. Confirm that the local rule is present on the Manager.
4. Check the frequency and time-window requirements. A single failed logon is not expected to satisfy a rule for repeated failures.
5. Inspect the Manager logs for rule-loading or configuration errors.

Rule `100100` is intended to detect repeated SMB authentication failures. It should not be expected to alert on unrelated account, process, or scheduled-task events.

## 9. Dashboard or API reports connection problems

### Symptoms

The dashboard reports that API connections may be unavailable or data cannot be retrieved.

### Checks

Check the relevant services:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

Inspect the service logs for startup, authentication, or connectivity errors. 

## 10. General diagnostic workflow

When telemetry is missing or a detection does not behave as expected, follow this order:

1. Verify that the event exists on the Windows endpoint.
2. Verify that the Host-Only network is operational.
3. Verify that the Wazuh Agent is running and connected.
4. Inspect the agent log for collection or forwarding errors.
5. Search for the event in Wazuh Discover.
6. Inspect the event fields and timestamp.
7. Check whether the expected Wazuh rule conditions are satisfied.
8. Review Manager logs if the event is received but the alert is not generated.
