# 02 - IP Address Not Assigned

## Overview

This project demonstrates an L1 network troubleshooting scenario where a user reports that the computer is connected to the network but did not receive an IP address.

The issue is investigated step-by-step using endpoint, switch, router interface, and DHCP checks.

## User Complaint

> "My computer is connected to the network, but it did not receive an IP address."

## Network Topology

```text
PC1 ───── SW1 ───── R1
         Fa0/1      G0/0
```

## Network Information

| Device | Interface | Configuration   |
| ------ | --------- | --------------- |
| PC1    | NIC       | DHCP            |
| SW1    | Fa0/1     | Access VLAN 10  |
| SW1    | Fa0/24    | Access VLAN 10  |
| R1     | G0/0      | 192.168.10.1/24 |
| R1     | DHCP      | 192.168.10.0/24 |

## Expected PC1 Address

PC1 should obtain an address from DHCP:

* IP Address: `192.168.10.x`
* Subnet Mask: `255.255.255.0`
* Default Gateway: `192.168.10.1`

## Problem

PC1 was unable to obtain a valid IP address through DHCP.

The client received an APIPA address:

`169.254.102.58`

The DHCP renewal also failed.

Investigation showed that the router's DHCP pool was configured for the wrong network:

```text
192.168.20.0/24
```

while the client network was:

```text
192.168.10.0/24
```

## Root Cause

The DHCP pool was configured with the wrong network and default gateway.

Incorrect DHCP configuration:

```text
Network:       192.168.20.0/24
Default Router: 192.168.20.1
```

Expected client network:

```text
Network:       192.168.10.0/24
Default Gateway: 192.168.10.1
```

## Solution

The DHCP configuration was corrected to provide addresses from the `192.168.10.0/24` network with the correct default gateway.

## Verification

After correcting the DHCP configuration, PC1 successfully obtained:

```text
IP Address      : 192.168.10.11
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.10.1
```

Gateway connectivity was also verified successfully:

```text
Sent = 4
Received = 4
Lost = 0 (0% loss)
```

## Final Result

| Item                 | Result   |
| -------------------- | -------- |
| IP Assignment        | PASS     |
| DHCP                 | PASS     |
| Default Gateway      | PASS     |
| Gateway Connectivity | PASS     |
| Packet Loss          | 0%       |
| Status               | RESOLVED |

## Skills Demonstrated

* L1 network troubleshooting
* DHCP troubleshooting
* APIPA address identification
* Endpoint IP configuration verification
* DHCP renewal
* Router interface verification
* DHCP pool verification
* Root-cause identification
* Connectivity verification after remediation

## Troubleshooting Methodology

```text
User Complaint
      ↓
Check Client IP Configuration
      ↓
Identify APIPA Address
      ↓
Attempt DHCP Renewal
      ↓
Test Gateway Connectivity
      ↓
Check Router Interface
      ↓
Check DHCP Configuration
      ↓
Identify DHCP Network Mismatch
      ↓
Correct DHCP Configuration
      ↓
Renew Client DHCP Lease
      ↓
Verify IP + Gateway Connectivity
```
