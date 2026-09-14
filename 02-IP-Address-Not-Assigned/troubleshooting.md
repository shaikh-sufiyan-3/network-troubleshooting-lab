# Troubleshooting - IP Address Not Assigned

## 1. User Complaint

The user reported:

> "My computer is connected to the network, but it did not receive an IP address."

The first troubleshooting objective was to determine whether the issue was related to the client configuration, physical connectivity, VLAN, router interface, or DHCP service.

---

## 2. Check PC IP Configuration

Command:

```text
ipconfig
```

### Actual Result

PC1 received:

```text
IP Address      : 169.254.102.58
Subnet Mask     : 255.255.0.0
Default Gateway : 0.0.0.0
```

### Observation

PC1 received a `169.254.x.x` address.

This is an APIPA address and indicates that the client did not receive a valid DHCP lease.

The expected network was:

```text
192.168.10.0/24
```

Therefore, the client did not have the expected network configuration.

---

## 3. Attempt DHCP Renewal

Command:

```text
ipconfig /renew
```

### Actual Result

The DHCP request failed.

### Observation

The renewal failure confirmed that PC1 was unable to obtain a DHCP address.

The investigation was continued from the client toward the network and DHCP server.

---

## 4. Test Default Gateway

Command:

```text
ping 192.168.10.1
```

### Actual Result

```text
Sent = 4
Received = 0
Lost = 4 (100% loss)
```

### Observation

PC1 could not reach the default gateway.

This was expected because PC1 did not have a valid `192.168.10.0/24` address or a valid default gateway.

The router interface was checked next.

---

## 5. Check Router Interface

Command:

```text
show ip interface brief
```

### Relevant Result

```text
GigabitEthernet0/0
IP Address: 192.168.10.1
Status: up
Protocol: up
```

### Observation

R1 G0/0 was:

* Configured with the correct IP address
* Administratively up
* Operationally up

Therefore, the router interface was not the cause of the DHCP failure.

---

## 6. Check DHCP Configuration

Command:

```text
show running-config | section dhcp
```

### Actual DHCP Configuration

```text
ip dhcp pool WRONG-POOL
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
```

### Observation

The DHCP pool was configured for:

```text
192.168.20.0/24
```

However, the client and router interface were using:

```text
192.168.10.0/24
```

### Network Comparison

```text
PC1 Network       → 192.168.10.0/24
R1 G0/0           → 192.168.10.1/24
DHCP Pool         → 192.168.20.0/24   ← Incorrect
DHCP Gateway      → 192.168.20.1     ← Incorrect
```

The DHCP network did not match the client network.

---

## 7. Root Cause

The root cause was an incorrectly configured DHCP pool.

The DHCP pool was configured for the `192.168.20.0/24` network instead of the required `192.168.10.0/24` network.

As a result, PC1 could not obtain a valid DHCP lease and received an APIPA address.

---

## 8. Apply Fix

The DHCP configuration was corrected to match the client network:

```text
Network:       192.168.10.0/24
Default Router: 192.168.10.1
```

The client was then renewed to obtain a new DHCP lease.

---

## 9. Verify Client IP Address

After the DHCP configuration was corrected, PC1 successfully received:

```text
IP Address      : 192.168.10.11
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.10.1
```

### Observation

The client now had:

* A valid IP address
* Correct subnet mask
* Correct default gateway

The APIPA address was no longer present.

---

## 10. Verify Gateway Connectivity

Command:

```text
ping 192.168.10.1
```

### Actual Result

```text
Reply from 192.168.10.1
Reply from 192.168.10.1
Reply from 192.168.10.1
Reply from 192.168.10.1

Sent = 4
Received = 4
Lost = 0 (0% loss)
```

### Observation

PC1 successfully reached the default gateway with `0% packet loss`.

---

## Final Result

**Problem:** PC1 did not receive a valid IP address.

**Root Cause:** DHCP pool was configured for the wrong subnet (`192.168.20.0/24`).

**Corrective Action:** DHCP pool was corrected to the `192.168.10.0/24` network with gateway `192.168.10.1`.

**Verification:** PC1 received `192.168.10.11` through DHCP and successfully pinged the gateway with `0% packet loss`.

**Status:** RESOLVED

## Troubleshooting Conclusion

The issue was isolated by starting at the endpoint and progressively checking DHCP renewal, gateway connectivity, router interface status, and DHCP configuration.

The evidence confirmed that the physical/router interface was operational and the actual fault was a DHCP network mismatch.
