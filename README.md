# Enterprise Network Project

Enterprise network design using **VLANs, VTP, STP, Multilayer Switching, HSRP, DHCP, and OSPF**, built with two Core Multilayer Switches and two Edge Routers connected to an ISP.

## Project Overview

| Feature | Description |
|---|---|
| VLANs | 4 departments (IT, HR, Sales, Marketing) |
| VTP | Centralized VLAN management |
| STP | Rapid-PVST+ with Root/Secondary Bridges per VLAN |
| MLS | Inter-VLAN routing on Core switches |
| HSRP | Gateway redundancy between MLS1 & MLS2 |
| DHCP | Automatic IP assignment per VLAN |
| Edge Routers | Dual-homed internet connectivity via OSPF |

## Configuration Steps

1. Configure Trunk Links
2. Enable VTP & Create VLANs
3. Assign Ports to VLANs (Configure Access Ports)
4. Configure STP Root Config & PortFast
5. Enable Rapid STP Version
6. Configure MLS IPs & Enable Routing
7. Configure MLS HSRP
8. Configure MLS DHCP
9. Configure Routed Ports for MLS
10. Configure Edge Routers Interfaces IPs
11. Configure Default Route for Edge Routers
12. Configure Routing Protocol for Edge Routers & MLS

---

## 1️) MLS Trunk Config

```
en
conf t
 int range f0/1-4
  switchport trunk encapsulation dot1q
  switchport mode trunk
```

## 2️) MLS VTP & VLAN Config

```
en
conf t
 vtp domain cisco
 vlan 2
  name HR
 vlan 3
  name Sales
 vlan 4
  name MRK
```

## 3️) Access Switches – VLAN Port Assignment

```
en
conf t
 int range f0/7-12
  switchport access vlan 2
 int range f0/13-18
  switchport access vlan 3
 int range f0/19-24
  switchport access vlan 4
```

## 4️) STP Config

**MLS1**
```
en
conf t
 spanning-tree vlan 1 root primary
 spanning-tree vlan 2 root primary
 spanning-tree vlan 3 root secondary
 spanning-tree vlan 4 root secondary
```

**MLS2**
```
en
conf t
 spanning-tree vlan 3 root primary
 spanning-tree vlan 4 root primary
 spanning-tree vlan 1 root secondary
 spanning-tree vlan 2 root secondary
```

**Access Switches – PortFast**
```
en
conf t
 int range f0/1-24
  switchport mode access
  spanning-tree portfast
```

## 5️) Rapid STP (All Switches)

```
en
conf t
 spanning-tree mode rapid-pvst
```

## 6️) MLS Routing

**MLS1**
```
en
conf t
 ip routing

 int vlan 1
  no sh
  ip address 192.168.1.200 255.255.255.0
 int vlan 2
  no sh
  ip address 192.168.2.200 255.255.255.0
 int vlan 3
  no sh
  ip address 192.168.3.200 255.255.255.0
 int vlan 4
  no sh
  ip address 192.168.4.200 255.255.255.0
```

**MLS2**
```
en
conf t
 ip routing

 int vlan 1
  no sh
  ip address 192.168.1.201 255.255.255.0
 int vlan 2
  no sh
  ip address 192.168.2.201 255.255.255.0
 int vlan 3
  no sh
  ip address 192.168.3.201 255.255.255.0
 int vlan 4
  no sh
  ip address 192.168.4.201 255.255.255.0
```

## 7️) MLS HSRP

**MLS1**
```
en
conf t
 int vlan 1
  standby ip 192.168.1.250
  standby priority 150
  standby preempt

 int vlan 2
  standby ip 192.168.2.250
  standby priority 150
  standby preempt

 int vlan 3
  standby ip 192.168.3.250
  standby preempt

 int vlan 4
  standby ip 192.168.4.250
  standby preempt
```

**MLS2**
```
en
conf t
 int vlan 1
  standby ip 192.168.1.250
  standby preempt

 int vlan 2
  standby ip 192.168.2.250
  standby preempt

 int vlan 3
  standby ip 192.168.3.250
  standby priority 150
  standby preempt

 int vlan 4
  standby ip 192.168.4.250
  standby priority 150
  standby preempt
```

## 8️) MLS DHCP Config

```
en
conf t
 ip dhcp pool IT
  network 192.168.1.0 255.255.255.0
  default-router 192.168.1.250
  dns-server 8.8.8.8

 ip dhcp pool HR
  network 192.168.2.0 255.255.255.0
  default-router 192.168.2.250
  dns-server 8.8.8.8

 ip dhcp pool Sales
  network 192.168.3.0 255.255.255.0
  default-router 192.168.3.250
  dns-server 8.8.8.8

 ip dhcp pool MRK
  network 192.168.4.0 255.255.255.0
  default-router 192.168.4.250
  dns-server 8.8.8.8
```

## 9️) Routed Ports for MLS

**MLS1**
```
en
conf t
 int G0/1
  no switchport
  ip address 192.168.10.1 255.255.255.0
  no sh

 int G0/2
  no switchport
  ip address 192.168.11.1 255.255.255.0
  no sh
```

**MLS2**
```
en
conf t
 int G0/1
  no switchport
  ip address 192.168.12.1 255.255.255.0
  no sh

 int G0/2
  no switchport
  ip address 192.168.13.1 255.255.255.0
  no sh
```

## Edge Routers Interfaces IPs

**Edge 1**
```
en
conf t
 int f0/0
  ip address 192.168.10.2 255.255.255.0
  no sh

 int f0/1
  ip address 192.168.13.2 255.255.255.0
  no sh

 int f1/0
  ip address 90.0.0.1 255.0.0.0
  no sh
```

**Edge 2**
```
en
conf t
 int f0/0
  ip address 192.168.12.2 255.255.255.0
  no sh

 int f0/1
  ip address 192.168.11.2 255.255.255.0
  no sh

 int f1/0
  ip address 100.0.0.1 255.0.0.0
  no sh
```

## 1️1) Default Route for Edge Routers

**Edge 1**
```
en
conf t
 ip route 0.0.0.0 0.0.0.0 f1/0
```

**Edge 2**
```
en
conf t
 ip route 0.0.0.0 0.0.0.0 f1/0
```

## 1️2) Routing Protocol (OSPF)

**Edge Routers**
```
en
conf t
 router ospf 100
  network 0.0.0.0 255.255.255.255 area 0
  passive-interface f1/0
  default-information originate
```

**MLS (Core)**
```
en
conf t
 router ospf 100
  network 0.0.0.0 255.255.255.255 area 0
```

---

