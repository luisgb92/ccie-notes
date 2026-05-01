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
Spine101_S2# show ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 2.0.0.1           1 FULL/ -          01:47:05 2.0.0.1         Eth1/1 
 2.0.0.111         1 FULL/ -          01:46:56 2.0.0.111       Eth1/2 
```

## 3. VLAN to VXLAN Mapping L2VNI/L3VNI

### Define L2VNI VLAN to VXLAN mapping

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

### Define L3VNI VLAN to VXLAN mapping

```python
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