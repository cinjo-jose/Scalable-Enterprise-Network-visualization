# Enterprise Network Design & Implementation

A complete enterprise networking project demonstrating hierarchical network design, multi-site connectivity, VLAN segmentation, advanced routing, security controls, and wireless access point configuration — all implemented and tested in Cisco Packet Tracer.

## Project Overview

This project implements a realistic enterprise network for a multi-site company with headquarters and multiple branch locations. It demonstrates industry-standard network design practices including hierarchical architecture, scalability, redundancy, and comprehensive security controls.

**Course-Based Learning:** This project is based on comprehensive enterprise networking principles covering design, implementation, and troubleshooting of production-grade networks.

## Technologies Implemented

### Core Networking
- ✓ **Hierarchical Network Design** — Three-tier architecture (Core, Distribution, Access layers)
- ✓ **ISP Connectivity** — Simulated internet service provider connection with public IP ranges
- ✓ **OSPF Dynamic Routing** — Multi-area OSPF for optimal path selection and failover
- ✓ **VLANs (Virtual LANs)** — Department-based segmentation across 6 VLANs
- ✓ **Inter-VLAN Routing** — Using Switch Virtual Interface (SVI) for VLAN-to-VLAN communication
- ✓ **Static IPv4 Addressing** — Manual IP assignment for infrastructure and servers
- ✓ **Dynamic IPv4 Addressing** — DHCP servers for client workstations

### Security Features
- ✓ **Access Control Lists (ACLs)** — Inbound/outbound traffic filtering by department
- ✓ **Port Security** — MAC address limiting and violation policies at access layer
- ✓ **SSH (Secure Shell)** — Encrypted remote administration (SSH v2 only)
- ✓ **NAT Overload (PAT)** — Port Address Translation for internet connectivity
- ✓ **VLAN Isolation** — Logical segmentation with security enforcement
- ✓ **DHCP Security** — Scope restriction by VLAN and device class

### Wireless & Advanced Features
- ✓ **WLAN Configuration** — Access Point (AP) setup with SSID and encryption
- ✓ **Host Configurations** — Complete end-device setup (PCs, printers, servers)
- ✓ **Spanning Tree Protocol (STP)** — Loop prevention and redundancy
- ✓ **Network Address Translation (NAT)** — Inside/outside networks, static and dynamic NAT
- ✓ **DHCP Server** — Centralized and distributed DHCP pools per VLAN

## Network Architecture

### Hierarchical Design (Three Tiers)

```
┌─────────────────────────────────────────────────────────────┐
│                     CORE LAYER (R1)                         │
│              Central Hub Connectivity                       │
│        High-Speed Backbone, OSPF Area 0                    │
│     Connects to ISP, Branches, and Distribution             │
└─────────────────────────────────────────────────────────────┘
          │                                    │
          │                                    │
┌─────────▼──────────┐              ┌─────────▼──────────┐
│ DISTRIBUTION LAYER │              │ DISTRIBUTION LAYER │
│   (Access Switch)  │              │   (Access Switch)  │
│   HQ Main Switch   │              │  Branch Switches   │
│   VLAN Mgmt, SVI   │              │   VLAN Mgmt, SVI   │
└─────────┬──────────┘              └─────────┬──────────┘
          │                                    │
    ┌─────┴──────────────┐           ┌────────┴──────────┐
    │                    │           │                   │
┌───▼───┐  ┌────┐  ┌────▼──┐   ┌────▼───┐   ┌─────┐  ┌──▼───┐
│ACCESS │  │WLAN│  │ACCESS │   │ACCESS  │   │WLAN │  │SERVER│
│LAYER  │  │ AP │  │LAYER  │   │LAYER   │   │ AP  │  │LAYER │
│Devices│  │802.│  │Devices│   │Devices │   │802. │  │DB/WEB│
└───────┘  └────┘  └───────┘   └────────┘   └─────┘  └──────┘
    │       │         │            │         │         │
┌─────────────────────────────────────────────────────────────┐
│              ACCESS LAYER (VLANs 10-60)                     │
│  End Devices: PCs, Printers, IP Phones, Servers, Guests    │
└─────────────────────────────────────────────────────────────┘
```

### Physical Sites

| Site | Location | Role | Router | Devices |
|------|----------|------|--------|---------|
| HQ | Headquarters | Core/Distribution | R1 | 20+ devices, AP, DHCP |
| Branch 1 | Regional Office | Access | R2 | 15+ devices, AP |
| Branch 2 | Regional Office | Access | R3 | 15+ devices, AP |

## VLAN Design & IP Addressing

### Headquarters (HQ)

| VLAN | Name | Subnet | Gateway | Device Count | Purpose |
|------|------|--------|---------|--------------|---------|
| 10 | Management | 192.168.10.0/24 | 192.168.10.1 | 5 | Network admin, IT staff |
| 20 | Finance | 192.168.20.0/24 | 192.168.20.1 | 8 | Finance department, restricted access |
| 30 | HR | 192.168.30.0/24 | 192.168.30.1 | 6 | HR & payroll staff |
| 40 | Guest | 192.168.40.0/24 | 192.168.40.1 | 10 | Guest WiFi, visitors, contractors |
| 50 | Servers | 192.168.50.0/24 | 192.168.50.1 | 3 | Web, Database, Mail servers (static IPs) |

### Branch Locations

| VLAN | Name | Subnet | Gateway | Sites | Purpose |
|------|------|--------|---------|-------|---------|
| 60 | Branch 1 | 192.168.60.0/24 | 192.168.60.1 | Branch 1 | Branch office users |
| 70 | Branch 2 | 192.168.70.0/24 | 192.168.70.1 | Branch 2 | Branch office users |

## Routing Configuration

### OSPF (Open Shortest Path First)

```
Topology: Multiarea OSPF with hierarchical design
- Area 0 (Backbone): R1 (Core)
- Area 1: Branch 1 (R2)
- Area 2: Branch 2 (R3)

Advantages:
✓ Dynamic routing (automatic path adaptation)
✓ Faster convergence than RIP
✓ Scalable to large networks
✓ Loop-free topology via SPF algorithm
✓ Supports multiple paths (load balancing)

Cost Calculation:
- LAN interfaces: Cost 1
- Serial/WAN: Cost 10-100 (based on bandwidth)
- OSPF selects lowest total cost path
```

### Inter-VLAN Routing (SVI Method)

```
Configuration on Distribution Switch (L3 Switch):

interface Vlan10
  ip address 192.168.10.1 255.255.255.0
  no shutdown

interface Vlan20
  ip address 192.168.20.1 255.255.255.0
  no shutdown

(... Vlan 30, 40, 50, 60, 70 ...)

Benefits of SVI vs Router-on-a-Stick:
✓ Better performance (wire-speed routing)
✓ Reduced latency (no serial interface bottleneck)
✓ More scalable (supports more VLANs)
✓ Industry standard in modern networks
```

## DHCP Configuration

### DHCP Server Setup (on R1)

```
DHCP pools per VLAN with scope isolation:

Pool: VLAN10_MGMT
  Network: 192.168.10.0 255.255.255.0
  Gateway: 192.168.10.1
  DNS: 8.8.8.8, 8.8.4.4
  Excluded: 192.168.10.1-192.168.10.20 (infrastructure)
  Lease: 1 day

Pool: VLAN20_FINANCE
  Network: 192.168.20.0 255.255.255.0
  Gateway: 192.168.20.1
  DNS: 8.8.8.8, 8.8.4.4
  Excluded: 192.168.20.1-192.168.20.20
  Lease: 1 day

(... Similar for VLAN 30, 40, 60, 70 ...)

Servers use static IPv4 addressing:
- Web Server: 192.168.50.10
- Database: 192.168.50.20
- Mail: 192.168.50.30
```

## Wireless Network (WLAN)

### Access Point Configuration

```
Access Points: 1 per site (HQ, Branch 1, Branch 2)

Settings:
├── SSID
│   ├── HQ-Enterprise (HQ)
│   ├── Branch1-Wireless (Branch 1)
│   └── Branch2-Wireless (Branch 2)
│
├── Security
│   ├── WPA2-PSK (Pre-Shared Key)
│   ├── Passphrase: [Strong password]
│   └── Encryption: AES
│
├── VLAN Mapping
│   ├── Guest SSID → VLAN 40
│   ├── Employee SSID → VLAN 10-30 (based on user role)
│   └── Management SSID → VLAN 10 (admin only)
│
└── IP Address (Static)
    ├── HQ AP: 192.168.10.50
    ├── Branch1 AP: 192.168.60.50
    └── Branch2 AP: 192.168.70.50

Clients connect to appropriate VLAN based on SSID:
- Employee connects to "HQ-Enterprise" → VLAN 10-30 based on credentials
- Guest connects to "Guest-WiFi" → VLAN 40 (isolated)
```

## Security Implementation

### Port Security

```
Access Layer Switches: Limit MAC addresses per port

Configuration:
interface FastEthernet0/2
  switchport mode access
  switchport access vlan 10
  switchport port-security
  switchport port-security maximum 1
  switchport port-security mac-address sticky
  switchport port-security violation shutdown

Prevents:
✓ Unauthorized device connection
✓ MAC address spoofing
✓ Rogue device injection
✓ "Port hop" attacks

Action on violation: Port disabled (shutdown), requires manual reenable
```

### SSH (Secure Shell)

```
All routers configured for secure remote access:

ip domain-name company.local
crypto key generate rsa modulus 1024
ip ssh version 2
ip ssh time-out 120
ip ssh authentication-retries 3

line vty 0 4
  transport input ssh
  login local
  exit

username admin privilege 15 secret [password]

Benefits:
✓ Encryption of all traffic (RSA-based)
✓ Authentication required (no guest access)
✓ No Telnet (plain-text) allowed
✓ Compliance with security standards
```

### Access Control Lists (ACLs)

```
Department-Based Access Control:

1. Finance VLAN Restrictions
   - Can access Finance resources (same VLAN)
   - Can access IT support (VLAN 10)
   - BLOCKED from HR payroll (VLAN 30)
   - BLOCKED from Guest network (VLAN 40)

2. Guest VLAN Isolation
   - Can access internet (via NAT)
   - BLOCKED from all internal VLANs
   - BLOCKED from servers
   - No access to management interfaces

3. Server Access Control
   - Web Server: HTTP/HTTPS from all VLANs
   - Database: SQL from Finance/Admin only
   - Mail: SMTP from Finance/HR, POP3 from all

Implementation:
- Standard ACL: Permit/deny by source IP
- Extended ACL: Permit/deny by source, destination, protocol, port
- Named ACL: Better readability (access-list descriptions)
```

### NAT Overload (Port Address Translation - PAT)

```
Configuration for internet connectivity:

ip nat inside source list 1 interface Serial0/0 overload

ip access-list standard NAT_INSIDE
  permit 192.168.0.0 0.0.255.255

interface FastEthernet0/0 (internal)
  ip nat inside

interface Serial0/0 (to ISP)
  ip nat outside

How it works:
- Internal hosts use private IPs (192.168.x.x)
- Router translates to single public IP
- Different ports map to different internal hosts
  Example: 
    - PC1:1025 → PublicIP:1001 → Server:80
    - PC2:1026 → PublicIP:1002 → Server:80

Benefits:
✓ Conserves public IP addresses
✓ Hides internal network topology
✓ Acts as implicit firewall
✓ Allows internet access for all internal users
```

## Device Configurations

### End Devices (Host Configuration)

```
Workstations:
├── IP Assignment Method
│   ├── DHCP (dynamic) for most users
│   └── Static IP for servers
├── Gateway Configuration
│   └── Points to VLAN gateway (SVI on switch)
├── DNS Configuration
│   ├── Primary: 8.8.8.8 (Google)
│   └── Secondary: 8.8.4.4
└── Optional
    ├── Hostname (e.g., "Finance-PC-01")
    └── Domain name (company.local)

Servers (Static IPv4):
├── Web Server
│   ├── IP: 192.168.50.10
│   ├── Services: HTTP (80), HTTPS (443)
│   └── Accessible from all VLANs
├── Database Server
│   ├── IP: 192.168.50.20
│   ├── Services: SQL (1433)
│   └── Restricted to Finance/Admin
└── Mail Server
    ├── IP: 192.168.50.30
    ├── Services: SMTP (25), POP3 (110)
    └── Restricted by department

Wireless Clients:
├── Connect to AP SSID
├── Obtain IP via DHCP
├── Assigned to appropriate VLAN
└── Subject to VLAN-based ACLs
```

## Network Testing & Verification

### Connectivity Tests Performed

```
✓ Ping tests within VLAN (0% loss)
✓ Ping tests across VLANs (0% loss, via SVI routing)
✓ Ping tests between sites (0% loss, via OSPF)
✓ DHCP lease assignment (clients receive IPs)
✓ Wireless connectivity (AP authentication successful)
✓ SSH access to routers (secure terminal access)
✓ NAT translation (internal hosts reach internet)
✓ ACL enforcement (blocked traffic as configured)
✓ Port security (rogue devices rejected)
✓ Server accessibility (from authorized VLANs only)
```

### Simulation Results

| Component | Status | Details |
|-----------|--------|---------|
| OSPF Neighbors | ✓ Established | R1-R2, R1-R3 full adjacency |
| VLAN Routing | ✓ Functional | All inter-VLAN pings successful |
| DHCP Assignment | ✓ Working | Clients receiving IPs from pools |
| Wireless | ✓ Connected | 3 APs, clients authenticated |
| Security Controls | ✓ Enforced | ACLs blocking as configured |
| NAT/PAT | ✓ Translating | Internal hosts reaching internet |
| SSH Access | ✓ Secured | Encrypted remote access active |

## Skills Demonstrated

### Network Design
- ✓ Hierarchical network architecture (Core, Distribution, Access)
- ✓ Scalable design supporting multiple sites
- ✓ Redundancy planning with OSPF failover
- ✓ VLAN design for departmental segmentation
- ✓ IP addressing scheme (subnetting, NAT)

### Routing & Switching
- ✓ OSPF multi-area configuration
- ✓ Inter-VLAN routing via SVI
- ✓ Static and dynamic routing decisions
- ✓ VLAN creation and trunk configuration
- ✓ Spanning Tree Protocol for loop prevention

### Security
- ✓ Access Control Lists (standard and extended)
- ✓ Port security implementation
- ✓ SSH encryption and authentication
- ✓ VLAN-based isolation
- ✓ NAT for network privacy
- ✓ DHCP scope management
- ✓ Wireless security (WPA2-PSK)

### Advanced Topics
- ✓ DHCP server configuration
- ✓ Wireless access point setup
- ✓ Static IPv4 addressing for servers
- ✓ Host configuration (PCs, printers, devices)
- ✓ Network Address Translation (NAT/PAT)
- ✓ End-to-end network testing and troubleshooting

## Project Specifications

| Aspect | Value |
|--------|-------|
| Number of Sites | 3 (HQ + 2 Branches) |
| Total VLANs | 7 |
| Total Devices | 60+ |
| Routers | 3 (OSPF enabled) |
| Switches | 3+ (SVI routing capable) |
| Access Points | 3 (WLAN coverage) |
| Servers | 3 (Web, Database, Mail) |
| DHCP Pools | 7 (one per VLAN) |
| Security Policies | 5+ ACLs |
| Simulation Tool | Cisco Packet Tracer 8.x+ |

## Reference Documentation

- [Cisco OSPF Configuration Guide](https://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/)
- [VLAN and Inter-VLAN Routing](https://www.cisco.com/c/en/us/support/docs/lan-switching/vlan/)
- [Switch Virtual Interface (SVI)](https://www.cisco.com/c/en/us/support/docs/switches/catalyst-6500-series/)
- [DHCP Configuration](https://www.cisco.com/c/en/us/support/docs/ip/dynamic-host-configuration-protocol-dhcp/)
- [SSH Security Configuration](https://www.cisco.com/c/en/us/support/docs/security/secure-shell-ssh/)
- [Access Control Lists](https://www.cisco.com/c/en/us/support/docs/security/access-lists/)
- [Port Security](https://www.cisco.com/c/en/us/support/docs/lan-switching/port-security/)
- [Wireless LAN Configuration](https://www.cisco.com/c/en/us/support/docs/wireless/)
- [NAT Configuration](https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/)

## Course Attribution

This project is based on comprehensive enterprise networking training covering:
- Hierarchical network design principles
- ISP connectivity and internet routing
- Multi-site enterprise network implementation
- Advanced VLAN and routing technologies
- Security controls and access management
- Wireless network deployment
- End-to-end network configuration and testing

**Video Reference:** [Cisco Packet Tracer Enterprise Network Course](https://www.youtube.com/watch?v=eqEd84yeRxg&list=PLvUOx2WG6R7PMM8UhMWevH75QzGyXOv4g&index=8)

## License

This project documentation is released under the MIT License.

## Contributing

This is a portfolio project demonstrating enterprise networking knowledge. For questions or suggestions:
- Open an issue for clarifications
- Submit a pull request with improvements
- Suggest additional security controls or optimizations

---

**Built with:** Cisco Packet Tracer, enterprise networking best practices, production-grade design principles.
