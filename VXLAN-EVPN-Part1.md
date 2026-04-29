# VXLAN BGP EVPN - OSPF Underlay

### Leaf-1 UNDERLAY

```
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
    ip unnnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown

```
### Leaf-2 UNDERLAY

```
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
    ip unnnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown
```

### Spine-101 UNDERLAY

```
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
    ip unnnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown

interface ethernet1/2
    no switchport
    medium p2p
    ip unnnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point
    no shutdown
```

### Troubleshoot the Underlay 

**Ping between Loo0 from Spine-101 to Leaf-1**
```
Leaf-1# ping 1.0.0.1 source-interface loo0
```

**Ping between Loo0 from Spine-101 to Leaf-2**
```
Leaf-1# ping 1.0.0.2 source-interface loo0
```

### Leaf-1 OVERLAY

```
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
        inherit iBGP-Leaf2Spine

interface loo1
    ip address 1.0.1.1/32
    ip address 1.0.1.121/32 secondary
    ip router ospf UNDERLAY area 0
    no shutdown

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    no shutdown
```


### Leaf-2 OVERLAY

```
feature bgp
feature nv overlay
nv overlay evpn

router bgp 65001
    router-id 1.0.0.2
    address-family l2vpn evpn
    template peer iBGP-Leaf2spine
        remote-as 65001
        update-source loo0
        address-family l2vpn evpn
            send-community both
    neighbor 1.0.0.101
        inherit iBGP-Leaf2Spine

interface loo1
    ip address 1.0.1.2/32
    ip address 1.0.1.121/32 secondary
    ip router ospf UNDERLAY area 0
    no shutdown

interface nve1
    source-interface loo1
    host-reachability protocol bgp
    no shutdown
```

### Spine-101 OVERLAY

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
        inherit peer iBGP-Spine2leaf
    neighbor 1.0.0.2
        inherit peer iBGP-Spine2leaf

interface loo1
    ip address 1.0.1.101/32
    ip router ospf UNDERLAY area 0
    no shutdown
```

## Leaf-1 to Leaf-2 VPC
```
feature vpc
feature lacp

int mgmt0
    ip address 192.168.1.1/30
    no shutdown

vpc domain 1
    peer-keepalive destination 192.168.1.2 source 192.168.1.1
    peer-switch
    peer-gateway

interface eth1/2
    switchport mode trunk
    channel-group 500 mode active

interface po500
    vpc peer-link

interface eth1/3
    switchport mode trunk
    channel-group 1 mode active

interface po1
    vpc 1
```


## Leaf-2 to Leaf-1 VPC
```
feature vpc
feature lacp

int mgmt0
    ip address 192.168.1.2/30
    no shutdown

vpc domain 1
    peer-keepalive destination 192.168.1.1 source 192.168.1.2
    peer-switch
    peer-gateway

interface eth1/2
    switchport mode trunk
    channel-group 500 mode active

interface po500
    vpc peer-link

interface eth1/3
    switchport mode trunk
    channel-group 1 mode active

interface po1
    vpc 1

```
