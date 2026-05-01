# VXLAN BGP EVPN Site-1

## 1. VXLAN Underlay

### Site1 Leaf-1

```python
feature ospf

router ospf UNDERLAY
    router-id 1.0.0.1

interface loo0
    ip address 1.0.0.1/32
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

### Site1 Spine-101

```python
feature ospf

router ospf UNDERLAY
    router-id 1.0.0.101

interface loo0
    ip address 1.0.0.101/32
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

### Site1 BGW-1

```python
feature ospf

router ospf UNDERLAY
    router-id 1.0.0.111

interface loo0
    ip address 1.0.0.111/32
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

**1. From Site1 Spine-101 check OSPF neighbors**

```python
Site1-S101(config)# show ip ospf neighbors 
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 1.0.0.1           1 FULL/ -          00:15:14 1.0.0.1         Eth1/1 
 1.0.0.111         1 FULL/ -          00:15:03 1.0.0.111       Eth1/2 
```

## 2. VXLAN Overlay

### Site1 Leaf-1

```python
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65001
    router-id 1.0.0.1
    address-family l2vpn evpn
    template peer Leaf2Spine
        remote-as 65001
        update-source loo0
        address-family l2vpn evpn
            send-community both
    neighbor 1.0.0.101
        inherit peer Leaf2Spine

interface loo1
    ip address 1.0.1.1/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    no shutdown
```

### Site1 Spine-101

```python
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65001
    router-id 1.0.0.101
    address-family l2vpn evpn
    template peer Spine2Leaf
        remote-as 65001
        update-source loo0
        address-family l2vpn evpn
            send-community both
            route-reflector-client
    neighbor 1.0.0.1
        inherit peer Spine2Leaf
    neighbor 1.0.0.111
        inherit peer Spine2Leaf
```

### Site1 BGW-1

```python
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65001
    router-id 1.0.0.111
    address-family l2vpn evpn
    template peer Leaf2Spine
        remote-as 65001
        update-source loo0
        address-family l2vpn evpn
            send-community both
    neighbor 1.0.0.101
        inherit peer Leaf2Spine

interface loo1
    ip address 1.0.1.111/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    no shutdown
```

### Troubleshoot the Overlay 

**1. From Site1-S101 check BGP L2VPN EVPN neighbors**

```python
Spine101_S1(config)# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.0.0.101, local AS number 65001
BGP table version is 5, L2VPN EVPN config peers 2, capable peers 2
0 network entries and 0 paths using 0 bytes of memory
BGP attribute entries [0/0], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
1.0.0.1         4 65001         79         79        5    0    0 01:12:42 0     
    
1.0.0.111       4 65001         69         76        5    0    0 01:04:00 0
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
Leaf1_S1(config-if-nve-vni)# show nve vni
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
BGW1_S1(config-if-nve-vni)# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 83, Local Router ID is 1.0.0.111
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 1.0.0.1:32777
*>i[2]:[0]:[0]:[48]:[0000.1000.0010]:[0]:[0.0.0.0]/216
                      1.0.1.1                           100          0 i

Route Distinguisher: 1.0.0.1:32787
*>i[2]:[0]:[0]:[48]:[0000.1000.0020]:[0]:[0.0.0.0]/216
                      1.0.1.1                           100          0 i
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
BGW1_S1(config-if-nve-vni)# show nve vni
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
nve1      100001   n/a               Up    CP   L3 [Tenant-1]            <<<
```

### Configure Anycast GW on Leaf switch

```python
fabric forwarding anycast-gateway-mac 0001.0001.0001

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

**1. From Host-S1, ping its GW and crosscheck ARP table**

```python
R1#ping 192.168.10.254
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.10.254, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R1#show ip arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.10.1            -   0000.1000.0010  ARPA   Vlan10
Internet  192.168.10.254         11   000a.000a.000a  ARPA   Vlan10
Internet  192.168.20.1            -   0000.1000.0020  ARPA   Vlan20
Internet  192.168.20.254         11   000a.000a.000a  ARPA   Vlan20
```

**2. On BGW make sure that routes Type2 for MAC/IP get installed into L2VPN EVPN**

```python
BGW1_S1(config-if-nve-vni)# show bgp l2vpn evpn 
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 83, Local Router ID is 1.0.0.111
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 1.0.0.1:32777
*>i[2]:[0]:[0]:[48]:[0000.1000.0010]:[0]:[0.0.0.0]/216
                      1.0.1.1                           100          0 i
*>i[2]:[0]:[0]:[48]:[0000.1000.0010]:[32]:[192.168.10.1]/272
                      1.0.1.1                           100          0 i
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
Leaf1_S1(config-vrf)# show bgp ipv4 unicast vrf Tenant-1 
BGP routing table information for VRF Tenant-1, address family IPv4 Unicast
BGP table version is 16, Local Router ID is 192.168.10.254
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
BGW1_S1(config-if-nve-vni)# show bgp l2vpn evpn 
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 83, Local Router ID is 1.0.0.111
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 1.0.0.1:4
*>i[5]:[0]:[0]:[24]:[192.168.10.0]/224
                      1.0.1.1                  0        100          0 ?
*>i[5]:[0]:[0]:[24]:[192.168.20.0]/224
                      1.0.1.1                  0        100          0 ?
```


## Optional VPC

### Site1-L1 to Site1-L2 VPC

```python
feature vpc
feature lacp

interface loo1
    ip address 1.0.1.121/32 secondary

int mgmt0
    ip address 192.168.1.1/30
    no shutdown

vpc domain 1
    peer-keepalive destination 192.168.1.2 source 192.168.1.1
    peer-switch
    peer-gateway

interface eth1/2
    switchport mode trunk
    switchport trunk allowed vlan all
    channel-group 500 mode active

interface po500
    vpc peer-link

interface eth1/3
    switchport mode trunk
    channel-group 1 mode active

interface po1
    vpc 1
```

## Site1-L2 to Site1-L1 VPC
```python
feature vpc
feature lacp

interface loo1
    ip address 1.0.1.121/32 secondary


int mgmt0
    ip address 192.168.1.2/30
    no shutdown

vpc domain 1
    peer-keepalive destination 192.168.1.1 source 192.168.1.2
    peer-switch
    peer-gateway

interface eth1/2
    switchport mode trunk
    switchport trunk allowed vlan all
    channel-group 500 mode active

interface po500
    vpc peer-link

interface eth1/3
    switchport mode trunk
    channel-group 1 mode active

interface po1
    vpc 1
```