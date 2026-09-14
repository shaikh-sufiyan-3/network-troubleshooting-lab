# Troubleshooting - No Network Connectivity

## 1. User Complaint

The user reported:

> "My computer is connected to the network, but I cannot access the network."

The first troubleshooting objective was to determine whether the problem was related to the PC configuration, physical connectivity, VLAN assignment, or gateway connectivity.

---

## 2. Check PC IP Configuration

Command:

```text
ipconfig
```

Actual output:

```text
C:\>ipconfig

FastEthernet0 Connection:(default port)
IPv4 Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
```

### Observation

PC1 had:

* Valid IP address: `192.168.10.10`
* Correct subnet mask: `255.255.255.0`
* Default gateway configured: `192.168.10.1`

The PC configuration did not immediately indicate an IP addressing problem.

---

## 3. Test Default Gateway

Command:

```text
ping 192.168.10.1
```

Actual result:

```text
Request timed out.
Request timed out.
Request timed out.
Request timed out.

Sent = 4, Received = 0, Lost = 4 (100% loss)
```

### Observation

PC1 could not reach its default gateway.

The next step was to check the switch port and VLAN assignment.

---

## 4. Check Switch Port Status

Command:

```text
SW1#show interfaces status
```

Relevant output:

```text
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1                        connected    20         a-full  a-100 10/100BaseTX
```

### Observation

`Fa0/1` was:

* Physically connected
* Operationally up
* Assigned to VLAN 20

The physical link was therefore not the primary problem.

---

## 5. Check VLAN Assignment

Command:

```text
SW1#show vlan brief
```

Relevant output:

```text
1    default    active    Fa0/2, Fa0/3, ...
10   SALES      active    Fa0/24
20   HR         active    Fa0/1
```

### Observation

PC1 was connected to `Fa0/1`, but `Fa0/1` was assigned to VLAN 20.

The expected VLAN for PC1 was VLAN 10.

### Fault Isolation

```text
PC1
192.168.10.10/24
     │
     │
   Fa0/1
     │
   VLAN 20  ← Incorrect
     │
   SW1
```

The incorrect VLAN assignment was identified as the root cause.

---

## 6. Apply Fix

The switch port was moved to VLAN 10:

```text
SW1(config)#interface fa0/1
SW1(config-if)#switchport access vlan 10
```

---

## 7. Verify VLAN Assignment

After the change, `show vlan brief` showed:

```text
10   SALES      active    Fa0/1, Fa0/24
20   HR         active
```

### Observation

`Fa0/1` was now correctly assigned to VLAN 10.

---

## 8. Verify Connectivity

PC1 was tested again:

```text
C:\>ping 192.168.10.1
```

Actual result:

```text
Reply from 192.168.10.1
Reply from 192.168.10.1
Reply from 192.168.10.1
Reply from 192.168.10.1

Sent = 4, Received = 4, Lost = 0 (0% loss)
Minimum = 0ms, Maximum = 13ms, Average = 3ms
```

## Final Result

**Issue resolved.**

### Root Cause

`SW1 Fa0/1` was incorrectly assigned to VLAN 20.

### Corrective Action

`Fa0/1` was assigned to VLAN 10.

### Verification

PC1 successfully reached the default gateway with:

```text
0% packet loss
```

## Troubleshooting Conclusion

The issue was isolated by moving systematically from the endpoint configuration to gateway testing and then to switch port/VLAN verification.

The evidence confirmed that the problem was a **switch access-port VLAN misconfiguration**, not an IP addressing or physical-link failure.
