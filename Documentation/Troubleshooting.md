# Troubleshooting

## Overview

This document records the troubleshooting process used during the deployment and verification of the **Enterprise EIGRP Multi-Site Network**.

Rather than assuming the network was fully operational after configuration, each service was systematically validated. Any issues encountered were isolated, analyzed, corrected, and reverified before the project was considered complete.

---

# Troubleshooting Methodology

The following troubleshooting approach was used throughout the project:

1. Verify physical and interface connectivity.
2. Verify IP addressing.
3. Verify EIGRP neighbor adjacencies.
4. Verify routing table population.
5. Verify VLAN and Router-on-a-Stick operation.
6. Verify DHCP services.
7. Verify DHCP relay (`ip helper-address`).
8. Verify NAT Overload (PAT).
9. Verify Internet connectivity.
10. Validate end-to-end communication from client devices.

This structured process ensured that each network service was validated independently before moving to the next layer.

---

# Issue 1 – HQ Clients Failed to Receive DHCP Addresses

## Symptoms

During client verification, HQPC1 failed to obtain an IPv4 address from the centralized DHCP server.

Observed behavior:

- DHCP request failed.
- No IPv4 address assigned.
- Unable to ping the default gateway.
- Unable to reach the centralized DHCP/DNS server.

---

## Investigation

The following components were verified:

- DHCP server status
- DHCP address pool
- HQ switch VLAN assignments
- Router-on-a-Stick configuration
- Trunk connectivity
- Client VLAN membership

All components were operating correctly.

The remaining verification focused on the HQ router subinterfaces.

The command:

```text
show ip interface GigabitEthernet0/2.20
```

revealed:

```text
Helper address is not set
```

---

## Root Cause

The Users VLAN interface on the HQ router was missing the DHCP relay configuration.

Because the DHCP server resided on a different VLAN, DHCP broadcast messages were not being forwarded.

---

## Resolution

Configured the DHCP relay:

```text
interface GigabitEthernet0/2.20
 ip helper-address 192.168.30.100
```

---

## Verification

After renewing the DHCP lease, the client successfully received:

- IPv4 Address
- Subnet Mask
- Default Gateway
- DHCP Server
- DNS Server

Successful connectivity was verified to:

- Default Gateway
- Centralized DHCP/DNS Server
- Simulated Internet (8.8.8.8)

Result:

✅ Resolved

---

# Issue 2 – NAT Overload Verification

## Symptoms

Initial NAT verification displayed:

```text
Total translations: 0
```

---

## Investigation

The NAT configuration itself was correct.

The lack of translations was expected because no enterprise clients had generated Internet traffic.

---

## Resolution

Internet traffic was generated from enterprise clients by successfully pinging:

```text
8.8.8.8
```

---

## Verification

Subsequent verification confirmed:

- Dynamic PAT translations
- Translation hits
- Successful Internet connectivity

Result:

✅ Resolved

---

# Issue 3 – EIGRP Route Verification

## Investigation

Verification focused on confirming:

- Neighbor adjacencies
- Routing tables
- Topology database
- Equal-Cost Multi-Path (ECMP)

Commands used:

```text
show ip eigrp neighbors
show ip route
show ip eigrp topology
show ip protocols
```

---

## Verification

The enterprise routing domain successfully converged.

Verification confirmed:

- Stable neighbor relationships
- Complete routing table population
- ECMP operation
- Default route redistribution

Result:

✅ Verified

---

# Issue 4 – End-to-End Connectivity

Connectivity testing was performed from every router and representative client devices.

Verification included:

- Router loopback reachability
- Branch-to-branch communication
- DHCP/DNS server reachability
- ISP reachability
- Simulated Internet connectivity

All connectivity tests completed successfully.

Result:

✅ Verified

---

# Lessons from Troubleshooting

This project reinforced several important enterprise networking principles:

- Always verify Layer 1 through Layer 3 before troubleshooting routing.
- Validate routing before investigating application services.
- DHCP relay is required whenever DHCP clients and servers reside on different broadcast domains.
- NAT translations appear only after inside hosts generate traffic.
- Verification should be performed after every configuration change.
- Documentation is an essential part of enterprise network deployment.

---

# Troubleshooting Summary

| Issue | Status |
|-------------------------------|--------|
| Missing DHCP Relay | ✅ Resolved |
| DHCP Address Assignment | ✅ Verified |
| NAT Overload Operation | ✅ Verified |
| EIGRP Neighbor Relationships | ✅ Verified |
| Routing Table Population | ✅ Verified |
| ECMP Operation | ✅ Verified |
| Internet Connectivity | ✅ Verified |
| End-to-End Enterprise Connectivity | ✅ Verified |

## Overall Result

The Enterprise EIGRP Multi-Site Network successfully passed all troubleshooting and validation procedures. Every identified issue was resolved and fully verified before project completion.
