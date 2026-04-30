# VXLAN BGP EVPN - Site-2

### Site2-L1 UNDERLAY

```javascript
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

### Site2-S101 UNDERLAY

```javascript
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

### Site2-L1 OVERLAY

```javascript
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65002
    router-id 2.0.0.1
    address-family l2vpn evpn
    template peer iBGP-Leaf2Spine
        remote-as 65002
        update-source loo0
        address-family l2vpn evpn
            send-community both
    neighbor 2.0.0.101
        inherit peer iBGP-Leaf2Spine

interface loo1
    ip address 2.0.1.1/32
    ip router ospf UNDERLAY area 0
    no shutdown

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    no shutdown
```

### Site2-S101 OVERLAY

```
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65002
    router-id 2.0.0.101
    address-family l2vpn evpn
    template peer iBGP-Spine2Leaf
        remote-as 65002
        update-source loo0
        address-family l2vpn evpn
            send-community both
            route-reflector-client
    neighbor 2.0.0.1
        inherit peer iBGP-Spine2Leaf
    neighbor 2.0.0.111
        inherit peer iBGP-Spine2Leaf

interface loo1
    ip address 2.0.1.101/32
    ip router ospf UNDERLAY area 0
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