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

interface loo1
    ip address 2.0.1.101/32
    ip router ospf UNDERLAY area 0
    no shutdown
```