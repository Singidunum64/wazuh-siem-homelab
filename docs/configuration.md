# Configuration Guide

## 1. Purpose

This document records the important network, Wazuh, Windows Agent, Sysmon, and detection settings used by the homelab. It is intended to explain the deployed configuration, not replace the official product documentation.

## 2. Network topology

The lab uses a VirtualBox Host-Only network for communication between the Linux Mint host and the two VMs.

| System | Host-Only address | Function |
|---|---|---|
| Linux Mint | `192.168.56.1` | Wazuh server |
| Windows 10 VM | `192.168.56.101` | Monitored endpoint |
| Kali Linux VM | `192.168.56.102` | Attack and testing system |

The subnet is `192.168.56.0/24`.

Linux Mint also uses LAN address `192.168.1.15`, the Wazuh Dashboard is accessed at `https://192.168.1.15`. The Windows agent is configured to communicate with the manager using its Host-Only address, `192.168.56.1`.

## 3. Wazuh services and ports

The Linux Mint host runs:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

Agent communication ports:

| Port | Protocol | Function |
|---|---|---|
| `1514` | TCP | Agent event communication |
| `1515` | TCP | Agent enrollment |

The Manager configuration is stored at:

`/var/ossec/etc/ossec.conf`

Local rules are stored on the manager at:

`/var/ossec/etc/rules/local_rules.xml`

The project maintains the corresponding rule file in the repository at:

`detections/local_rules.xml`

## 4. Host firewall

UFW is enabled on the Linux Mint host. The lab permits the Windows Host-Only subnet (`192.168.56.0/24`) to reach TCP ports 1514 and 1515.

## 5. Windows Wazuh Agent

The Windows endpoint runs Wazuh Agent 4.14.4.

Configuration file:

`C:\Program Files (x86)\ossec-agent\ossec.conf`

Manager address:

`192.168.56.1`

Agent service:

`WazuhSvc`

Agent log:

`C:\Program Files (x86)\ossec-agent\ossec.log`

## 6. Sysmon collection

The Windows endpoint runs Sysmon 15.20. Its operational event channel is:

`Microsoft-Windows-Sysmon/Operational`

## 7. Custom detection rule

The repository's `detections/local_rules.xml` contains custom rule `100100`.

Rule `100100` detects repeated SMB authentication failures within a configured frequency/time window. It supplements the built-in Wazuh authentication detection rules.

## 8. Verification after configuration changes

After changing the Manager or agent configuration:

1. Check the configuration for syntax and rule errors.
2. Restart only the affected service when required.
3. Confirm the service is running.
4. Confirm the agent is active.
5. Verify that the relevant event channel is arriving in Wazuh Discover.
6. Confirm that the intended built-in or custom alert is generated.

For production guidance and configuration options, consult the official Wazuh documentation:

https://documentation.wazuh.com/current/
