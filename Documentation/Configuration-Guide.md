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
