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
    address-family ip4 unicast
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