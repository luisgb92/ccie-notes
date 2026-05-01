# VXLAN BGP EVPN - Site-2

### Site2-Leaf1 UNDERLAY

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

### Site2-Spine101 UNDERLAY

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

### Site2-BGW1 UNDERLAY

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

**1. From Site1-Spine101 check OSPF neighbors**

```python
Site1-S101(config-if)# show ip ospf neighbors 
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 1.0.0.1           1 FULL/ -          00:01:05 1.0.0.1         Eth1/1 
 1.0.0.2           1 FULL/ -          00:00:05 1.0.0.2         Eth1/2 
```

### Site2-Leaf1 OVERLAY

```python
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65002
    router-id 2.0.0.1
    address-family l2vpn evpn
    template peer Leaf2Spine
        remote-as 65001
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

### Site2-Spine101 OVERLAY

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

### Site1-BGW1 OVERLAY

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
        inherit peer iBGP-Leaf2Spine

interface loo1
    ip address 2.0.1.111/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    no shutdown
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


### Define L3VNI VLAN to VXLAN mapping

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


### Configure Anycast GW on Site2-L1 and Site2-L2

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