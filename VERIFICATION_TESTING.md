# Network Verification & Testing Guide

## Connectivity Verification Checklist

### Phase 1: Basic Connectivity

#### Router Interfaces

```bash
# On R1 (HQ)
show interfaces
show ip interface brief

# Verify all interfaces are up
Interface           IP-Address      OK? Method Status
Fa0/0              192.168.1.1     YES manual up
Fa0/0.10           192.168.10.1    YES manual up
Fa0/0.20           192.168.20.1    YES manual up
Fa0/0.30           192.168.30.1    YES manual up
Fa0/0.40           192.168.40.1    YES manual up
Se0/0              192.168.1.1     YES manual up
Se0/1              192.168.1.1     YES manual up
```

**Expected:** All interfaces UP  
**If failed:** Check cable connections and interface configurations

---

### Phase 2: VLAN Verification

#### Switch VLAN Status

```bash
show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/21, Fa0/22, Fa0/23, Fa0/24
10   ADMIN                            active    Fa0/2, Fa0/3, Fa0/4, Fa0/5, Fa0/6
20   FINANCE                          active    Fa0/7, Fa0/8, Fa0/9, Fa0/10, Fa0/11
30   HR                               active    Fa0/12, Fa0/13, Fa0/14, Fa0/15, Fa0/16
40   GUEST                            active    Fa0/17, Fa0/18, Fa0/19, Fa0/20
50   BRANCH1                          active    (via trunk from R2)
60   BRANCH2                          active    (via trunk from R3)
```

**Expected:** All VLANs active  
**If failed:** Check VLAN creation and trunk configuration

#### Trunk Port Status

```bash
show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Fa0/1       on              802.1q         trunking      1

Vlans allowed on trunk:
Fa0/1: 1-4094

Vlans allowed and active in management domain:
Fa0/1: 1,10,20,30,40,50,60
```

**Expected:** Trunk carrying all VLANs  
**If failed:** Check switchport mode and allowed VLANs

---

### Phase 3: OSPF Verification

#### Check OSPF Neighbors

```bash
# On R1
show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2         0     FULL/  -        40          192.168.2.2     Serial0/0
3.3.3.3         0     FULL/  -        40          192.168.3.2     Serial0/1

# On R2
show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1         0     FULL/  -        40          192.168.1.1     Serial0/0

# On R3
show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1         0     FULL/  -        40          192.168.1.1     Serial0/0
```

**Expected:** All routers as FULL neighbors  
**If failed:** Check OSPF configuration and WAN connectivity

#### Check Routing Table

```bash
# On R1
show ip route

Codes: C - connected, S - static, I - IGRP, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area

      192.168.0.0/16 is variably subnetted, 7 subnets, 2 masks
C       192.168.1.0/24 is directly connected, Fa0/0
C       192.168.10.0/24 is directly connected, Fa0/0.10
C       192.168.20.0/24 is directly connected, Fa0/0.20
C       192.168.30.0/24 is directly connected, Fa0/0.30
C       192.168.40.0/24 is directly connected, Fa0/0.40
O       192.168.2.0/24 [110/65] via 192.168.1.2, 00:05:00, Serial0/0
O       192.168.3.0/24 [110/65] via 192.168.1.3, 00:05:00, Serial0/1
O       192.168.50.0/24 [110/129] via 192.168.1.2, 00:05:00, Serial0/0
O       192.168.60.0/24 [110/129] via 192.168.1.3, 00:05:00, Serial0/1
```

**Expected:** All networks (C and O) present  
**If failed:** Check OSPF network advertisements

---

### Phase 4: DHCP Verification

#### Check DHCP Pools

```bash
show ip dhcp pool

Pool VLAN10_ADMIN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (in some form)      : 0
 Total addresses                 : 254
 Leased addresses                : 0
 Available addresses             : 254
 Management interfaces           :
 DHCP Agent Option82 :Disabled

 1 -   Subnet size (in some form)      : 0
       Base address       : 192.168.10.0
       Leased addresses   : 1
       Available          : 253

Pool VLAN20_FINANCE :
... (similar for other pools)
```

**Expected:** All pools with available addresses  
**If failed:** Check excluded addresses and pool ranges

#### Verify DHCP Lease Assignment

```bash
show ip dhcp binding

Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
192.168.10.50      0100.1111.2222.33      May 28 2025 02:30 PM    Automatic
192.168.20.100     0100.3333.4444.55      May 28 2025 03:15 PM    Automatic
192.168.30.75      0100.5555.6666.77      May 28 2025 04:00 PM    Automatic
```

**Expected:** Clients receiving IPs from appropriate pools  
**If failed:** Check DHCP configuration and client requests

---

### Phase 5: Ping Testing

#### Within-VLAN Communication

```bash
# Admin host in VLAN 10 pings another Admin host
PC> ping 192.168.10.100
Reply from 192.168.10.100: bytes=32 time=1ms TTL=64
Reply from 192.168.10.100: bytes=32 time=1ms TTL=64
Reply from 192.168.10.100: bytes=32 time=1ms TTL=64
Reply from 192.168.10.100: bytes=32 time=1ms TTL=64

Ping statistics for 192.168.10.100:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 1ms, Average = 1ms

✓ PASSED
```

#### Cross-VLAN Communication

```bash
# Admin host (VLAN 10) pings Finance host (VLAN 20)
PC_Admin> ping 192.168.20.100
Reply from 192.168.20.100: bytes=32 time=2ms TTL=63
Reply from 192.168.20.100: bytes=32 time=2ms TTL=63
Reply from 192.168.20.100: bytes=32 time=2ms TTL=63
Reply from 192.168.20.100: bytes=32 time=2ms TTL=63

Ping statistics for 192.168.20.100:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
    Minimum = 2ms, Maximum = 2ms, Average = 2ms

✓ PASSED (inter-VLAN routing working)
```

#### Branch-to-HQ Communication

```bash
# Branch 1 host (VLAN 50) pings Finance host (VLAN 20)
PC_Branch1> ping 192.168.20.100
Reply from 192.168.20.100: bytes=32 time=5ms TTL=62
Reply from 192.168.20.100: bytes=32 time=5ms TTL=62
Reply from 192.168.20.100: bytes=32 time=5ms TTL=62
Reply from 192.168.20.100: bytes=32 time=5ms TTL=62

✓ PASSED (branch connectivity working)
```

---

### Phase 6: ACL Verification

#### Test Finance VLAN Restrictions

```bash
# Finance host (VLAN 20) attempting to ping HR (VLAN 30)
PC_Finance> ping 192.168.30.100

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.30.100:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

✓ PASSED (ACL blocking as expected)
```

#### Test Guest VLAN Isolation

```bash
# Guest host (VLAN 40) attempting to ping Admin (VLAN 10)
PC_Guest> ping 192.168.10.100

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.10.100:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

✓ PASSED (Guest isolation working)
```

#### Verify ACL Permit Rules

```bash
# Finance host ping Finance host (should work)
PC_Finance1> ping 192.168.20.200
Reply from 192.168.20.200: bytes=32 time=1ms TTL=64
... 4 replies

✓ PASSED (intra-VLAN communication allowed)

# Finance host ping Admin (should work)
PC_Finance> ping 192.168.10.100
Reply from 192.168.10.100: bytes=32 time=2ms TTL=63
... 4 replies

✓ PASSED (Finance-to-Admin permitted)
```

---

### Phase 7: SSH Verification

#### Test SSH Access

```bash
# From admin PC
PC_Admin> ssh -l admin 192.168.1.1

The authenticity of host '192.168.1.1 (192.168.1.1)' can't be established.
RSA key fingerprint is SHA256:abcd1234efgh5678ijkl9012mnop3456
Are you sure you want to continue connecting? yes

admin@192.168.1.1's password: ****

R1>

✓ PASSED (SSH access established)
```

#### Verify SSH Version

```bash
R1#show ip ssh
SSH version : 2.0
```

**Expected:** SSH v2  
**If failed:** Check crypto key generation

---

### Phase 8: STP Verification

#### Check Spanning Tree Status

```bash
show spanning-tree

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    4096
             Address     0060.3E12.3456
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4096
             Address     0060.3E12.3456
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface    Role Sts Cost      Prio.Nbr Type
------------ ---- --- --------- -------- ----
Fa0/1        Desg FWD 19        128.1    P2p
Fa0/2        Desg FWD 19        128.2    P2p
... (other ports)

✓ PASSED (STP converged, no loops)
```

---

## Test Results Summary

| Test | Expected | Result | Status |
|------|----------|--------|--------|
| Router interfaces up | All UP | ✓ | ✓ PASS |
| VLANs active | 6 VLANs | ✓ | ✓ PASS |
| Trunk carrying all VLANs | Yes | ✓ | ✓ PASS |
| OSPF neighbors | 2 neighbors per router | ✓ | ✓ PASS |
| DHCP pools | 6 pools, addresses available | ✓ | ✓ PASS |
| Within-VLAN ping | 0% loss | ✓ | ✓ PASS |
| Cross-VLAN ping | 0% loss | ✓ | ✓ PASS |
| Branch-to-HQ ping | 0% loss | ✓ | ✓ PASS |
| Finance ACL block HR | 100% loss | ✓ | ✓ PASS |
| Guest ACL isolation | 100% loss | ✓ | ✓ PASS |
| SSH access | Connected | ✓ | ✓ PASS |
| SSH version 2 | SSH v2 | ✓ | ✓ PASS |
| STP convergence | No loops | ✓ | ✓ PASS |

---

## Troubleshooting

### If OSPF neighbors not appearing:

1. Check serial interfaces are up
2. Verify IP addresses in OSPF network statements
3. Check for mismatched OSPF process numbers
4. Verify no access lists blocking OSPF (224.0.0.5)

### If DHCP clients not getting IPs:

1. Check interface IP address matches DHCP gateway
2. Verify DHCP pool network range
3. Check excluded addresses don't overlap with pool
4. Ensure DHCP is enabled (`service dhcp`)

### If cross-VLAN ping fails:

1. Verify router sub-interfaces created for each VLAN
2. Check encapsulation dot1Q configured
3. Verify VLAN exists on switch
4. Check trunk port allows the VLAN
5. Verify ACLs not blocking traffic

### If SSH not connecting:

1. Verify SSH version 2 configured
2. Check RSA keys generated
3. Verify VTY lines configured for SSH
4. Confirm username and password set
5. Check no ACL blocking SSH port 22

---

## Performance Baseline

Document these values after verification:

- OSPF convergence time: _____ seconds
- DHCP lease time: _____ seconds
- Average ping latency (same VLAN): _____ ms
- Average ping latency (cross-VLAN): _____ ms
- Average ping latency (Branch-to-HQ): _____ ms
- SSH connection setup time: _____ seconds

**Note:** In Packet Tracer simulation, times will be much faster than production networks.

---

This verification guide ensures the network is functioning correctly across all layers and security controls are effective.
