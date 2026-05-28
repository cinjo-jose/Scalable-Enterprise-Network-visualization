# Advanced Networking Technologies & Implementation

Comprehensive guide to advanced networking features implemented in the enterprise network project.

---

## Table of Contents

1. [Hierarchical Network Design](#hierarchical-network-design)
2. [VLAN & Inter-VLAN Routing (SVI)](#vlan--inter-vlan-routing-svi)
3. [DHCP Server Configuration](#dhcp-server-configuration)
4. [Port Security](#port-security)
5. [SSH Configuration](#ssh-configuration)
6. [NAT Overload (PAT)](#nat-overload-pat)
7. [Access Control Lists (ACLs)](#access-control-lists-acls)
8. [Wireless LAN (WLAN) Setup](#wireless-lan-wlan-setup)
9. [Static IPv4 Addressing](#static-ipv4-addressing)
10. [Host Configurations](#host-configurations)

---

## Hierarchical Network Design

### Three-Tier Architecture

```
TIER 1: CORE LAYER
├── Purpose: High-speed backbone connectivity
├── Components: Core routers, backbone links
├── Traffic: Backbone (inter-site) traffic
├── Devices: R1 (HQ Core)
└── Characteristics: Highest availability, redundancy focus

TIER 2: DISTRIBUTION LAYER
├── Purpose: Route traffic between access and core
├── Components: Distribution switches, VLANs, SVI
├── Traffic: Aggregated VLAN traffic
├── Devices: Access switches with L3 capabilities
└── Characteristics: Policy enforcement, VLAN routing

TIER 3: ACCESS LAYER
├── Purpose: Connect end devices to network
├── Components: Access switches, access points, ports
├── Traffic: Individual device traffic
├── Devices: PCs, printers, IP phones, wireless clients
└── Characteristics: Port security, QoS, edge policies
```

### Design Benefits

```
Scalability:
✓ Easy to add new sites or VLANs
✓ Modular design supports growth
✓ Each tier handles specific function

Performance:
✓ Reduced congestion through hierarchy
✓ Optimized paths via OSPF
✓ Lower latency through efficient routing

Redundancy:
✓ Multiple paths (OSPF convergence)
✓ STP prevents loops
✓ Failover automatic with dynamic routing

Manageability:
✓ Clear separation of concerns
✓ Policy enforcement at distribution
✓ Consistent configuration across sites
```

---

## VLAN & Inter-VLAN Routing (SVI)

### VLAN Creation

```
Switch Configuration:

vlan 10
  name Management
  description IT and network administrators

vlan 20
  name Finance
  description Finance department - restricted access

vlan 30
  name HR
  description Human Resources staff

vlan 40
  name Guest
  description Guest WiFi and visitor network

vlan 50
  name Servers
  description Database and web servers

vlan 60
  name Branch1
  description Branch office 1 users

vlan 70
  name Branch2
  description Branch office 2 users
```

### Trunk Configuration

```
Port connecting switch to router/distribution switch:

interface GigabitEthernet0/1
  description Trunk to Core Router
  switchport mode trunk
  switchport trunk encapsulation dot1Q
  switchport trunk allowed vlan 10,20,30,40,50,60,70
  switchport trunk native vlan 1
  spanning-tree portfast disable
  no shutdown

Benefits of 802.1Q Trunking:
✓ Carries multiple VLANs on single link
✓ Reduces physical cables needed
✓ Improves scalability
✓ Standard method across vendors
```

### Switch Virtual Interface (SVI) - Inter-VLAN Routing

```
L3 Switch (Distribution Layer) Configuration:

Interface configuration for each VLAN:

interface Vlan10
  description Management VLAN Gateway
  ip address 192.168.10.1 255.255.255.0
  no shutdown

interface Vlan20
  description Finance VLAN Gateway
  ip address 192.168.20.1 255.255.255.0
  no shutdown

interface Vlan30
  description HR VLAN Gateway
  ip address 192.168.30.1 255.255.255.0
  no shutdown

interface Vlan40
  description Guest VLAN Gateway
  ip address 192.168.40.1 255.255.255.0
  no shutdown

interface Vlan50
  description Server VLAN Gateway
  ip address 192.168.50.1 255.255.255.0
  no shutdown

interface Vlan60
  description Branch1 VLAN Gateway
  ip address 192.168.60.1 255.255.255.0
  no shutdown

interface Vlan70
  description Branch2 VLAN Gateway
  ip address 192.168.70.1 255.255.255.0
  no shutdown

Enable routing:
ip routing

OSPF for inter-site communication:
router ospf 1
  network 192.168.0.0 0.0.255.255 area 0
  router-id 1.1.1.1
```

### How SVI Routing Works

```
Communication Flow:

Host in VLAN 10 (192.168.10.100) → Host in VLAN 20 (192.168.20.100)

Step 1: Source host sends ARP for its gateway
  - Finds 192.168.10.1 (VLAN 10 SVI)

Step 2: Switch performs routing decision
  - Destination IP 192.168.20.100 not in VLAN 10
  - Consults routing table
  - Routes through VLAN 20 SVI (192.168.20.1)

Step 3: Switch forwards frame to VLAN 20
  - Changes source MAC to switch MAC
  - Changes destination MAC to destination host MAC

Step 4: Destination host receives frame
  - Replies with return path through VLAN 10

Result: Inter-VLAN communication established
```

### SVI vs Router-on-a-Stick Comparison

| Aspect | SVI (L3 Switch) | Router-on-a-Stick |
|--------|-----------------|-------------------|
| **Speed** | Wire-speed routing | Limited by serial link |
| **Latency** | Lower (<1ms) | Higher (2-5ms) |
| **Scalability** | Supports many VLANs | Limited by router ports |
| **Cost** | Higher initial | Lower initial |
| **Throughput** | Gbps-level | Mbps-level |
| **Complexity** | Moderate | Simple |
| **Modern Use** | Industry standard | Legacy/small networks |

---

## DHCP Server Configuration

### DHCP Pool Setup

```
Router R1 (DHCP Server):

ip dhcp pool VLAN10_MANAGEMENT
  network 192.168.10.0 255.255.255.0
  default-router 192.168.10.1
  dns-server 8.8.8.8 8.8.4.4
  domain-name company.local
  lease 1

ip dhcp pool VLAN20_FINANCE
  network 192.168.20.0 255.255.255.0
  default-router 192.168.20.1
  dns-server 8.8.8.8 8.8.4.4
  domain-name company.local
  lease 1

ip dhcp pool VLAN30_HR
  network 192.168.30.0 255.255.255.0
  default-router 192.168.30.1
  dns-server 8.8.8.8 8.8.4.4
  domain-name company.local
  lease 1

ip dhcp pool VLAN40_GUEST
  network 192.168.40.0 255.255.255.0
  default-router 192.168.40.1
  dns-server 8.8.8.8 8.8.4.4
  domain-name company.local
  lease 1

(Similar for VLAN 50, 60, 70...)

Exclude addresses (reserved for infrastructure):
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.20.1 192.168.20.20
ip dhcp excluded-address 192.168.30.1 192.168.30.20
ip dhcp excluded-address 192.168.40.1 192.168.40.20
ip dhcp excluded-address 192.168.50.1 192.168.50.20
ip dhcp excluded-address 192.168.60.1 192.168.60.20
ip dhcp excluded-address 192.168.70.1 192.168.70.20
```

### DHCP Lease Process

```
DHCP 4-Way Handshake:

1. DISCOVER (Client → Server)
   - Client broadcasts: "Any DHCP servers?"
   - Destination: 255.255.255.255

2. OFFER (Server → Client)
   - Server responds: "Here's an IP: 192.168.10.100"
   - Lease time: 86400 seconds (1 day)

3. REQUEST (Client → Server)
   - Client broadcasts: "I accept 192.168.10.100"
   - Reserves address across network

4. ACK (Server → Client)
   - Server confirms: "Lease granted"
   - Client can now use IP address

Renewal:
- At 50% lease expiration: Client renews with server
- At 87.5% lease expiration: Client broadcasts renewal request
- At 100% expiration: Lease lost, must restart DISCOVER
```

### Verification

```
show ip dhcp pool
show ip dhcp binding
show ip dhcp server statistics
```

---

## Port Security

### Configuration

```
Access Layer Switch:

interface FastEthernet0/2
  switchport mode access
  switchport access vlan 10
  description Management VLAN - Port Security Enabled
  
  ! Port security configuration
  switchport port-security
  switchport port-security maximum 1
  switchport port-security mac-address sticky
  switchport port-security violation shutdown
  
  no shutdown
```

### Port Security Modes

```
1. SHUTDOWN (Default and Recommended)
   - Violating port immediately disabled
   - No traffic allowed
   - Manual intervention required
   - Security: Highest

2. RESTRICT
   - Port remains enabled
   - Violating packets dropped
   - Sends alert
   - Monitoring enabled

3. PROTECT
   - Port remains enabled
   - Violating packets dropped
   - No alert generated
   - Silent blocking

Configuration examples:
  switchport port-security violation shutdown
  switchport port-security violation restrict
  switchport port-security violation protect
```

### Sticky MAC Address

```
Two methods:

1. Sticky (Automatic Learning)
   switchport port-security mac-address sticky
   
   Result:
   - First device to connect has MAC learned
   - MAC stored in running-config
   - Device must disconnectto change MAC
   - Persists across reboots

2. Static Configuration
   switchport port-security mac-address 0011.2233.4455
   
   Result:
   - Manual entry required
   - Only specified device allowed
   - Cannot change without reconfiguration
   - More restrictive

Best practice: Use sticky for user ports, static for important devices
```

### Benefits

```
✓ Prevents unauthorized device connection
✓ Stops MAC address spoofing
✓ Limits devices per port (usually 1)
✓ Defends against "port hopping" attacks
✓ Compliance with security policies
✓ Easy to implement and manage
```

---

## SSH Configuration

### Router SSH Setup

```
Router Configuration:

! 1. Set domain name (required for RSA key generation)
ip domain-name company.local

! 2. Generate RSA key pair
crypto key generate rsa modulus 1024
  (Answer: yes to export keys)

! 3. Configure SSH version
ip ssh version 2
ip ssh time-out 120
ip ssh authentication-retries 3

! 4. Configure VTY lines
line vty 0 4
  transport input ssh
  login local
  exit

! 5. Create local user
username admin privilege 15 secret MySecurePassword123

! 6. Verify SSH
show ip ssh

Output should show:
SSH version : 2.0
SSH Enabled : true
Authentication timeout: 120 secs
Authentication retries: 3
```

### SSH vs Telnet

```
TELNET (Plain-text - NEVER use)
├── Port: 23
├── Encryption: None
├── Authentication: Plain-text
├── Vulnerability: Packet sniffing
└── Use: Legacy only

SSH (Encrypted - Always use)
├── Port: 22
├── Encryption: RSA/AES-based
├── Authentication: Encrypted
├── Vulnerability: Resistant to sniffing
└── Use: Production standard

Compliance:
- PCI-DSS: Requires SSH only
- HIPAA: Mandates encryption
- SOX: Prohibits Telnet
- ISO 27001: SSH recommended
```

### SSH Session Initiation

```
From management PC:

$ ssh admin@192.168.1.1

Establish connection? yes

R1#

(Now connected securely)
```

---

## NAT Overload (PAT)

### Configuration

```
Router (Connecting to ISP):

! 1. Define inside local network
ip access-list standard NAT_INSIDE
  permit 192.168.0.0 0.0.255.255

! 2. Configure NAT overload (PAT)
ip nat inside source list 1 interface Serial0/0 overload

! 3. Designate inside interface
interface FastEthernet0/0
  ip nat inside

! 4. Designate outside interface
interface Serial0/0
  ip nat outside
```

### How NAT Translation Works

```
Internal Network: 192.168.0.0/16
External Network: ISP provided public IP (e.g., 203.0.113.5)

Example Scenario:
- PC1 at 192.168.10.100 sends request to 8.8.8.8:80
- R1 translates:
  Before NAT: 192.168.10.100:51200 → 8.8.8.8:80
  After NAT:  203.0.113.5:5001 → 8.8.8.8:80

- Web server responds:
  Before NAT: 8.8.8.8:80 → 203.0.113.5:5001
  After NAT:  8.8.8.8:80 → 192.168.10.100:51200

Translation Table (NAT):
Source (Internal)        │ Translated (External)
─────────────────────────┼──────────────────────
192.168.10.100:51200     │ 203.0.113.5:5001
192.168.20.100:51201     │ 203.0.113.5:5002
192.168.30.100:51202     │ 203.0.113.5:5003

Benefits:
✓ Multiple internal hosts share one public IP
✓ Hides internal network topology
✓ Acts as firewall (external can't initiate)
✓ Conserves public IP addresses (major IPv4 benefit)
```

### Verification

```
show ip nat translations
show ip nat statistics
```

---

## Access Control Lists (ACLs)

### ACL Types

```
1. STANDARD ACL (1-99)
   - Filters by source IP only
   - Applied close to destination
   - Simple, limited functionality
   
   access-list 1 permit 192.168.10.0 0.0.0.255
   access-list 1 deny any
   
   interface FastEthernet0/1
     ip access-group 1 in

2. EXTENDED ACL (100-199)
   - Filters by source, destination, protocol, port
   - Applied close to source
   - More granular control
   
   access-list 100 permit tcp 192.168.20.0 0.0.0.255 192.168.50.0 0.0.0.255 eq 1433
   access-list 100 deny tcp 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
   access-list 100 permit ip any any
   
   interface Vlan20
     ip access-group 100 in

3. NAMED ACL (Readable names)
   
   ip access-list extended FINANCE_RESTRICT
     permit tcp 192.168.20.0 0.0.0.255 192.168.20.0 0.0.0.255
     permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
     deny tcp 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
     permit ip any any
   
   interface Vlan20
     ip access-group FINANCE_RESTRICT in
```

### ACL Processing

```
Traffic Flow:

1. Packet arrives at interface
2. ACL evaluation (top to bottom)
3. First match applies (implicit deny all at end)
4. Action taken (permit/deny)

Example:
  access-list 100 permit tcp 192.168.20.0 0.0.0.255 any eq 80
  access-list 100 permit tcp 192.168.20.0 0.0.0.255 any eq 443
  access-list 100 deny tcp 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
  access-list 100 permit ip any any

Matching:
  Finance PC → HTTP (80) to server
    ✓ Matches line 1 (permit) → ALLOWED

  Finance PC → SQL (1433) to HR server
    ✓ Matches line 3 (deny) → BLOCKED

  Finance PC → ICMP ping to HR
    ✓ Matches line 4 (permit ip any any) → ALLOWED

Note: IP allows all protocols (TCP, UDP, ICMP, etc.)
```

---

## Wireless LAN (WLAN) Setup

### Access Point Configuration

```
Access Point (Cisco):

1. Basic Settings
   SSID: HQ-Enterprise
   Mode: 802.11 a/b/g/n (mixed)
   Channel: Auto
   Power: Full

2. Security Settings
   Security Type: WPA2-Personal
   Authentication: PSK (Pre-Shared Key)
   Encryption: AES
   Passphrase: MySecure123Pass!
   
3. Network Settings
   IP Address: 192.168.10.50 (static)
   Subnet: 255.255.255.0
   Gateway: 192.168.10.1
   VLAN: 10 (management)

4. VLAN Tagging (if supported)
   Native VLAN: 1
   Allowed VLANs: 10,40 (employees, guests)
```

### WLAN Security Standards

```
DEPRECATED (DO NOT USE):
├── WEP
│   ├── 40-bit encryption
│   ├── Many known vulnerabilities
│   ├── Cracked in minutes
│   └── Still showing in legacy devices

ACCEPTABLE:
├── WPA-Personal
│   ├── Improved over WEP
│   ├── Still has vulnerabilities
│   └── Acceptable for small networks

RECOMMENDED (CURRENT STANDARD):
└── WPA2-Personal (WPA-PSK)
    ├── AES-128 encryption
    ├── CCMP (Counter Mode CBC-MAC Protocol)
    ├── Pre-shared key (passphrase)
    ├── Strong against current attacks
    └── Required for PCI-DSS compliance

FUTURE:
└── WPA3-Personal
    ├── Individualized Data Encryption (OWE)
    ├── Simultaneous Authentication of Equals (SAE)
    ├── Protection against brute-force
    └── Emerging standard
```

### Client Connection Process

```
Wireless Client Connection:

1. Scan
   - Client scans available SSIDs
   - Displays: "HQ-Enterprise" signal strength 85%

2. Select
   - User selects "HQ-Enterprise"
   - Prompted for passphrase

3. Authenticate
   - Client sends passphrase
   - AP validates against configured password
   - If match: proceed to step 4
   - If no match: deny, return to step 2

4. Association
   - AP assigns IP (via DHCP or static)
   - Client obtains VLAN assignment
   - Client obtains IP from appropriate pool

5. Connected
   - Client shows "Connected to HQ-Enterprise"
   - Can communicate with network
   - Subject to ACLs for that VLAN

Example:
- Employee connects → Gets VLAN 10 → Access all resources
- Guest connects → Gets VLAN 40 → Limited to internet only
```

---

## Static IPv4 Addressing

### Server Configuration

```
Database Server (192.168.50.20)

Interface Configuration:
- IP Address: 192.168.50.20
- Subnet Mask: 255.255.255.0
- Default Gateway: 192.168.50.1
- DNS Primary: 8.8.8.8
- DNS Secondary: 8.8.4.4
- Hostname: DB-Server-01
- Domain: company.local

Why Static for Servers:
✓ Consistent IP for service access
✓ Database clients always know address
✓ DNS can reliably resolve
✓ Port forwarding works reliably
✓ Backup/replication uses fixed IPs
✓ Monitoring easier (known IPs)
```

### Static vs Dynamic Comparison

```
STATIC IP (Servers, Infrastructure)
├── Advantages
│   ├── Always same address
│   ├── Easy to remember
│   ├── Reliable for services
│   └── DNS stable
└── Disadvantages
    ├── Manual configuration
    ├── Easy to misconfigure
    └── Not scalable for many devices

DYNAMIC IP (Workstations, Guests)
├── Advantages
│   ├── Automatic assignment
│   ├── Less admin work
│   ├── Scalable for many devices
│   └── Easy onboarding
└── Disadvantages
    ├── Address changes
    ├── May disrupt services
    └── Requires DHCP server
```

---

## Host Configurations

### Workstation Setup

```
Windows PC (Finance Department):

1. Network Settings
   Method: DHCP (Automatic)
   Or Manual:
   - IP: 192.168.20.100 (in Finance VLAN range)
   - Subnet: 255.255.255.0
   - Gateway: 192.168.20.1
   - DNS 1: 8.8.8.8
   - DNS 2: 8.8.4.4

2. Computer Name
   Name: Finance-PC-01
   Domain: company.local

3. Wireless (if WiFi)
   SSID: HQ-Enterprise
   Security: WPA2-PSK
   Password: [passphrase]

4. Verification
   ipconfig /all (show current config)
   ping 192.168.20.1 (test gateway)
   ping 8.8.8.8 (test internet)
   nslookup google.com (test DNS)
```

### Network Printer Configuration

```
Printer (Shared Device):

IP Configuration:
- Method: Static or DHCP reservation
- IP: 192.168.10.200 (Management VLAN)
- Subnet: 255.255.255.0
- Gateway: 192.168.10.1
- DNS: 8.8.8.8

Access:
- Accessible from all VLANs (via subnet routing)
- ACL may restrict based on VLAN
- Print queue managed by server

SNMP for Monitoring:
- Community: public
- OIDs for:
  - Toner levels
  - Page count
  - Error messages
```

### IP Phone Configuration

```
VoIP Phone:

Configuration:
- IP: 192.168.10.150 (via DHCP)
- VLAN: 1 (access VLAN) + VoIP VLAN (voice)
- DHCP Option 150: TFTP server for configs
- TFTP Server: 192.168.10.1

Voice VLAN (802.1p priority):
- Voice VLAN: Separate from data
- Priority: 5 (highest for voice)
- QoS: High priority in scheduling

Registration:
1. Phone boots
2. Requests IP via DHCP
3. Gets TFTP server from DHCP Option 150
4. Downloads configuration
5. Registers with Call Manager
6. Ready for calls

Benefits:
✓ Separate network for voice (QoS)
✓ Priority over data packets
✓ Isolated from data security risks
```

---

## Network Verification Checklist

```
□ All routers have OSPF enabled and showing neighbors
□ All VLANs created on switches
□ Trunk ports configured with correct VLANs
□ SVI (Vlan interfaces) created with correct IPs
□ Routing enabled on L3 switch
□ DHCP pools configured for each VLAN
□ Excluded addresses set for gateways
□ Port security enabled on access ports
□ SSH configured on all routers
□ ACLs defined and applied to VLANs
□ NAT/PAT configured for internet access
□ Wireless APs configured with correct SSID/security
□ Servers have static IPs assigned
□ Test ping within VLAN (0% loss)
□ Test ping cross-VLAN (0% loss)
□ Test ping between sites (0% loss)
□ DHCP clients receive IPs
□ Wireless clients connect successfully
□ ACL blocking working as expected
□ SSH access functional
```

---

This comprehensive guide covers all technologies implemented in the enterprise network project, demonstrating proficiency in modern network design and administration.
