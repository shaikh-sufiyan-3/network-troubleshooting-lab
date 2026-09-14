# 01 - No Network Connectivity

## Overview

This project demonstrates an L1 network troubleshooting scenario where a user reports that the computer is connected to the network but cannot access the network.

The issue is investigated step-by-step using basic endpoint and switch-level checks.

## User Complaint

> "My computer is connected to the network, but I cannot access the network."

## Network Topology

```text
PC1 ───── SW1 ───── R1
         Fa0/1      G0/0
```

## Network Information

| Device | Interface | Configuration    |
| ------ | --------- | ---------------- |
| PC1    | NIC       | 192.168.10.10/24 |
| SW1    | Fa0/1     | Access VLAN 10   |
| SW1    | Fa0/24    | Access VLAN 10   |
| R1     | G0/0      | 192.168.10.1/24  |

### PC1 Configuration

* IP Address: `192.168.10.10`
* Subnet Mask: `255.255.255.0`
* Default Gateway: `192.168.10.1`

## Problem

PC1 had a valid IP configuration and the physical link was connected, but it could not reach its default gateway.

Investigation showed that SW1 interface `Fa0/1` was incorrectly assigned to **VLAN 20** instead of **VLAN 10**.

## Root Cause

The root cause was an incorrect VLAN assignment on SW1:

```text
Fa0/1 → VLAN 20
```

PC1 belonged to the `192.168.10.0/24` network, which was associated with VLAN 10. Because the switch port was placed in VLAN 20, PC1 could not communicate with the gateway in VLAN 10.

## Solution

The switch port was moved from VLAN 20 to VLAN 10:

```text
SW1(config)# interface fa0/1
SW1(config-if)# switchport access vlan 10
```

## Verification

After correcting the VLAN assignment:

```text
Fa0/1 → VLAN 10
```

PC1 successfully reached the default gateway:

```text
Reply from 192.168.10.1
Reply from 192.168.10.1
Reply from 192.168.10.1
Reply from 192.168.10.1
```

Final result:

```text
Sent = 4, Received = 4, Lost = 0 (0% loss)
```

## Skills Demonstrated

* L1 network troubleshooting
* Endpoint IP configuration verification
* Basic connectivity testing
* Switch port status verification
* VLAN verification
* Root-cause identification
* Access-port troubleshooting
* Connectivity verification after remediation

## Troubleshooting Methodology

```text
User Complaint
      ↓
Check PC IP Configuration
      ↓
Test Default Gateway
      ↓
Check Switch Port Status
      ↓
Check VLAN Assignment
      ↓
Identify Root Cause
      ↓
Correct VLAN Assignment
      ↓
Retest Connectivity
      ↓
Confirm Resolution
```
