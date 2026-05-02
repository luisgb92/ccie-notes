# VXLAN BGP EVPN - MSITE

## 1. Site-External Underlay

### Site1 BGW1 

```python
feature bgp 

route-map RMAP-REDIST-DIRECT permit 10
    match tag 54321

interface loo0
    ip address 1.0.0.111/32 tag 54321
    ip router ospf UNDERLAY area 0
    no shutdown

interface loo1
    ip address 1.0.1.111/32 tag 54321
    ip router ospf UNDERLAY area 0
    no shutdown

interface loo100
    ip add 1.0.100.111/32 tag 54321
    ip router ospf UNDERLAY area 0
    no shutdown

interface eth1/2
    no switchport
    ip address 172.16.1.1/30 tag 54321
    no shutdown

route-map RMAP-REDIST-DIRECT permit 10
    match tag 54321

router bgp 65001
    router-id 1.0.0.111
    address-family ipv4 unicast
        redistribute direct route-map RMAP-REDIST-DIRECT
        maximum-paths 4
    neighbor 172.16.1.2 remote-as 65100
        update-source eth1/2
        address-family ipv4 unicast
```

### Site2 BGW1

```python
feature bgp

route-map RMAP-REDIST-DIRECT permit 10
    match tag 54321

interface loo0
    ip address 2.0.0.111/32 tag 54321
    ip router ospf UNDERLAY area 0
    no shutdown

interface loo1
    ip address 2.0.1.111/32 tag 54321
    ip router ospf UNDERLAY area 0
    no shutdown

interface loo100
    ip address 2.0.100.111/32 tag 54321
    ip router ospf UNDERLAY area 0
    no shutdown

interface eth1/2
    no switchport
    ip address 172.16.2.1/30 tag 54321
    no shutdown

router bgp 65002
    router-id 2.0.0.111
    address-family ipv4 unicast
        redistribute direct route-map RMAP-REDIST-DIRECT
        maximum-paths 4
    neighbor 172.16.2.2 remote-as 65100
        update-source eth1/2
        address-family ipv4 unicast
```

### Route-Server DCI

```python
feature bgp 

route-map RMAP-REDIST-DIRECT permit 10
    match tag 54321

int loo0
    ip address 65.100.0.0/32 tag 54321
    no sh

int eth1/1
    no switchport
    ip address 172.16.1.2/30 tag 54321
    no sh

int eth1/2
    no switchport
    ip address 172.16.2.2/30 tag 54321
    no sh

router bgp 65100
    router-id 65.100.0.0
    address-family ipv4 unicast
        redistribute direct route-map RMAP-REDIST-DIRECT
        maximum-paths 4
    neighbor 172.16.1.1 remote-as 65001
        update-source eth1/1
        address-family ipv4 unicast
    neighbor 172.16.2.1 remote-as 65002
        update-source eth1/2
        address-family ipv4 unicast
```

## 2. Site-External Overlay

### Site1 BGW1

```python
feature nv overlay
nv overlay evpn

router bgp 65001
    address-family l2vpn evpn
    template peer BGW2DCI
        remote-as 65100
        update-source loo0
        ebgp-multihop 5
        peer-type fabric-external
        address-family l2vpn evpn
            send-community both
            rewrite-evpn-rt-asn
    neighbor 65.100.0.0
        inherit peer BGW2DCI

evpn multisite border-gateway 101

interface eth1/1
    evpn multisite fabric-tracking

interface eth1/2
    evpn multisite dci-tracking

interface nve1
    multisite border-gateway interface loo100
    member vni 10010
        multisite ingress-replication
    member vni 10020
        multisite ingress-replication
    member vni 100001
```

### Site2 BGW1

```python
feature nv overlay
nv overlay evpn

router bgp 65002
    address-family l2vpn evpn
    template peer BGW2DCI
        remote-as 65100
        update-source loo0
        ebgp-multihop 5
        peer-type fabric-external
        address-family l2vpn evpn
            send-community both
            rewrite-evpn-rt-asn
    neighbor 65.100.0.0
        inherit peer BGW2DCI

evpn multisite border-gateway 102

interface eth1/1
    evpn multisite fabric-tracking

interface eth1/2
    evpn multisite dci-tracking

interface nve1
    multisite border-gateway interface loo100
    member vni 10010
        multisite ingress-replication
    member vni 10020
        multisite ingress-replication
    member vni 100001
```

### DCI Overlay

```python
feature nv overlay
nv overlay evpn

route-map UNCHANGED permit 10
    set ip next-hop unchanged

router bgp 65100
    address-family l2vpn evpn
    retain route-target all
    template peer DCI2BGW
        update-source loo0
        ebgp-multihop 5
        address-family l2vpn evpn
            send-community both
            route-map UNCHANGED out
    neighbor 1.0.0.111 remote-as 65001
        inherit peer DCI2BGW
        address-family l2vpn evpn
            rewrite-evpn-rt-asn
    neighbor 2.0.0.111 remote-as 65002
        inherit peer DCI2BGW
        address-family l2vpn evpn
            rewrite-evpn-rt-asn
```

