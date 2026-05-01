# VXLAN BGP EVPN Site-2

## 1. VXLAN Underlay

### Site2 Leaf-1

```python
feature ospf

router ospf UNDERLAY
    router-id 2.0.0.1

interface loo0
    ip address 2.0.0.1/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface ethernet1/1
    no switchport
    medium p2p
    ip unnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown
```

### Site2 Spine-101

```python
feature ospf

router ospf UNDERLAY
    router-id 2.0.0.101

interface loo0
    ip address 2.0.0.101/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface ethernet1/1
    no switchport
    medium p2p
    ip unnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown

interface ethernet1/2
    no switchport
    medium p2p
    ip unnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown
```

### Site2 BGW-1

```python
feature ospf

router ospf UNDERLAY
    router-id 2.0.0.111

interface loo0
    ip address 2.0.0.111/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface ethernet1/1
    no switchport
    medium p2p
    ip unnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown
```

### Troubleshoot the Underlay 

**1. From Site2-Spine101 check OSPF neighbors**

```python
Spine101_S2# show ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 2.0.0.1           1 FULL/ -          01:47:05 2.0.0.1         Eth1/1 
 2.0.0.111         1 FULL/ -          01:46:56 2.0.0.111       Eth1/2 
```

## 2. VXLAN Overlay

### Site2 Leaf-1 OVERLAY

```python
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65002
    router-id 2.0.0.1
    address-family l2vpn evpn
    template peer Leaf2Spine
        remote-as 65002
        update-source loo0
        address-family l2vpn evpn
            send-community both
    neighbor 2.0.0.101
        inherit peer Leaf2Spine

interface loo1
    ip address 2.0.1.1/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    no shutdown
```

### Site2 Spine-101 OVERLAY

```python
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65002
    router-id 2.0.0.101
    address-family l2vpn evpn
    template peer Spine2Leaf
        remote-as 65002
        update-source loo0
        address-family l2vpn evpn
            send-community both
            route-reflector-client
    neighbor 2.0.0.1
        inherit peer Spine2Leaf
    neighbor 2.0.0.111
        inherit peer Spine2Leaf
```

### Site2-BGW1 OVERLAY

```python
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65002
    router-id 2.0.0.111
    address-family l2vpn evpn
    template peer Leaf2Spine
        remote-as 65002
        update-source loo0
        address-family l2vpn evpn
            send-community both
    neighbor 2.0.0.101
        inherit peer Leaf2Spine

interface loo1
    ip address 2.0.1.111/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    no shutdown
```

### Troubleshoot the Overlay 

**1. From Site2 Spine-101 check BGP L2VPN EVPN neighbors**

```python
Spine101_S2# show bgp l2vpn evpn summary 
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 2.0.0.101, local AS number 65002
BGP table version is 26, L2VPN EVPN config peers 2, capable peers 2
10 network entries and 10 paths using 3640 bytes of memory
BGP attribute entries [9/3312], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
2.0.0.1         4 65002       1131       1127       26    0    0 02:33:48 0     
    
2.0.0.111       4 65002       1126       1127       26    0    0 18:37:03 0    
```

## 3. VLAN to VXLAN Mapping L2VNI/L3VNI

### Define L2VNI

**1. Configure on BGW and Leaf switches**

```python
feature vn-segment-vlan-based

vlan 10
    vn-segment 10010
vlan 20
    vn-segment 10020

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    member vni 10010
        ingress-replication protocol bgp
    member vni 10020
        ingress-replication protocol bgp
```

### Validate L2VNI

**1. Check on Leaf and BGW that L2 VLAN to VXLAN mapping is up and running**

```python
Leaf1_S2(config-if-nve-vni)# show nve vni
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       S-ND - Suppress ND        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      10010    UnicastBGP        Up    CP   L2 [10]                 
nve1      10020    UnicastBGP        Up    CP   L2 [20]                 
```

**2. On BGW check L2VPN EVPN and confirm it is receving Type2 routes from Leaf switch**

```python
BGW1_S2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 26, Local Router ID is 2.0.0.111
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 2.0.0.1:32777
*>i[2]:[0]:[0]:[48]:[0000.2000.0010]:[0]:[0.0.0.0]/216
                      2.0.1.1                           100          0 i

Route Distinguisher: 2.0.0.1:32787
*>i[2]:[0]:[0]:[48]:[0000.2000.0020]:[0]:[0.0.0.0]/216
                      2.0.1.1                           100          0 i
```

### Define L3VNI

**1. Configure on BGW and Leaf switches**

```python
feature interface-vlan

vrf context Tenant-1
    vni 100001
    rd auto
    address-family ipv4 unicast
        route-target both auto
        route-target both auto evpn

vlan 1001
    name L3VNI_Tenant-1 
    vn-segment 100001

interface vlan 1001
    vrf member Tenant-1
    ip forward
    no shutdown

interface nve1
    member vni 100001 associate-vrf
```

### Validate L3VNI

**1. Check on Leaf and BGW that L3 VLAN to VXLAN mapping is up and running**

```python
BGW1_S2# show nve vni
Codes: CP - Control Plane        DP - Data Plane          
       UC - Unconfigured         SA - Suppress ARP        
       S-ND - Suppress ND        
       SU - Suppress Unknown Unicast 
       Xconn - Crossconnect      
       MS-IR - Multisite Ingress Replication 
       HYB - Hybrid IRB mode
    
Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      10010    UnicastBGP        Up    CP   L2 [10]                 
nve1      10020    UnicastBGP        Up    CP   L2 [20]                 
nve1      100001   n/a               Up    CP   L3 [Tenant-1]           <<<
```

### Configure Anycast GW on Leaf switch

```python
fabric forwarding anycast-gateway-mac 0002.0002.0002

interface vlan 10
    vrf member Tenant-1
    ip address 192.168.10.254/24
    fabric forwarding mode anycast-gateway
    no sh

interface vlan 20
    vrf member Tenant-1
    ip address 192.168.20.254/24
    fabric forwarding mode anycast-gateway
    no sh
```

### Validate Anycast-Gateway

**1. From Host-S2, ping its GW and crosscheck ARP table**

```python
R2#ping 192.168.20.254
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.20.254, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R2#show ip arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.10.2            -   0000.2000.0010  ARPA   Vlan10
Internet  192.168.10.254          0   000b.000b.000b  ARPA   Vlan10
Internet  192.168.20.2            -   0000.2000.0020  ARPA   Vlan20
Internet  192.168.20.254          0   000b.000b.000b  ARPA   Vlan20
```

**2. On BGW make sure that routes Type2 for MAC/IP get installed into L2VPN EVPN**

```python
BGW1_S2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 34, Local Router ID is 2.0.0.111
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 2.0.0.1:32777
*>i[2]:[0]:[0]:[48]:[0000.2000.0010]:[0]:[0.0.0.0]/216
                      2.0.1.1                           100          0 i
*>i[2]:[0]:[0]:[48]:[0000.2000.0010]:[32]:[192.168.10.2]/272
                      2.0.1.1                           100          0 i
```

### Redistribute direct networks to IPV4 unicast family on Leaf switch

**1. You must enable it on BGW and Leaf switch**

```python
route-map permit-all

router bgp 65001
    address-family ipv4 unicast
        redistribute direct route-map permit-all
```

**2. On Leaf switch make sure that direct routes are installed into IPV4 BGP**

```python
Leaf1_S2(config-if)# show bgp ipv4 unicast vrf Tenant-1 
BGP routing table information for VRF Tenant-1, address family IPv4 Unicast
BGP table version is 4, Local Router ID is 192.168.10.254
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>r192.168.10.0/24    0.0.0.0                  0        100      32768 ?
*>r192.168.20.0/24    0.0.0.0                  0        100      32768 ?
```

**3. On BGW make sure that routes Type5 get installed into L2VPN EVPN**

```python
BGW1_S2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 34, Local Router ID is 2.0.0.111
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 2.0.0.1:4
*>i[5]:[0]:[0]:[24]:[192.168.10.0]/224
                      2.0.1.1                  0        100          0 ?
*>i[5]:[0]:[0]:[24]:[192.168.20.0]/224
                      2.0.1.1                  0        100          0 ?
```