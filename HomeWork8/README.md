# Домашнее задание №8
## Построение Overlay сети VxLAN маршруты type 5

### Задание:
- 
- 
- 

## Решение:

### Схема сети
![](img/Schema_VxLAN_L3_R5.png)

## Конфигурации:

- [spine-1](Config/spine-1.cfg)

```
interface Loopback1
   ip address 10.0.1.0/32

ip prefix-list BGP_out seq 10 permit 10.0.1.0/32

route-map BGP_out_map permit 10
   match ip address prefix-list BGP_out

peer-filter EVPN
   10 match as-range 65001-65003 result accept

peer-filter LEAFS
   10 match as-range 65001-65003 result accept

router bgp 65000
   router-id 10.0.1.0
   bgp default ipv4-unicast transport ipv6
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 10.0.3.0/24 peer-group EVPN peer-filter EVPN
   bgp listen range fe80::/64 peer-group LEAFS peer-filter LEAFS
   neighbor EVPN peer group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor LEAFS peer group
   neighbor LEAFS bfd
   neighbor LEAFS auto-local-addr
   neighbor LEAFS password 7 b86CRLUyvVyBSvjcT7UH5A==
   neighbor LEAFS send-community
   redistribute connected route-map BGP_out_map
   
   address-family evpn
      neighbor EVPN activate
   
   address-family ipv4
      neighbor LEAFS activate
   
   address-family ipv6
      neighbor LEAFS activate

```
- [spine-2](Config/spine-2.cfg)

```
interface Loopback1
   ip address 10.0.2.0/32

ip prefix-list BGP_out seq 10 permit 10.0.2.0/32

route-map BGP_out_map permit 10
   match ip address prefix-list BGP_out

peer-filter EVPN
   10 match as-range 65001-65003 result accept

peer-filter LEAFS
   10 match as-range 65001-65003 result accept

router bgp 65000
   router-id 10.0.2.0
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   bgp listen range 10.0.3.0/24 peer-group EVPN peer-filter EVPN
   bgp listen range fe80::/64 peer-group LEAFS peer-filter LEAFS
   neighbor EVPN peer group
   neighbor EVPN next-hop-unchanged
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor LEAFS peer group
   neighbor LEAFS bfd
   neighbor LEAFS auto-local-addr
   neighbor LEAFS password 7 b86CRLUyvVyBSvjcT7UH5A==
   neighbor LEAFS send-community
   redistribute connected route-map BGP_out_map
   
   address-family evpn
      neighbor EVPN activate
   
   address-family ipv4
      neighbor LEAFS activate
   
   address-family ipv6
      neighbor LEAFS activate

```
- [leaf-1](Config/leaf-1.cfg)

```
vlan 100,201

vrf instance BLUE

vrf instance RED

interface Ethernet3
   description Link to VPC_1
   switchport access vlan 100

interface Ethernet4
   description Link to VPC_4
   switchport access vlan 201

interface Vlan100
   vrf RED
   ip address virtual 192.168.100.1/24

interface Vlan201
   vrf BLUE
   ip address virtual 192.168.201.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 100 vni 10010
   vxlan vlan 201 vni 10021
   vxlan vrf BLUE vni 444
   vxlan vrf RED vni 333
   vxlan learn-restrict any

ip routing
ip routing vrf BLUE
ip routing vrf RED

router bgp 65001
...
vlan 100
      rd 65001:10010
      route-target both 10:10010
      redistribute learned
   
   vlan 201
      rd 65001:10021
      route-target both 20:10021
      redistribute learned
   
   address-family evpn
      neighbor EVPN activate
   
   address-family ipv4
      neighbor SPINES activate
   
   address-family ipv6
      neighbor SPINES activate
   
   vrf BLUE
      rd 65001:444
      route-target import evpn 33:444
      route-target export evpn 33:444
   
   vrf RED
      rd 65001:333
      route-target import evpn 33:333
      route-target export evpn 33:333
```

- [leaf-2](Config/leaf-2.cfg)

```
vlan 101,200

vrf instance BLUE

vrf instance RED

interface Ethernet3
   description Link to VPC_3
   switchport access vlan 200

interface Ethernet4
   description Linkto VPC_4
   switchport access vlan 101

interface Vlan101
   vrf RED
   ip address virtual 192.168.101.1/24

interface Vlan200
   vrf BLUE
   ip address virtual 192.168.200.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 101 vni 10011
   vxlan vlan 200 vni 10020
   vxlan vrf BLUE vni 444
   vxlan vrf RED vni 333
   vxlan learn-restrict any

ip routing
ip routing vrf BLUE
ip routing vrf RED

router bgp 65002
...
vlan 101
      rd 65002:333
      route-target both 10:10011
      redistribute learned
   !
   vlan 200
      rd 65002:10020
      route-target both 20:10020
      redistribute learned
   !
   address-family evpn
      neighbor EVPN activate
   !
   address-family ipv4
      neighbor SPINES activate
   !
   address-family ipv6
      neighbor SPINES activate
   !
   vrf BLUE
      rd 65002:444
      route-target import evpn 33:444
      route-target export evpn 33:444
   !
   vrf RED
      rd 65002:333
      route-target import evpn 33:333
      route-target export evpn 33:333
```
- [leaf-3](Config/leaf-3.cfg)

```
vlan 10,20,110-111,120-121

vrf instance BLUE

vrf instance RED

interface Ethernet3
   description Link to VPC_3
   switchport access vlan 110

interface Ethernet4
   description Link to VPC_4
   switchport access vlan 120

interface Ethernet5
   description Link to VPC_7
   switchport access vlan 111
   switchport trunk allowed vlan 10,20

interface Ethernet6
   description Link tto VPC_8
   switchport access vlan 121

interface Ethernet8
   description Link to GW
   switchport trunk allowed vlan 10,20
   switchport mode trunk

interface Vlan10
   description Link to GW 
   vrf RED
   ip address 172.17.0.1/30

interface Vlan20
   description Link to GW 
   vrf BLUE
   ip address 172.17.0.5/30

interface Vlan110
   vrf RED
   ip address virtual 192.168.100.1/24

interface Vlan111
   vrf RED
   ip address virtual 192.168.101.1/24

interface Vlan120
   vrf BLUE
   ip address virtual 192.168.200.1/24

interface Vlan121
   vrf BLUE
   ip address virtual 192.168.201.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 110 vni 10010
   vxlan vlan 111 vni 10011
   vxlan vlan 120 vni 10020
   vxlan vlan 121 vni 10021
   vxlan vrf BLUE vni 444
   vxlan vrf RED vni 333
   vxlan learn-restrict any

ip access-list standard BLUE_IP
   10 permit 192.168.200.0/24
   20 permit 192.168.201.0/24

ip access-list standard RED_IP
   10 permit 192.168.100.0/24
   20 permit 192.168.101.0/24

ip routing
ip routing vrf BLUE
ip routing vrf RED

route-map BGP_out_map permit 10
   match ip address prefix-list BGP_out

route-map BLUE_LIST permit 10
   match ip address access-list BLUE_IP

route-map RED_LIST permit 10
   match ip address access-list RED_IP

router bgp 65003
...
vlan 110
      rd 65003:10010
      route-target both 10:10010
      redistribute learned
   
   vlan 111
      rd 65003:10011
      route-target both 10:10011
      redistribute learned
   
   vlan 120
      rd 65003:10020
      route-target both 20:10020
      redistribute learned
   
   vlan 121
      rd 65003:10021
      route-target both 20:10021
      redistribute learned
   
   address-family evpn
      neighbor EVPN activate
   
   address-family ipv4
      neighbor SPINES activate
   
   address-family ipv6
      neighbor SPINES activate
   
   vrf BLUE
      rd 65003:444
      route-target import evpn 33:444
      route-target export evpn 33:444
      neighbor 172.17.0.6 remote-as 64999
      neighbor 172.17.0.6 allowas-in 3
      network 192.168.200.0/24
      network 192.168.201.0/24
      aggregate-address 192.168.200.0/23 summary-only match-map BLUE_LIST
   
   vrf RED
      rd 65003:333
      route-target import evpn 33:333
      route-target export evpn 33:333
      neighbor 172.17.0.2 remote-as 64999
      neighbor 172.17.0.2 allowas-in 3
      network 192.168.100.0/24
      network 192.168.101.0/24
      aggregate-address 192.168.100.0/23 summary-only match-map RED_LIS

```

- [GW](Config/GW.cfg)

```
vlan 10,20

interface Ethernet1
   description Link to leaf-3
   switchport trunk allowed vlan 10,20
   switchport mode trunk

interface Loopback1
   ip address 10.0.4.1/32

interface Loopback2
   ip address 77.88.8.8/32

interface Loopback3
   ip address 8.8.8.8/32

interface Management1

interface Vlan10
   ip address 172.17.0.2/30

interface Vlan20
   ip address 172.17.0.6/30

ip routing

ip route 0.0.0.0/0 Null0

router bgp 64999
   router-id 10.0.4.1
   bgp listen range 172.17.0.0/29 peer-group leaf remote-as 65003
   neighbor leaf peer group
   redistribute static
```

