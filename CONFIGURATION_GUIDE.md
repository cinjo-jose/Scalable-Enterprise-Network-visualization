# Enterprise Network Configuration Reference

## Router Configurations

### R1 (Headquarters) - Core Router

#### Interface Configuration

```
Interface Fa0/0 (To Switch - Trunk)
  ip address 192.168.1.1 255.255.255.0
  no shutdown

Subinterface Fa0/0.10 (VLAN 10 - Admin)
  encapsulation dot1Q 10
  ip address 192.168.10.1 255.255.255.0
  
Subinterface Fa0/0.20 (VLAN 20 - Finance)
  encapsulation dot1Q 20
  ip address 192.168.20.1 255.255.255.0

Subinterface Fa0/0.30 (VLAN 30 - HR)
  encapsulation dot1Q 30
  ip address 192.168.30.1 255.255.255.0

Subinterface Fa0/0.40 (VLAN 40 - Guest)
  encapsulation dot1Q 40
  ip address 192.168.40.1 255.255.255.0

Interface Se0/0 (WAN to Branch 1)
  ip address 192.168.1.1 255.255.255.0
  no shutdown

Interface Se0/1 (WAN to Branch 2)
  ip address 192.168.1.1 255.255.255.0
  no shutdown
```

#### OSPF Configuration

```
router ospf 1
  network 192.168.1.0 0.0.0.255 area 0
  network 192.168.10.0 0.0.0.255 area 0
  network 192.168.20.0 0.0.0.255 area 0
  network 192.168.30.0 0.0.0.255 area 0
  network 192.168.40.0 0.0.0.255 area 0
  router-id 1.1.1.1
```

#### DHCP Configuration

```
ip dhcp pool VLAN10_ADMIN
  network 192.168.10.0 255.255.255.0
  default-router 192.168.10.1
  dns-server 8.8.8.8 8.8.4.4
  lease 1

ip dhcp pool VLAN20_FINANCE
  network 192.168.20.0 255.255.255.0
  default-router 192.168.20.1
  dns-server 8.8.8.8 8.8.4.4
  lease 1

ip dhcp pool VLAN30_HR
  network 192.168.30.0 255.255.255.0
  default-router 192.168.30.1
  dns-server 8.8.8.8 8.8.4.4
  lease 1

ip dhcp pool VLAN40_GUEST
  network 192.168.40.0 255.255.255.0
  default-router 192.168.40.1
  dns-server 8.8.8.8 8.8.4.4
  lease 1

ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp excluded-address 192.168.40.1 192.168.40.10
```

#### ACL Configuration

```
! Finance VLAN restrictions
ip access-list extended FINANCE_ACL
  permit tcp 192.168.20.0 0.0.0.255 192.168.20.0 0.0.0.255
  permit tcp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
  deny tcp 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
  deny tcp 192.168.20.0 0.0.0.255 192.168.40.0 0.0.0.255
  permit ip any any

! Guest VLAN isolation
ip access-list extended GUEST_ACL
  deny tcp 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
  deny tcp 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
  deny tcp 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
  permit ip 192.168.40.0 0.0.0.255 0.0.0.0 0.0.0.0

interface Fa0/0.20
  ip access-group FINANCE_ACL in

interface Fa0/0.40
  ip access-group GUEST_ACL in
```

#### SSH Configuration

```
ip domain-name cisco.lab
crypto key generate rsa
  modulus 1024

ip ssh version 2
ip ssh time-out 120
ip ssh authentication-retries 3

line vty 0 4
  transport input ssh
  login local
  exit

username admin privilege 15 secret admin123
```

#### NAT Configuration (Optional - for Internet Access)

```
ip nat inside source list 1 interface Se0/0 overload
ip access-list standard NAT_INSIDE
  permit 192.168.0.0 0.0.255.255

interface Fa0/0
  ip nat inside

interface Se0/0
  ip nat outside
```

---

### R2 (Branch 1 Router)

#### Interface Configuration

```
Interface Fa0/0 (To Switch)
  ip address 192.168.2.1 255.255.255.0
  no shutdown

Subinterface Fa0/0.50 (VLAN 50 - Branch 1)
  encapsulation dot1Q 50
  ip address 192.168.50.1 255.255.255.0

Interface Se0/0 (WAN to HQ - R1)
  ip address 192.168.2.2 255.255.255.0
  no shutdown
```

#### OSPF Configuration

```
router ospf 1
  network 192.168.2.0 0.0.0.255 area 0
  network 192.168.50.0 0.0.0.255 area 0
  router-id 2.2.2.2
```

#### DHCP Configuration

```
ip dhcp pool VLAN50_BRANCH1
  network 192.168.50.0 255.255.255.0
  default-router 192.168.50.1
  dns-server 8.8.8.8 8.8.4.4
  lease 1

ip dhcp excluded-address 192.168.50.1 192.168.50.10
```

#### SSH Configuration

```
ip domain-name cisco.lab
crypto key generate rsa modulus 1024
ip ssh version 2
line vty 0 4
  transport input ssh
  login local

username admin privilege 15 secret admin123
```

---

### R3 (Branch 2 Router)

#### Interface Configuration

```
Interface Fa0/0 (To Switch)
  ip address 192.168.3.1 255.255.255.0
  no shutdown

Subinterface Fa0/0.60 (VLAN 60 - Branch 2)
  encapsulation dot1Q 60
  ip address 192.168.60.1 255.255.255.0

Interface Se0/0 (WAN to HQ - R1)
  ip address 192.168.3.2 255.255.255.0
  no shutdown
```

#### OSPF Configuration

```
router ospf 1
  network 192.168.3.0 0.0.0.255 area 0
  network 192.168.60.0 0.0.0.255 area 0
  router-id 3.3.3.3
```

#### DHCP Configuration

```
ip dhcp pool VLAN60_BRANCH2
  network 192.168.60.0 255.255.255.0
  default-router 192.168.60.1
  dns-server 8.8.8.8 8.8.4.4
  lease 1

ip dhcp excluded-address 192.168.60.1 192.168.60.10
```

#### SSH Configuration

```
ip domain-name cisco.lab
crypto key generate rsa modulus 1024
ip ssh version 2
line vty 0 4
  transport input ssh
  login local

username admin privilege 15 secret admin123
```

---

## Switch Configuration

### HQ Switch (Layer 3 capable)

#### VLAN Configuration

```
vlan 10
  name ADMIN

vlan 20
  name FINANCE

vlan 30
  name HR

vlan 40
  name GUEST

vlan 50
  name BRANCH1

vlan 60
  name BRANCH2
```

#### Trunk Port (to R1)

```
interface FastEthernet0/1
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30,40,50,60
  switchport trunk native vlan 1
  spanning-tree portfast disable
```

#### Access Ports

```
! Admin VLAN
interface range FastEthernet0/2-0/6
  switchport mode access
  switchport access vlan 10
  spanning-tree portfast

! Finance VLAN
interface range FastEthernet0/7-0/11
  switchport mode access
  switchport access vlan 20
  spanning-tree portfast

! HR VLAN
interface range FastEthernet0/12-0/16
  switchport mode access
  switchport access vlan 30
  spanning-tree portfast

! Guest VLAN
interface range FastEthernet0/17-0/20
  switchport mode access
  switchport access vlan 40
  spanning-tree portfast
```

#### STP Configuration

```
spanning-tree mode pvst
spanning-tree vlan 1,10,20,30,40,50,60 priority 4096
spanning-tree vlan 1,10,20,30,40,50,60 portfast bpdu guard default
```

---

## Verification Commands

```
! Check OSPF neighbors
show ip ospf neighbor

! Check routing table
show ip route

! Verify VLAN configuration
show vlan brief

! Check DHCP pools
show ip dhcp pool

! Verify ACL configuration
show access-lists

! Test connectivity
ping 192.168.20.10
ping 192.168.50.10
ping 192.168.60.10

! Verify SSH
show ip ssh

! Check interface status
show interfaces
```

---

## Troubleshooting Checklist

- [ ] All routers have OSPF enabled and neighbors established
- [ ] All VLANs created on all switches
- [ ] Trunk ports configured with correct VLAN allowance
- [ ] Sub-interfaces created on router for each VLAN
- [ ] DHCP pools configured for each VLAN
- [ ] Default gateway points to correct router sub-interface
- [ ] SSH keys generated and configured
- [ ] ACLs applied to correct interfaces
- [ ] STP converged without loops
- [ ] All devices can ping their default gateway
- [ ] Inter-VLAN routing working (test cross-VLAN pings)
- [ ] Guest VLAN properly isolated per ACL
