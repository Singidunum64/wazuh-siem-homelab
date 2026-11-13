# Installation Guide

## 1. Purpose and scope

This guide describes how to reproduce the Wazuh SIEM homelab at a high level. It covers the three-machine topology, the VirtualBox network, the Wazuh server, the Windows agent, Sysmon, and initial validation.

The environment documented here uses:

| System | Role | Host-Only address |
|---|---|---|
| Linux Mint host | Wazuh Manager, Indexer, Dashboard | `192.168.56.1` |
| Windows 10 VM | Monitored endpoint; Wazuh Agent and Sysmon | `192.168.56.101` |
| Kali Linux VM | Attack and testing system | `192.168.56.102` |

The Linux Mint machine also has a LAN address (`192.168.1.15`) used to access the dashboard at `https://192.168.1.15`. The Windows agent uses the manager's Host-Only address, `192.168.56.1`, for lab communication.

> **Version note:** The Windows agent used in the lab is Wazuh Agent 4.14.4 and Sysmon 15.20. Record the exact Wazuh server and operating-system versions from the deployed environment before attempting a version-for-version rebuild.

## 2. Prerequisites

- Linux Mint host with sufficient memory, storage, and CPU resources.
- Oracle VirtualBox.
- Windows 10 virtual machine.
- Kali Linux virtual machine.
- Internet access for downloading packages and installation media.
- Administrative access to the host and both virtual machines.

The lab is intended for isolated, authorized testing. Do not point the attack VM at systems outside the lab.

## 3. Create the VirtualBox Host-Only network

Create a VirtualBox Host-Only network with the following addressing plan:

| Setting | Value |
|---|---|
| Network | `192.168.56.0/24` |
| Linux Mint host interface | `192.168.56.1` |
| Windows VM | `192.168.56.101` |
| Kali VM | `192.168.56.102` |
| DHCP | Disabled in the documented setup; addresses are configured manually |

Attach a Host-Only adapter to both VMs. Keep a separate NAT adapter if internet access is required.

On the Linux Mint host, inspect the Host-Only interface:

```bash
ip addr show vboxnet0
```

If the interface exists but is down, bring it up:

```bash
sudo ip link set vboxnet0 up
```

Verify the IP configuration on Windows and Kali, then test connectivity between the lab systems.

## 4. Install the Wazuh server

The Wazuh server components run on the Linux Mint host. The deployment includes:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

Use the official Wazuh installation documentation and select a supported all-in-one deployment procedure appropriate to the target Wazuh release:

https://documentation.wazuh.com/current/installation-guide/index.html

Keep the Manager, Indexer, Dashboard, and agent versions compatible. Record the exact release and installation method used in the project before publishing a fully reproducible, command-by-command build guide.

After installation, check the services:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

The dashboard in this lab is accessed on:

`https://192.168.1.15`

## 5. Configure host firewall access

The Linux Mint host uses UFW. The Windows endpoint must be able to reach the Wazuh Manager over the Host-Only network.

The lab uses these TCP ports:

| Port | Purpose |
|---|---|
| `1514/TCP` | Wazuh agent communication |
| `1515/TCP` | Agent enrollment |

Allow access only from the Host-Only subnet, using the appropriate UFW rules for the local deployment. For example:

```bash
sudo ufw allow from 192.168.56.0/24 to any port 1514 proto tcp
sudo ufw allow from 192.168.56.0/24 to any port 1515 proto tcp
sudo ufw status verbose
```

## 6. Install and enroll the Windows agent

Install the Wazuh Agent on the Windows 10 VM. The documented lab used Wazuh Agent 4.14.4.

Official Windows agent instructions:

https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html

Configure the manager address as:

`192.168.56.1`

The Windows agent configuration file is:

`C:\Program Files (x86)\ossec-agent\ossec.conf`

## 7. Install Sysmon

Install Microsoft Sysinternals Sysmon on the Windows endpoint. The documented lab used Sysmon 15.20.

Official download and documentation:

https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

Apply the Sysmon configuration used by the lab and verify that events are being written to:

`Microsoft-Windows-Sysmon/Operational`

## 8. Prepare Kali Linux

Attach Kali Linux to the same VirtualBox Host-Only network and configure its lab address as:

`192.168.56.102`

NetExec is used for the documented SMB authentication testing. Use it only against the intentionally configured Windows lab endpoint.

## 9. Validate the installation

Before running any attack exercise, confirm:

- The Host-Only interface is up on the Linux Mint host.
- Windows and Kali have the expected Host-Only addresses.
- The Wazuh Manager, Indexer, and Dashboard services are running.
- The Windows agent is active in the Wazuh Dashboard.
- TCP ports 1514 and 1515 are reachable from Windows where required.
- Windows Security events are arriving in Wazuh.
- Sysmon events are arriving in Wazuh.
- Kali can reach the Windows endpoint over the Host-Only network.

Once these checks pass, use the individual exercise guides in `attacks/`.
