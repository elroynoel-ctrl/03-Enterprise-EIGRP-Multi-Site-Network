# Configuration Guide

## Project Overview

This guide documents the complete configuration process for the **Enterprise EIGRP Multi-Site Network** developed as part of the **Project Elevate** portfolio.

The network simulates a small enterprise consisting of a Headquarters (HQ), three branch offices (BR1, BR2, and BR3), and an ISP connected through a resilient EIGRP ring topology.

The project demonstrates enterprise routing, centralized network services, redundancy, and Internet access using Cisco IOS.

---

# Network Features

The completed network includes:

- EIGRP Autonomous System 100
- Four enterprise routers
- One ISP router
- Four Layer 2 switches
- Router-on-a-Stick inter-VLAN routing
- VLAN segmentation
- Centralized DHCP Server
- Centralized DNS Server
- DHCP Relay (`ip helper-address`)
- NAT Overload (PAT)
- Static default route redistribution into EIGRP
- Equal-Cost Multi-Path (ECMP)
- Simulated Internet connectivity

---

# Enterprise Topology

The topology consists of:

```text
                 ISP
                  |
             209.165.200.224/30
                  |
                 HQ
              /       \
          BR1 ------- BR3
             \       /
               BR2
```

The enterprise ring provides redundant paths between all locations while maintaining dynamic routing through EIGRP.

---

# VLAN Design

## Headquarters

| VLAN | Name | Network |
|------|------|----------------|
|10|Management|192.168.10.0/24|
|20|Users|192.168.20.0/24|
|30|Servers|192.168.30.0/24|
|99|Native|Native VLAN|

---

## BR1

| VLAN | Name | Network |
|------|------|----------------|
|40|Users|192.168.40.0/24|
|50|Servers|192.168.50.0/24|
|99|Native|Native VLAN|

---

## BR2

| VLAN | Name | Network |
|------|------|----------------|
|60|Users|192.168.60.0/24|
|70|Servers|192.168.70.0/24|
|99|Native|Native VLAN|

---

## BR3

| VLAN | Name | Network |
|------|------|----------------|
|80|Users|192.168.80.0/24|
|90|Servers|192.168.90.0/24|
|99|Native|Native VLAN|

---

# WAN Addressing

| Link | Network |
|------|----------------|
|HQ — BR1|10.0.12.0/30|
|BR1 — BR2|10.0.23.0/30|
|BR2 — BR3|10.0.34.0/30|
|BR3 — HQ|10.0.41.0/30|
|HQ — ISP|209.165.200.224/30|

---

# Loopback Addresses

| Router | Loopback |
|---------|-----------|
|HQ|1.1.1.1/32|
|BR1|2.2.2.2/32|
|BR2|3.3.3.3/32|
|BR3|4.4.4.4/32|

These loopbacks were used as stable EIGRP router IDs.
---

# Switch Configuration

Each site uses a Layer 2 switch to provide VLAN segmentation and connectivity to the local router using Router-on-a-Stick.

## Configure VLANs

Example (HQ Switch):

```text
vlan 10
 name Management

vlan 20
 name Users

vlan 30
 name Servers

vlan 99
 name Native
```

Configure the branch switches using their respective VLAN IDs.

---

## Configure Access Ports

Assign end devices to their appropriate VLAN.

Example:

```text
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 20

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 30
```

---

## Configure the Trunk

The router connection is configured as an IEEE 802.1Q trunk.

Example:

```text
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 99
```

---

# Router-on-a-Stick Configuration

Each router uses subinterfaces for inter-VLAN routing.

Example (HQ):

```text
interface GigabitEthernet0/2.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/2.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.30.100

interface GigabitEthernet0/2.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
```

Branch routers use the same design with their respective VLANs and IP addressing.

---

# EIGRP Configuration

All enterprise routers participate in EIGRP Autonomous System 100.

Example:

```text
router eigrp 100
 eigrp router-id 1.1.1.1
 passive-interface default
 no passive-interface GigabitEthernet0/0
 no passive-interface GigabitEthernet0/1

 network 10.0.12.0 0.0.0.3
 network 10.0.41.0 0.0.0.3
 network 192.168.10.0
 network 192.168.20.0
 network 192.168.30.0
 network 1.1.1.1 0.0.0.0
```

Each branch router uses the same configuration model with its own router ID and local networks.

---

# DHCP Configuration

The centralized DHCP server resides on the HQ Servers VLAN.

Configured scopes include:

- HQ Users
- BR1 Users
- BR2 Users
- BR3 Users

Each scope contains:

- Network
- Default Gateway
- DNS Server
- Starting Address
- Maximum Users

---

# DHCP Relay

Because the DHCP server resides at HQ, each branch Users VLAN requires DHCP relay.

Example:

```text
interface GigabitEthernet0/2.20
 ip helper-address 192.168.30.100
```

This forwards DHCP broadcasts to the centralized DHCP server.

---

# NAT Overload (PAT)

The HQ router performs Internet address translation.

Configuration includes:

```text
access-list 1 permit 10.0.0.0 0.255.255.255
access-list 1 permit 192.168.0.0 0.0.255.255

ip nat inside source list 1 interface Serial0/0/0 overload
```

Interfaces were designated as inside or outside accordingly.

---

# Default Route Redistribution

The HQ router provides Internet access for the enterprise.

Configuration:

```text
ip route 0.0.0.0 0.0.0.0 209.165.200.226
```

The static default route is redistributed into EIGRP.

```text
router eigrp 100
 redistribute static
```

This allows all branch routers to learn the enterprise exit point dynamically.

---

# ISP Configuration

The ISP router simulates Internet connectivity.

Configuration includes:

- WAN connection to HQ
- Loopback interface (8.8.8.8)
- Connected WAN network

The ISP provides external reachability for PAT verification.

---

# Configuration Verification

After configuration, verify the following:

## Routing

```text
show ip route
show ip protocols
```

## EIGRP

```text
show ip eigrp neighbors
show ip eigrp topology
```

## Interfaces

```text
show ip interface brief
```

## NAT

```text
show ip nat translations
show ip nat statistics
```

## DHCP

```text
show ip interface
show ip dhcp binding
```

## Connectivity

Verify:

- Router loopback connectivity
- Branch-to-branch communication
- DHCP server reachability
- DNS server reachability
- Internet connectivity
- Client DHCP operation

---

# Configuration Complete

The Enterprise EIGRP Multi-Site Network is now fully configured and ready for verification, troubleshooting, and documentation.

The completed implementation demonstrates enterprise routing, centralized services, resilient WAN connectivity, and secure Internet access using Cisco EIGRP and NAT Overload (PAT).
