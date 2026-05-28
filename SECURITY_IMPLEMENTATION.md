# Security Implementation & Design

## Overview

This document details the security controls implemented in the enterprise network design, demonstrating defense-in-depth principles across multiple layers.

## Layer 2 Security (Data Link Layer)

### Spanning Tree Protocol (STP)

**Purpose:** Prevent Layer 2 loops and broadcast storms

**Configuration:**
- Mode: PVST+ (Per-VLAN Spanning Tree Plus)
- Priority: 4096 (HQ switch is root bridge)
- BPDUGuard: Enabled globally
- PortFast: Enabled on access ports

**Benefits:**
- Automatic loop detection and blocking
- Fast convergence on topology changes
- Per-VLAN topology management

### Port Security (Recommended Enhancement)

```
interface FastEthernet0/2
  switchport mode access
  switchport access vlan 10
  switchport port-security
  switchport port-security maximum 1
  switchport port-security mac-address sticky
  switchport port-security violation shutdown
```

**Prevents:**
- MAC address spoofing
- Rogue device connection
- Unauthorized access via physical connection

---

## Layer 3 Security (Network Layer)

### Access Control Lists (ACLs)

#### VLAN 20 (Finance) - Restricted Access

**Scenario:** Finance department handles sensitive data and requires restricted access.

**Configuration:**
```
ip access-list extended FINANCE_ACL
  permit tcp 192.168.20.0 0.0.0.255 192.168.20.0 0.0.0.255
  permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
  deny tcp 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
  deny tcp 192.168.20.0 0.0.0.255 192.168.40.0 0.0.0.255
  permit ip any any

interface FastEthernet0/0.20
  ip access-group FINANCE_ACL in
```

**Rules:**
1. Finance users can communicate with each other (same VLAN)
2. Finance users can reach Admin VLAN (IT support)
3. Finance users **BLOCKED** from HR VLAN (payroll confidentiality)
4. Finance users **BLOCKED** from Guest VLAN (external threat isolation)

**Business Justification:**
- Prevents unauthorized payroll data access
- Controls lateral movement in case of breach
- Maintains confidentiality of sensitive financial data

#### VLAN 40 (Guest) - Maximum Isolation

**Scenario:** Guest network must be completely isolated from internal resources.

**Configuration:**
```
ip access-list extended GUEST_ACL
  deny tcp 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
  deny tcp 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
  deny tcp 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
  permit ip 192.168.40.0 0.0.0.255 0.0.0.0 0.0.0.0

interface FastEthernet0/0.40
  ip access-group GUEST_ACL in
```

**Rules:**
1. Guest users **BLOCKED** from Admin VLAN
2. Guest users **BLOCKED** from Finance VLAN
3. Guest users **BLOCKED** from HR VLAN
4. Guest users **PERMITTED** to internet (assumed via NAT)

**Security Benefits:**
- Prevents guest devices from scanning internal network
- Blocks access to sensitive business data
- Isolates potential malware/compromised guest devices
- Reduces attack surface for internal systems

### OSPF Security (Recommended Enhancement)

While not implemented in this simulation, production networks should use:

```
interface Serial0/0
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 MySecurePassword123

router ospf 1
  area 0 authentication message-digest
```

**Purpose:**
- Authenticates OSPF neighbors
- Prevents rogue routers from joining the network
- Uses MD5 hashing for authentication

---

## Layer 4 Security (Transport Layer)

### SSH (Secure Shell)

**Purpose:** Secure remote administration instead of Telnet

**Configuration on All Routers:**
```
ip domain-name cisco.lab
crypto key generate rsa modulus 1024
ip ssh version 2
ip ssh time-out 120
ip ssh authentication-retries 3

line vty 0 4
  transport input ssh
  login local
  password [disabled via local auth]
  exit

username admin privilege 15 secret admin123
```

**Security Features:**
- Encryption: All traffic encrypted (RSA-based)
- No plain-text password transmission
- SSH v2: Removes known vulnerabilities of v1
- Authentication retries: Brute-force prevention (3 attempts max)
- Session timeout: Prevents idle connection hijacking (120 seconds)

**Comparison: SSH vs Telnet**

| Feature | SSH | Telnet |
|---------|-----|--------|
| Encryption | Yes (RSA) | No (plain-text) |
| Authentication | Secure | Plain-text |
| Port | 22 | 23 |
| Man-in-the-middle | Resistant | Vulnerable |
| Compliance | Required | Failing |

---

## VLAN Segmentation (Logical Security)

### Department Isolation

```
ADMIN (VLAN 10)      → 192.168.10.0/24
FINANCE (VLAN 20)    → 192.168.20.0/24
HR (VLAN 30)         → 192.168.30.0/24
GUEST (VLAN 40)      → 192.168.40.0/24
BRANCH1 (VLAN 50)    → 192.168.50.0/24
BRANCH2 (VLAN 60)    → 192.168.60.0/24
```

### Benefits

| Benefit | Explanation |
|---------|-------------|
| **Broadcast Isolation** | Each VLAN is a separate broadcast domain, reducing traffic |
| **Lateral Movement** | Attackers cannot easily move between departments |
| **Policy Enforcement** | Different ACLs per VLAN enforce departmental policies |
| **Compliance** | PCI-DSS, HIPAA, SOX require network segmentation |
| **Performance** | Smaller broadcast domains = better network efficiency |

### Attack Mitigation

**Scenario:** Finance workstation gets infected with malware

Without VLAN segmentation:
- ❌ Malware scans entire network
- ❌ Attacker reaches HR payroll systems
- ❌ Attacker reaches Guest network (pivot point)
- ❌ Full network compromise

With VLAN segmentation + ACLs:
- ✓ Malware confined to Finance VLAN
- ✓ Cannot reach HR or Admin networks (ACL blocks)
- ✓ Cannot reach Guest network (ACL blocks)
- ✓ Attack contained; lateral movement prevented

---

## DHCP Security (Recommended Enhancement)

### DHCP Snooping

```
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,40,50,60

interface FastEthernet0/1
  ip dhcp snooping trust

interface range FastEthernet0/2-0/20
  ip dhcp snooping limit rate 15
  ip dhcp snooping trust disable
```

**Purpose:**
- Prevents rogue DHCP servers
- Validates DHCP messages
- Limits DHCP requests per port (prevents DHCP starvation)

### Dynamic ARP Inspection (DAI)

```
ip arp inspection vlan 10,20,30,40,50,60

interface FastEthernet0/1
  ip arp inspection trust

interface range FastEthernet0/2-0/20
  ip arp inspection limit rate 15
  ip arp inspection trust disable
```

**Purpose:**
- Prevents ARP spoofing attacks
- Validates ARP messages against DHCP snooping database
- Blocks gratuitous ARP attacks

---

## NAT/PAT for Anonymity

**Configuration (if internet connection exists):**

```
ip nat inside source list 1 interface Serial0/0 overload

ip access-list standard NAT_INSIDE
  permit 192.168.0.0 0.0.255.255

interface FastEthernet0/0
  ip nat inside

interface Serial0/0
  ip nat outside
```

**Benefits:**
- **Hides internal IP addresses** from external networks
- **Reduces exposure** to direct attacks from internet
- **Conserves public IP addresses** (PAT — Port Address Translation)
- **Acts as implicit firewall** by hiding topology

---

## Monitoring & Logging (Recommended)

### Syslog Configuration

```
logging host 192.168.10.5
logging level warnings
logging source-interface FastEthernet0/0

service timestamps log datetime msec
no logging console
logging buffered 4096
```

**Logs to monitor:**
- ACL denials (potential attacks)
- OSPF neighbor changes (topology issues)
- SSH failed authentication (brute-force attempts)
- DHCP lease rejections
- Interface state changes

### SNMP Monitoring (Recommended)

```
snmp-server community public RO
snmp-server community private RW
snmp-server contact admin@company.com
snmp-server location HQ-R1
snmp-server host 192.168.10.5 traps public
```

---

## Security Best Practices Demonstrated

| Practice | Implementation | Benefit |
|----------|-----------------|---------|
| Defense in Depth | Multiple layers (L2, L3, L4) | No single point of failure |
| Least Privilege | ACLs restrict based on role | Users only access needed resources |
| Network Segmentation | VLANs isolate departments | Reduces breach impact |
| Encryption | SSH for admin access | Protects credentials in transit |
| Loop Prevention | STP blocks redundant paths | Prevents DoS via broadcast storms |
| Authentication | SSH key-based, OSPF auth | Only authorized devices connect |
| Isolation of Guests | Separate VLAN + ACL | External devices cannot reach internal |
| Monitoring (Recommended) | Syslog, SNMP | Early detection of attacks |

---

## Compliance Alignment

### PCI-DSS (Payment Card Industry)

- **Requirement 1:** Network firewall config (✓ ACLs)
- **Requirement 4:** Encryption in transit (✓ SSH)
- **Requirement 7:** Data isolation by role (✓ VLANs + ACLs)

### HIPAA (Healthcare)

- **§164.312(e)(2)(i):** Access controls (✓ VLAN isolation)
- **§164.312(a)(2)(i):** Encryption (✓ SSH for admin)

### SOX (Sarbanes-Oxley)

- **Section 404:** IT controls documentation (✓ This document)
- **Section 302:** Financial system segregation (✓ Finance VLAN)

---

## Future Security Enhancements

Priority | Enhancement | Benefit |
|----------|-------------|---------|
| High | Implement DHCP Snooping | Prevent rogue DHCP servers |
| High | Enable OSPF Authentication | Prevent rogue routers |
| High | Dynamic ARP Inspection | Prevent ARP spoofing |
| Medium | Implement SNMP monitoring | Real-time threat detection |
| Medium | Add IPS/IDS | Detect intrusion attempts |
| Medium | Implement VPN for branches | Secure WAN links |
| Low | Add authentication radius | Centralized credential management |

---

## Incident Response Scenarios

### Scenario 1: Finance VLAN Compromise

**Detection:** Multiple DHCP requests from Finance VLAN  
**Response:**
1. ACL blocks Finance from reaching HR/Admin/Guest
2. Malware cannot propagate to other VLANs
3. Isolate infected workstation at switch level
4. Investigate Finance VLAN in quarantine

**Outcome:** Breach contained to single VLAN

### Scenario 2: Guest Network Abuse

**Detection:** Guest attempting to access Finance subnet  
**Response:**
1. ACL GUEST_ACL blocks traffic (implicit deny)
2. Switch logs port activity
3. Identify and disable attacker's port
4. Review syslog for access attempts

**Outcome:** Attack prevented; logs document evidence

### Scenario 3: Rogue Router Injection

**Detection:** Unknown OSPF neighbor (if auth enabled)  
**Response:**
1. OSPF authentication rejects rogue router
2. Network topology unchanged
3. Router isolated from OSPF domain
4. Alert network administrator

**Outcome:** Rogue device has no impact on network

---

This document demonstrates enterprise-grade security thinking and implementation for a network design project.
