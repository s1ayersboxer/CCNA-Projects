🌐 CCNA Enterprise Network Lab — Cisco Packet Tracer

A hands-on enterprise network simulation built in Cisco Packet Tracer, demonstrating core CCNA concepts including VLANs, Inter-VLAN Routing, STP, Port Security, DHCP/DNS, and Dynamic Routing Protocols.


📋 Project Overview

This project simulates a multi-router enterprise topology with segmented networks, dynamic routing, and Layer 2 security. It was designed to reinforce CCNA exam objectives and real-world network engineering skills.


🖧 Network Topology

                        [R3]
                          |
              [R1] ---- [R2]
                |          |
              [SW1]      [SW2]
              /   \      /   \    \
           [PC1] [PC4] [PC2] [PC3] [SRV1]

Devices Used

DeviceModelQuantityRouterCisco 29113 (R1, R2, R3)SwitchCisco 29602 (SW1, SW2)PCGeneric PC4 (PC1, PC2, PC3, PC4)ServerServer-PT1 (SRV1)


📐 IP Addressing Scheme

Subnets

SubnetNetworkMaskPurposeSubnet 1192.168.1.0/24255.255.255.0SW1 side — VLAN 10Subnet 2192.168.2.0/24255.255.255.0SW2 side — VLAN 20Subnet 3192.168.12.0/30255.255.255.252R1 ↔ R2 linkSubnet 4192.168.23.0/30255.255.255.252R2 ↔ R3 link

Device IP Assignments

DeviceInterfaceIP AddressGatewayR1Gi0/0.10192.168.1.1—R1Gi0/1192.168.12.1—R2Gi0/0.20192.168.2.1—R2Gi0/1192.168.12.2—R2Gi0/2192.168.23.1—R3Gi0/0192.168.23.2—PC1NIC192.168.1.10192.168.1.1PC4NIC192.168.1.11192.168.1.1PC2NICDHCP (192.168.2.50+)192.168.2.1PC3NIC192.168.2.11192.168.2.1SRV1NIC192.168.2.20192.168.2.1


🔧 Configurations

Phase 1 — Physical Topology


Placed 3 routers, 2 switches, 4 PCs, and 1 server
Connected devices using copper straight-through and crossover cables
Verified physical connectivity across all links



Phase 2 — IP Addressing


Planned subnetting using /24 for LAN segments and /30 for point-to-point router links
Manually assigned static IPs to all PCs and SRV1
Configured and verified all router interfaces using show ip interface brief



Phase 3 — VLANs & 802.1Q Trunking

VLAN Plan

VLANNameDevices10SALESPC1, PC4 (SW1)20ITPC2, PC3, SRV1 (SW2)

SW1 Configuration

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

SW2 Configuration

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

Verification Commands

show vlan brief
show interfaces trunk
show cdp neighbors


Phase 4 — Inter-VLAN Routing (Router-on-a-Stick)

R1 Subinterface Configuration

interface gigabitEthernet 0/0
 no ip address

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.0

interface gigabitEthernet 0/1
 ip address 192.168.12.1 255.255.255.252
 no shutdown

R2 Subinterface Configuration

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

Verification

show ip interface brief
show interfaces gigabitEthernet 0/0.10


Phase 5 — Spanning Tree Protocol (STP)


STP runs automatically on all Cisco 2960 switches using IEEE 802.1D
SW1 elected as Root Bridge for VLAN 1 and VLAN 10 based on lowest MAC address
SW2 elected as Root Bridge for VLAN 20
Each VLAN runs its own STP instance


Verification

show spanning-tree

Key STP Output Fields

FieldMeaningRoot IDMAC of the Root BridgeThis bridge is the rootConfirms this switch is the Root BridgeDesg FWDDesignated port — actively forwarding802.1QEncapsulation standard confirmed


Phase 6 — Port Security

Configured sticky MAC port security on SW1 Fa0/2 (PC1's port):

interface fastEthernet 0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

Verification

show port-security interface fastEthernet 0/2
show mac address-table

Result

FieldValuePort StatusSecure-upSticky MAC1 (PC1's MAC locked)Violation ModeShutdown


Phase 7 — DHCP & DNS

DHCP — Configured on SRV1

FieldValuePool NameVLAN20-POOLDefault Gateway192.168.2.1DNS Server192.168.2.20Start IP192.168.2.50Subnet Mask255.255.255.0Max Users50

PC2 successfully received 192.168.2.50 via DHCP.

DNS — Configured on SRV1

NameTypeAddressserver.localA Record192.168.2.20

Successfully resolved ping server.local → 192.168.2.20 from PC1 and PC2.


Phase 8 — Dynamic Routing Protocols

Configured and tested all three dynamic routing protocols in sequence:

RIP v2

router rip
 version 2
 network 192.168.1.0
 network 192.168.12.0
 no auto-summary

Route code: R | Admin Distance: 120

OSPF

router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.3 area 0

Route code: O | Admin Distance: 110

EIGRP (Final — Currently Active)

router eigrp 100
 network 192.168.1.0
 network 192.168.12.0
 no auto-summary

Route code: D | Admin Distance: 90

Protocol Comparison

ProtocolCodeAdmin DistanceTypeRIP v2R120Distance VectorOSPFO110Link StateEIGRPD90Hybrid (Cisco)

Verification

show ip route
show ip protocols


Phase 9 — Troubleshooting

Commands Used & Results

CommandResultping 192.168.2.50✅ 4/4 replies, TTL=126ping server.local✅ Resolved to 192.168.2.20tracert 192.168.2.50✅ 3 hops: R1 → R2 → PC2show ip route✅ EIGRP routes confirmedshow ip protocols✅ EIGRP AS 100 activeshow interfaces Gi0/1✅ 0 errors, full-duplex

Traceroute Output (PC1 → PC2)

1   0ms   192.168.1.1    (R1)
2   0ms   192.168.12.2   (R2)
3   1ms   192.168.2.50   (PC2)

OSI Layer Fault Isolation Examples

IssueLayerFix AppliedTrunk port on wrong interfaceLayer 2Used show cdp neighbors to find correct portIP overlap on subinterfacesLayer 3Removed IP from physical interfaceDNS not resolving on PC1Layer 7Manually added DNS server IP to PC1Static route not appearingLayer 3Removed conflicting connected route first


📊 Key Concepts Demonstrated

ConceptStandard/ProtocolStatusVLAN SegmentationIEEE 802.1Q✅Trunk Links802.1Q✅Inter-VLAN RoutingRouter-on-a-Stick✅Spanning TreeIEEE 802.1D✅Port SecuritySticky MAC✅DHCPRFC 2131✅DNSRFC 1035✅Dynamic RoutingRIP v2, OSPF, EIGRP✅Troubleshootingping, traceroute, show✅


🛠️ Tools Used


Cisco Packet Tracer — Network simulation
Cisco IOS CLI — Router and switch configuration
show commands — Verification and troubleshooting



📁 Repository Structure

CCNA-Enterprise-Lab/
│
├── README.md
├── CCNA_Enterprise_Lab.pkt      ← Packet Tracer file
└── configs/
    ├── R1-config.txt
    ├── R2-config.txt
    ├── R3-config.txt
    ├── SW1-config.txt
    └── SW2-config.txt


👤 Author

Built as part of CCNA study and hands-on lab practice.
ArtifactsReadmeDocument · MD Content(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.nonce='OpkpWWXfp8rVwelXg0bZJA==';d.innerHTML="window.__CF$cv$params={r:'a0e77ee18b4345c2',t:'MTc4MTkyNDEzNw=='};var a=document.createElement('script');a.nonce='OpkpWWXfp8rVwelXg0bZJA==';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();