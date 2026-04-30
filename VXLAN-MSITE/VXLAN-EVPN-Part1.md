# VXLAN BGP EVPN - Site-1

### Site1-L1 UNDERLAY

```javascript
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
### Site1-L2 UNDERLAY

```javascript
feature ospf

router ospf UNDERLAY
    router-id 1.0.0.2

interface loo0
    ip address 1.0.0.2/32
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

### Site1-S101 UNDERLAY

```javascript
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

interface ethernet1/3
    no switchport
    medium p2p
    ip unnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown
```

### Troubleshoot the Underlay 

**1. Ping between Loo0 from Site1-S101 to Site1-L1**
```
Site1-S101# ping 1.0.0.1 source-interface loo0
```

**2. Ping between Loo0 from Site1-S101 to Site1-L2**
```
Site1-L101# ping 1.0.0.2 source-interface loo0
```

**3. From Site1-S101 check OSPF neighbors**

```javascript
Site1-S101(config-if)# show ip ospf neighbors 
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 1.0.0.1           1 FULL/ -          00:01:05 1.0.0.1         Eth1/1 
 1.0.0.2           1 FULL/ -          00:00:05 1.0.0.2         Eth1/2 
```

### Site1-L1 OVERLAY

```javascript
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65001
    router-id 1.0.0.1
    address-family l2vpn evpn
    template peer iBGP-Leaf2Spine
        remote-as 65001
        update-source loo0
        address-family l2vpn evpn
            send-community both
    neighbor 1.0.0.101
        inherit peer iBGP-Leaf2Spine

interface loo1
    ip address 1.0.1.1/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    no shutdown
```


### Site1-L2 OVERLAY

```javascript
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65001
    router-id 1.0.0.2
    address-family l2vpn evpn
    template peer iBGP-Leaf2Spine
        remote-as 65001
        update-source loo0
        address-family l2vpn evpn
            send-community both
    neighbor 1.0.0.101
        inherit peer iBGP-Leaf2Spine

interface loo1
    ip address 1.0.1.2/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    no shutdown
```

### Site1-S101 OVERLAY

```
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65001
    router-id 1.0.0.101
    address-family l2vpn evpn
    template peer iBGP-Spine2Leaf
        remote-as 65001
        update-source loo0
        address-family l2vpn evpn
            send-community both
            route-reflector-client
    neighbor 1.0.0.1
        inherit peer iBGP-Spine2Leaf
    neighbor 1.0.0.2
        inherit peer iBGP-Spine2Leaf
    neighbor 1.0.0.111
        inherit peer iBGP-Spine2Leaf

interface loo1
    ip address 1.0.1.101/32
    ip router ospf UNDERLAY area 0
    no shutdown
```

### Troubleshoot the Overlay 

**1. From Site1-S101 check BGP L2VPN EVPN neighbors**

```
Site1-S101(config-router-neighbor)# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 1.0.0.101, local AS number 65001
BGP table version is 4, L2VPN EVPN config peers 2, capable peers 2
0 network entries and 0 paths using 0 bytes of memory
BGP attribute entries [0/0], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS    MsgRcvd    MsgSent   TblVer  InQ OutQ Up/Down  State/
PfxRcd
1.0.0.1         4 65001          8          8        4    0    0 00:02:18 0     
    
1.0.0.2         4 65001          6         10        4    0    0 00:00:15 0     
```

## Site1-L1 to Site1-L2 VPC
```
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
```javascript
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

## Define L2VNI VLAN to VXLAN mapping

**1. Apply in both Site1-L1 and Site1-L2**

```javascript
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


## Define L3VNI VLAN to VXLAN mapping

```javascript
feature interface-vlan

vrf context Tenant-1
    vni 100001
    rd auto
    address-family ipv4 unicast
        route-target both auto
        route-target both auto evpn

vrf context Tenant-2
    vni 100002

vlan 1000
    vn-segment 100001

interface vlan 1000
    vrf member Tenant-1
    ip forward
    no shutdown

interface nve1
    member vni 100001 associate-vrf
```


## Configure Anycast GW on Site1-L1 and Site1-L2

```javascript
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