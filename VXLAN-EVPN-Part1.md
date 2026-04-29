# VLAN EVPN

## L101 UNDERLAY

```

feature OSPF


interface loo0
    ip address 1.0.0.1/32
    no shutdown

interface ethernet1/1
    no switchport
    medium p2p
    ip unnnumbered loo0
    ip ospf router UNDERLAY area 0

```
