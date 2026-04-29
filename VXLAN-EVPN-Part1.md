# VLAN EVPN

### Leaf-1 UNDERLAY

```
feature ospf

interface loo0
    ip address 1.0.0.1/32
    no shutdown

router ospf UNDERLAY
    router-id 1.0.0.1

interface ethernet1/1
    no switchport
    medium p2p
    ip unnnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point

```
### Leaf-2 UNDERLAY

```
feature ospf

interface loo0
    ip address 1.0.0.2/32
    no shutdown

router ospf UNDERLAY
    router-id 1.0.0.2

interface ethernet1/1
    no switchport
    medium p2p
    ip unnnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point

```

### Spine-101 UNDERLAY

```
feature ospf

interface loo0
    ip address 1.0.0.101/32
    no shutdown

router ospf UNDERLAY
    router-id 1.0.0.101

interface ethernet1/1
    no switchport
    medium p2p
    ip unnnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point

interface ethernet1/2
    no switchport
    medium p2p
    ip unnnumbered loo0
    ip router ospf UNDERLAY area 0
    ip ospf network point-to-point

```
