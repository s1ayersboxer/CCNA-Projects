# 🌐 CCNA Enterprise Network Lab — Cisco Packet Tracer

A hands-on enterprise network simulation built in Cisco Packet Tracer, demonstrating core CCNA concepts including VLANs, Inter-VLAN Routing, STP, Port Security, DHCP/DNS, and Dynamic Routing Protocols.

---

## 📋 Project Overview

This project simulates a multi-router enterprise topology with segmented networks, dynamic routing, and Layer 2 security. It was designed to reinforce CCNA exam objectives and real-world network engineering skills.

---

## 🖧 Network Topology

```
                        [R3]
                          |
              [R1] ---- [R2]
                |          |
              [SW1]      [SW2]
              /   \      /   \    \
           [PC1] [PC4] [PC2] [PC3] [SRV1]
```

### Devices Used

| Device | Model      | Label              | Qty |
|--------|------------|--------------------|-----|
| Router | Cisco 2911 | R1, R2, R3         | 3   |
| Switch | Cisco 2960 | SW1, SW2           | 2   |
| PC     | Generic PC | PC1, PC2, PC3, PC4 | 4   |
| Server | Server-PT  | SRV1               | 1   |

---

## 📐 IP Addressing Scheme

### Subnets

| Subnet   | Network            | Mask              | Purpose               |
|----------|--------------------|-------------------|-----------------------|
| Subnet 1 | 192.168.1.0/24     | 255.255.255.0     | SW1 side — VLAN 10    |
| Subnet 2 | 192.168.2.0/24     | 255.255.255.0     | SW2 side — VLAN 20    |
| Subnet 3 | 192.168.12.0/30    | 255.255.255.252   | R1 to R2 link         |
| Subnet 4 | 192.168.23.0/30    | 255.255.255.252   | R2 to R3 link         |

### Device IP Assignments

| Device | Interface | IP Address            | Default Gateway |
|--------|-----------|-----------------------|-----------------|
| R1     | Gi0/0.10  | 192.168.1.1           | — (Router)      |
| R1     | Gi0/1     | 192.168.12.1          | — (Router)      |
| R2     | Gi0/0.20  | 192.168.2.1           | — (Router)      |
| R2     | Gi0/1     | 192.168.12.2          | — (Router)      |
| R2     | Gi0/2     | 192.168.23.1          | — (Router)      |
| R3     | Gi0/0     | 192.168.23.2          | — (Router)      |
| PC1    | NIC       | 192.168.1.10          | 192.168.1.1     |
| PC4    | NIC       | 192.168.1.11          | 192.168.1.1     |
| PC2    | NIC       | DHCP (192.168.2.50+)  | 192.168.2.1     |
| PC3    | NIC       | 192.168.2.11          | 192.168.2.1     |
| SRV1   | NIC       | 192.168.2.20          | 192.168.2.1     |

---

## 🔧 Configurations

### Phase 1 — Physical Topology

- Placed 3 routers, 2 switches, 4 PCs, and 1 server
- Connected devices using copper straight-through and crossover cables
- Verified physical connectivity across all links

---

### Phase 2 — IP Addressing

- Planned subnetting using /24 for LAN segments and /30 for point-to-point router links
- Manually assigned static IPs to all PCs and SRV1
- Configured and verified all router interfaces using `show ip interface brief`

---

### Phase 3 — VLANs & 802.1Q Trunking

#### VLAN Plan

| VLAN | Name  | Devices                  |
|------|-------|--------------------------|
| 10   | SALES | PC1, PC4 (SW1)           |
| 20   | IT    | PC2, PC3, SRV1 (SW2)     |

#### SW1 Configuration

```
vlan 10
 name SALES
vlan 20
 name IT

interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10

interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 10

interface fastEthernet 0/3
 switchport mode trunk
```

#### SW2 Configuration

```
vlan 10
 name SALES
vlan 20
 name IT

interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 20

interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20

interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 20

interface fastEthernet 0/3
 switchport mode trunk
```

#### Verification Commands

```
show vlan brief
show interfaces trunk
show cdp neighbors
```

---

### Phase 4 — Inter-VLAN Routing (Router-on-a-Stick)

#### R1 Subinterface Configuration

```
interface gigabitEthernet 0/0
 no ip address

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.0

interface gigabitEthernet 0/1
 ip address 192.168.12.1 255.255.255.252
 no shutdown
```

#### R2 Subinterface Configuration

```
interface gigabitEthernet 0/0
 no ip address

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.2.1 255.255.255.0

interface gigabitEthernet 0/1
 ip address 192.168.12.2 255.255.255.252
 no shutdown

interface gigabitEthernet 0/2
 ip address 192.168.23.1 255.255.255.252
 no shutdown
```

#### Verification

```
show ip interface brief
show interfaces gigabitEthernet 0/0.10
```

---

### Phase 5 — Spanning Tree Protocol (STP)

- STP runs automatically on all Cisco 2960 switches using IEEE 802.1D
- SW1 elected as Root Bridge for VLAN 1 and VLAN 10 based on lowest MAC address
- SW2 elected as Root Bridge for VLAN 20
- Each VLAN runs its own STP instance

#### Verification

```
show spanning-tree
```

#### Key STP Output Fields

| Field                  | Meaning                                        |
|------------------------|------------------------------------------------|
| Root ID                | MAC address of the elected Root Bridge         |
| This bridge is the root| Confirms this switch is the Root Bridge        |
| Desg FWD               | Designated port — actively forwarding traffic  |
| 802.1Q                 | Trunking encapsulation standard confirmed      |

---

### Phase 6 — Port Security

Configured sticky MAC port security on SW1 Fa0/2 (PC1's port):

```
interface fastEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```

#### Verification

```
show port-security interface fastEthernet 0/2
show mac address-table
```

#### Result

| Field               | Value                          |
|---------------------|--------------------------------|
| Port Status         | Secure-up                      |
| Sticky MAC          | 1 (PC1's MAC locked in)        |
| Maximum MACs        | 1                              |
| Violation Mode      | Shutdown                       |
| Security Violations | 0                              |

---

### Phase 7 — DHCP & DNS

#### DHCP — Configured on SRV1

| Field           | Value           |
|-----------------|-----------------|
| Pool Name       | VLAN20-POOL     |
| Default Gateway | 192.168.2.1     |
| DNS Server      | 192.168.2.20    |
| Start IP        | 192.168.2.50    |
| Subnet Mask     | 255.255.255.0   |
| Max Users       | 50              |

PC2 successfully received `192.168.2.50` via DHCP.

#### DNS — Configured on SRV1

| Name         | Type     | Resolved IP  |
|--------------|----------|--------------|
| server.local | A Record | 192.168.2.20 |

Successfully resolved `ping server.local` to `192.168.2.20` from both PC1 and PC2.

---

### Phase 8 — Dynamic Routing Protocols

Configured and tested all three dynamic routing protocols in sequence:

#### RIP v2

```
router rip
 version 2
 network 192.168.1.0
 network 192.168.12.0
 no auto-summary
```

Route code: `R` | Admin Distance: `120`

#### OSPF

```
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.3 area 0
```

Route code: `O` | Admin Distance: `110`

#### EIGRP (Final — Currently Active)

```
router eigrp 100
 network 192.168.1.0
 network 192.168.12.0
 no auto-summary
```

Route code: `D` | Admin Distance: `90`

#### Protocol Comparison

| Protocol | Code | Admin Distance | Type            |
|----------|------|----------------|-----------------|
| RIP v2   | R    | 120            | Distance Vector |
| OSPF     | O    | 110            | Link State      |
| EIGRP    | D    | 90             | Hybrid (Cisco)  |

#### Verification

```
show ip route
show ip protocols
```

---

### Phase 9 — Troubleshooting

#### Commands Used & Results

| Command                    | Result                              |
|----------------------------|-------------------------------------|
| `ping 192.168.2.50`        | 4/4 replies, TTL=126                |
| `ping server.local`        | Resolved to 192.168.2.20            |
| `tracert 192.168.2.50`     | 3 hops: R1 > R2 > PC2              |
| `show ip route`            | EIGRP D routes confirmed            |
| `show ip protocols`        | EIGRP AS 100 active, distance 90    |
| `show interfaces Gi0/1`    | 0 errors, full-duplex confirmed     |

#### Traceroute Output (PC1 to PC2)

```
1   0ms   192.168.1.1    (R1)
2   0ms   192.168.12.2   (R2)
3   1ms   192.168.2.50   (PC2)
```

#### OSI Layer Fault Isolation

| Issue                        | OSI Layer | Fix Applied                                   |
|------------------------------|-----------|-----------------------------------------------|
| Trunk port on wrong interface| Layer 2   | Used show cdp neighbors to find correct port  |
| IP overlap on subinterfaces  | Layer 3   | Removed IP from physical Gi0/0 interface      |
| DNS not resolving on PC1     | Layer 7   | Manually added DNS server IP to PC1           |
| Static route not appearing   | Layer 3   | Removed conflicting connected route first     |

---

## 📊 Key Concepts Demonstrated

| Concept            | Standard / Protocol    | Status |
|--------------------|------------------------|--------|
| VLAN Segmentation  | IEEE 802.1Q            | Done   |
| Trunk Links        | 802.1Q                 | Done   |
| Inter-VLAN Routing | Router-on-a-Stick      | Done   |
| Spanning Tree      | IEEE 802.1D            | Done   |
| Port Security      | Sticky MAC / Shutdown  | Done   |
| DHCP               | RFC 2131               | Done   |
| DNS                | RFC 1035               | Done   |
| Dynamic Routing    | RIP v2, OSPF, EIGRP    | Done   |
| Troubleshooting    | ping, traceroute, show | Done   |

---

## 🛠️ Tools Used

- **Cisco Packet Tracer** — Network simulation
- **Cisco IOS CLI** — Router and switch configuration
- **show commands** — Verification and troubleshooting

---

## 📁 Repository Structure

```
CCNA-Enterprise-Lab/
│
├── README.md
├── CCNA_Enterprise_Lab.pkt      <- Packet Tracer file
└── configs/
    ├── R1-config.txt
    ├── R2-config.txt
    ├── R3-config.txt
    ├── SW1-config.txt
    └── SW2-config.txt
```

---

## 👤 Author

Built as part of CCNA study and hands-on lab practice.
