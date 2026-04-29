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
