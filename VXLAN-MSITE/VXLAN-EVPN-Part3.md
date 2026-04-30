# VXLAN BGP EVPN - MSITE

### Site1-BGW1 Intenal-Underlay

```python
feature ospf 

router ospf UNDERLAY
    router-id 1.0.0.111

interface loo0
    ip add 1.0.0.111/32 tag 54321
    ip router ospf UNDERLAY area 0
    no shutdown

interface loo1
    ip add 1.0.1.111/32 tag 54321
    ip router ospf UNDERLAY area 0
    no shutdown

interface eth1/1
    no switchport
    medium p2p
    ip unnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown
```

### Site2-BGW1 Intenal-Underlay

```python
feature ospf 

router ospf UNDERLAY
    router-id 2.0.0.111

interface loo0
    ip add 2.0.0.111/32 tag 54321
    ip router ospf UNDERLAY area 0
    no shutdown

interface loo1
    ip add 2.0.1.111/32 tag 54321
    ip router ospf UNDERLAY area 0
    no shutdown

interface eth1/3
    no switchport
    medium p2p
    ip unnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown
```


### Site1-BGW1 External-Underlay

```python
feature bgp 

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

### RS DCI

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

int eth1/3
        no switchport
    ip address 172.16.2.6/30 tag 54321
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


### Site2-BGW1 External-Underlay

```python
feature bgp

route-map RMAP-REDIST-DIRECT permit 10
    match tag 54321

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

### Site1-BGW VXLAN Mapping

```python
feature nv overlay
feature vn-segment-vlan-based

vlan 10
    vn-segment 10010
vlan 20
    vn-segment 10020
vlan 1000
    vn-segment 100001

vrf context Tenant-1
    vni 100001
    rd auto
    address-family ipv4 unicast
        route-target both auto
        route-target both auto evpn

interface vlan 1000
    vrf member Tenant-1
    ip forward
    no shutdown

interface nve1
    host-reachability protocol bgp
    source-interface loo1
    member vni 10010
        ingress-replication protocol bgp
    member vni 10020
        ingress-replication protocol bgp
    member vni 100001 associate-vrf
    no shutdown
```

### Site1-BGW1 External Overlay to DCI

```python
feature nv overlay
nv overlay evpn

router bgp 65001
    address-family l2vpn evpn
    template peer eBGP-BGW2DCI
        remote-as 65100
        update-source loo0
        ebgp-multihop 5
        peer-type fabric-external
        address-family l2vpn evpn
            send-community both
            rewrite-evpn-rt-asn
    template peer iBGP-BGW2Spine
        remote-as 65001
        update-source loopback0
            address-family l2vpn evpn
            send-community
            send-community extended
    neighbor 65.100.0.0
        inherit peer eBGP-BGW2DCI
    neighbor 1.0.0.101
        inherit peer iBGP-BGW2Spine
    

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


```

### DCI to Site1-BGW1 Overlay

```python
feature nv overlay
nv overlay evpn

route-map UNCHANGED permit 10
    set ip next-hop unchanged

router bgp 65100
    address-family l2vpn evpn
    retain route-target all
    template peer eBGP-DCI2BGW
        update-source loo0
        ebgp-multihop 5
        address-family l2vpn evpn
            send-community both
            route-map UNCHANGED out
            rewrite-evpn-rt-asn
    neighbor 1.0.0.111
        remote-as 65001
        inherit peer eBGP-DCI2BGW
    neighbor 2.0.0.111
        remote-as 65002
        inherit peer eBGP-DCI2BGW

```

