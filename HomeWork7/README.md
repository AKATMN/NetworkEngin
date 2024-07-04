# Домашнее задание №7
## Построение Overlay сети VxLAN Multihoming

### Задание:
- Подключите клиентов 2-я линками к различным Leaf
- Настроите агрегированный канал со стороны клиента
- Настроите multihoming для работы в Overlay сети. Если используете Cisco NXOS - vPC, если иной вендор - то ESI LAG (либо MC-LAG с поддержкой VXLAN)

## Решение:

### Схема сети
![](img/Schema_VxLAN_L3_MH.png)

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
spanning-tree mode mstp
no spanning-tree vlan-id 4094

vlan 100,200

vlan 4094
   trunk group mlagpeer

vrf instance OTUS

interface Port-Channel1
   description MLAG
   switchport mode trunk
   switchport trunk group mlagpeer

interface Port-Channel10
   switchport trunk allowed vlan 100,200
   switchport mode trunk
   mlag 10

interface Port-Channel11
   switchport trunk allowed vlan 100,200
   switchport mode trunk
   mlag 11

interface Ethernet1
   description Link to spine-1
   no switchport
   ip address 10.2.1.1/31
   ipv6 enable
   ipv6 address fe80::11/64 link-local

interface Ethernet2
   description Link to spine-2
   no switchport
   ip address 10.2.2.1/31
   ipv6 enable
   ipv6 address fe80::11/64 link-local

interface Ethernet3
   channel-group 10 mode active

interface Ethernet4
   channel-group 11 mode active

interface Ethernet5
   description Port-channel-1 Link to LEAF-2
   channel-group 1 mode active

interface Ethernet6
   description Port-channel-1 Link to LEAF-2
   channel-group 1 mode active

interface Loopback1
   description EVPN_VxLAN
   ip address 10.0.3.1/32

interface Loopback2
   description router-id for BGP
   ip address 10.1.0.1/32

interface Vlan100
   vrf OTUS
   ip address virtual 192.168.100.1/24

interface Vlan200
   vrf OTUS
   ip address virtual 192.168.200.1/24

interface Vlan4094
   description MLAG Peer Link
   ip address 10.3.0.1/30

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 100 vni 10010
   vxlan vlan 200 vni 10020
   vxlan vrf OTUS vni 333
   vxlan virtual-vtep local-interface Loopback1
   vxlan learn-restrict any

ip virtual-router mac-address 00:00:00:00:00:01

mlag configuration
   domain-id MLAG-OTUS
   local-interface Vlan4094
   peer-address 10.3.0.2
   peer-link Port-Channel1

```

- [leaf-2](Config/leaf-2.cfg)

```
spanning-tree mode mstp
no spanning-tree vlan-id 4094

vlan 100,200

vlan 4094
   trunk group mlagpeer

vrf instance OTUS

interface Port-Channel1
   description MLAG
   switchport mode trunk
   switchport trunk group mlagpeer

interface Port-Channel10
   switchport trunk allowed vlan 100,200
   switchport mode trunk
   mlag 10

interface Port-Channel11
   switchport trunk allowed vlan 100,200
   switchport mode trunk
   mlag 11

interface Ethernet1
   description Link to spine-1
   no switchport
   ip address 10.2.1.3/31
   ipv6 enable
   ipv6 address fe80::12/64 link-local

interface Ethernet2
   description Link to spine-2
   no switchport
   ip address 10.2.2.3/31
   ipv6 enable
   ipv6 address fe80::12/64 link-local

interface Ethernet3
   channel-group 10 mode active

interface Ethernet4
   shutdown
   channel-group 11 mode active

interface Ethernet5
   description Port-channel-1 Link to LEAF-1
   channel-group 1 mode active

interface Ethernet6
   description Port-channel-1 Link to LEAF-1
   channel-group 1 mode active

interface Loopback1
   description EVPN_VxLAN
   ip address 10.0.3.2/32

interface Loopback2
   description router-id for BGP
   ip address 10.1.0.2/32

interface Vlan100
   vrf OTUS
   ip address virtual 192.168.100.1/24
   ip virtual-router address 192.168.100.1
   mac address virtual-router

interface Vlan200
   vrf OTUS
   ip address virtual 192.168.200.1/24
   ip virtual-router address 192.168.200.1
   mac address virtual-router

interface Vlan4094
   description MLAG Peer Link
   ip address 10.3.0.2/30

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 100 vni 10010
   vxlan vlan 200 vni 10020
   vxlan vrf OTUS vni 333
   vxlan virtual-vtep local-interface Loopback1
   vxlan learn-restrict any

ip virtual-router mac-address 00:00:00:00:00:02

mlag configuration
   domain-id MLAG-OTUS
   local-interface Vlan4094
   peer-address 10.3.0.1
   peer-link Port-Channel1

```

- [leaf-3](Config/leaf-3.cfg)

```
vlan 100,200

vrf instance OTUS

interface Ethernet1
   description Link to spine-1
   no switchport
   ip address 10.2.1.5/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 enable
   ipv6 address fe80::13/64 link-local

interface Ethernet2
   description Link to spine-2
   no switchport
   ip address 10.2.2.5/31
   bfd interval 100 min-rx 100 multiplier 3
   ipv6 enable
   ipv6 address fe80::13/64 link-local

interface Ethernet3
   description Link to VPC_3
   switchport access vlan 100

interface Ethernet4
   description Link to VPC_4
   switchport access vlan 200

interface Loopback1
   description EVPN_VxLAN
   ip address 10.0.3.3/32

interface Loopback2
   description router-id for BGP
   ip address 10.1.0.3/32

interface Vlan100
   vrf OTUS
   ip address virtual 192.168.100.1/24

interface Vlan200
   vrf OTUS
   ip address virtual 192.168.200.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 100 vni 10010
   vxlan vlan 200 vni 10020
   vxlan vrf OTUS vni 333
   vxlan learn-restrict any

ip virtual-router mac-address 00:00:00:00:00:03

```

## Проверка связанности

- Проверка mlag на leaf-1 и leaf-2

```
leaf-1#show mlag
MLAG Configuration:
domain-id                          :           MLAG-OTUS
local-interface                    :            Vlan4094
peer-address                       :            10.3.0.2
peer-link                          :       Port-Channel1
peer-config                        :        inconsistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:03:37:66
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   1
Active-full                        :                   1

leaf-1#
```
```
leaf-2#show mlag
MLAG Configuration:
domain-id                          :           MLAG-OTUS
local-interface                    :            Vlan4094
peer-address                       :            10.3.0.1
peer-link                          :       Port-Channel1
peer-config                        :        inconsistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:03:37:66
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   1
Active-full                        :                   1

leaf-2#
```

- Проверка VxLAN

```
leaf-1#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.0.3.3       flood

Total number of remote VTEPS:  1
leaf-1#
```
```
leaf-2#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.0.3.3       flood

Total number of remote VTEPS:  1
leaf-2#
```
```
leaf-3#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.0.3.1       flood
10.0.3.2       flood

Total number of remote VTEPS:  2
leaf-3#
```

```
leaf-1#show ip route vrf OTUS

VRF: OTUS
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B E      192.168.100.13/32 [200/0] via VTEP 10.0.3.3 VNI 333 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        192.168.100.0/24 is directly connected, Vlan100
 B E      192.168.200.14/32 [200/0] via VTEP 10.0.3.3 VNI 333 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        192.168.200.0/24 is directly connected, Vlan200

leaf-1#
```
```
leaf-2#show ip route vrf OTUS

VRF: OTUS
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B E      192.168.100.13/32 [200/0] via VTEP 10.0.3.3 VNI 333 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        192.168.100.0/24 is directly connected, Vlan100
 B E      192.168.200.14/32 [200/0] via VTEP 10.0.3.3 VNI 333 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        192.168.200.0/24 is directly connected, Vlan200

leaf-2#
```
```
leaf-3#show ip route vrf OTUS

VRF: OTUS
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B E      192.168.100.11/32 [200/0] via VTEP 10.0.3.1 VNI 333 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 C        192.168.100.0/24 is directly connected, Vlan100
 B E      192.168.200.12/32 [200/0] via VTEP 10.0.3.1 VNI 333 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 C        192.168.200.0/24 is directly connected, Vlan200

leaf-3#
```
### Проверка связанности между хостами

- VPC_4

```
VPC_4> ping 192.168.100.11

192.168.100.11 icmp_seq=1 timeout
192.168.100.11 icmp_seq=2 timeout
84 bytes from 192.168.100.11 icmp_seq=3 ttl=62 time=168.986 ms
84 bytes from 192.168.100.11 icmp_seq=4 ttl=62 time=113.278 ms
84 bytes from 192.168.100.11 icmp_seq=5 ttl=62 time=146.431 ms

VPC_4> ping 192.168.200.11

84 bytes from 192.168.200.11 icmp_seq=1 ttl=64 time=126.162 ms
84 bytes from 192.168.200.11 icmp_seq=2 ttl=64 time=26.410 ms
84 bytes from 192.168.200.11 icmp_seq=3 ttl=64 time=19.071 ms
84 bytes from 192.168.200.11 icmp_seq=4 ttl=64 time=20.336 ms
84 bytes from 192.168.200.11 icmp_seq=5 ttl=64 time=22.821 ms

VPC_4> ping 192.168.200.12

84 bytes from 192.168.200.12 icmp_seq=1 ttl=64 time=583.500 ms
84 bytes from 192.168.200.12 icmp_seq=2 ttl=64 time=94.192 ms
84 bytes from 192.168.200.12 icmp_seq=3 ttl=64 time=32.995 ms
84 bytes from 192.168.200.12 icmp_seq=4 ttl=64 time=37.272 ms
84 bytes from 192.168.200.12 icmp_seq=5 ttl=64 time=60.454 ms

VPC_4> ping 192.168.100.12

84 bytes from 192.168.100.12 icmp_seq=1 ttl=62 time=121.696 ms
84 bytes from 192.168.100.12 icmp_seq=2 ttl=62 time=35.877 ms
84 bytes from 192.168.100.12 icmp_seq=3 ttl=62 time=28.093 ms
84 bytes from 192.168.100.12 icmp_seq=4 ttl=62 time=38.372 ms
84 bytes from 192.168.100.12 icmp_seq=5 ttl=62 time=91.212 ms

VPC_4>
```

- VPC_3

```
VPC_3> ping 192.168.100.11

84 bytes from 192.168.100.11 icmp_seq=1 ttl=64 time=99.278 ms
84 bytes from 192.168.100.11 icmp_seq=2 ttl=64 time=18.602 ms
84 bytes from 192.168.100.11 icmp_seq=3 ttl=64 time=20.829 ms
84 bytes from 192.168.100.11 icmp_seq=4 ttl=64 time=24.538 ms
84 bytes from 192.168.100.11 icmp_seq=5 ttl=64 time=21.451 ms

VPC_3> ping 192.168.100.12

84 bytes from 192.168.100.12 icmp_seq=1 ttl=64 time=24.744 ms
84 bytes from 192.168.100.12 icmp_seq=2 ttl=64 time=28.187 ms
84 bytes from 192.168.100.12 icmp_seq=3 ttl=64 time=24.489 ms
84 bytes from 192.168.100.12 icmp_seq=4 ttl=64 time=17.836 ms
84 bytes from 192.168.100.12 icmp_seq=5 ttl=64 time=36.738 ms

VPC_3> ping 192.168.200.12

84 bytes from 192.168.200.12 icmp_seq=1 ttl=63 time=135.970 ms
84 bytes from 192.168.200.12 icmp_seq=2 ttl=62 time=39.836 ms
84 bytes from 192.168.200.12 icmp_seq=3 ttl=62 time=75.037 ms
84 bytes from 192.168.200.12 icmp_seq=4 ttl=62 time=56.229 ms
84 bytes from 192.168.200.12 icmp_seq=5 ttl=62 time=49.094 ms

VPC_3> ping 192.168.100.12

84 bytes from 192.168.100.12 icmp_seq=1 ttl=64 time=27.595 ms
84 bytes from 192.168.100.12 icmp_seq=2 ttl=64 time=21.218 ms
84 bytes from 192.168.100.12 icmp_seq=3 ttl=64 time=17.442 ms
84 bytes from 192.168.100.12 icmp_seq=4 ttl=64 time=22.786 ms
84 bytes from 192.168.100.12 icmp_seq=5 ttl=64 time=22.296 ms

VPC_3>
```

- VPC_1

```
VPC_1> ping 192.168.100.12

84 bytes from 192.168.100.12 icmp_seq=1 ttl=64 time=196.472 ms
84 bytes from 192.168.100.12 icmp_seq=2 ttl=64 time=10.891 ms
84 bytes from 192.168.100.12 icmp_seq=3 ttl=64 time=15.744 ms
84 bytes from 192.168.100.12 icmp_seq=4 ttl=64 time=73.595 ms
84 bytes from 192.168.100.12 icmp_seq=5 ttl=64 time=6.259 ms

VPC_1> ping 192.168.200.12

84 bytes from 192.168.200.12 icmp_seq=1 ttl=63 time=30.702 ms
84 bytes from 192.168.200.12 icmp_seq=2 ttl=63 time=18.514 ms
84 bytes from 192.168.200.12 icmp_seq=3 ttl=63 time=27.257 ms
84 bytes from 192.168.200.12 icmp_seq=4 ttl=63 time=15.671 ms
84 bytes from 192.168.200.12 icmp_seq=5 ttl=63 time=21.253 ms

VPC_1> ping 192.168.100.13

84 bytes from 192.168.100.13 icmp_seq=1 ttl=64 time=422.196 ms
84 bytes from 192.168.100.13 icmp_seq=2 ttl=64 time=24.882 ms
84 bytes from 192.168.100.13 icmp_seq=3 ttl=64 time=23.768 ms
84 bytes from 192.168.100.13 icmp_seq=4 ttl=64 time=18.357 ms
84 bytes from 192.168.100.13 icmp_seq=5 ttl=64 time=24.070 ms

VPC_1> ping 192.168.200.14

84 bytes from 192.168.200.14 icmp_seq=1 ttl=63 time=275.138 ms
84 bytes from 192.168.200.14 icmp_seq=2 ttl=63 time=47.717 ms
84 bytes from 192.168.200.14 icmp_seq=3 ttl=63 time=31.902 ms
84 bytes from 192.168.200.14 icmp_seq=4 ttl=63 time=96.668 ms
84 bytes from 192.168.200.14 icmp_seq=5 ttl=63 time=86.243 ms

VPC_1>
```

- VPC_6

```
VPC_6> ping 192.168.100.11

84 bytes from 192.168.100.11 icmp_seq=1 ttl=63 time=63.539 ms
84 bytes from 192.168.100.11 icmp_seq=2 ttl=63 time=19.086 ms
84 bytes from 192.168.100.11 icmp_seq=3 ttl=63 time=26.893 ms
84 bytes from 192.168.100.11 icmp_seq=4 ttl=63 time=12.802 ms
84 bytes from 192.168.100.11 icmp_seq=5 ttl=63 time=17.417 ms

VPC_6> ping 192.168.200.11

84 bytes from 192.168.200.11 icmp_seq=1 ttl=64 time=66.075 ms
84 bytes from 192.168.200.11 icmp_seq=2 ttl=64 time=23.853 ms
84 bytes from 192.168.200.11 icmp_seq=3 ttl=64 time=7.171 ms
84 bytes from 192.168.200.11 icmp_seq=4 ttl=64 time=11.507 ms
84 bytes from 192.168.200.11 icmp_seq=5 ttl=64 time=80.379 ms

VPC_6> ping 192.168.200.14

84 bytes from 192.168.200.14 icmp_seq=1 ttl=64 time=22.130 ms
84 bytes from 192.168.200.14 icmp_seq=2 ttl=64 time=16.253 ms
84 bytes from 192.168.200.14 icmp_seq=3 ttl=64 time=24.133 ms
84 bytes from 192.168.200.14 icmp_seq=4 ttl=64 time=20.751 ms
84 bytes from 192.168.200.14 icmp_seq=5 ttl=64 time=24.026 ms

VPC_6> ping 192.168.100.13

84 bytes from 192.168.100.13 icmp_seq=1 ttl=62 time=50.251 ms
84 bytes from 192.168.100.13 icmp_seq=2 ttl=62 time=30.884 ms
84 bytes from 192.168.100.13 icmp_seq=3 ttl=62 time=89.824 ms
84 bytes from 192.168.100.13 icmp_seq=4 ttl=62 time=72.375 ms
84 bytes from 192.168.100.13 icmp_seq=5 ttl=62 time=109.100 ms

VPC_6>
```

### Проверка доступности после потери линка между Server-1 и leaf-1

```
leaf-1(config)#interface ethernet 3
leaf-1(config-if-Et3)#
leaf-1(config-if-Et3)#shut
leaf-1(config-if-Et3)#

```
- Проверка с VPC_4

```
VPC_4> ping 192.168.200.11

84 bytes from 192.168.200.11 icmp_seq=1 ttl=64 time=21.878 ms
84 bytes from 192.168.200.11 icmp_seq=2 ttl=64 time=45.423 ms
84 bytes from 192.168.200.11 icmp_seq=3 ttl=64 time=28.598 ms
84 bytes from 192.168.200.11 icmp_seq=4 ttl=64 time=26.567 ms
84 bytes from 192.168.200.11 icmp_seq=5 ttl=64 time=20.653 ms

VPC_4> ping 192.168.100.11

192.168.100.11 icmp_seq=1 timeout
84 bytes from 192.168.100.11 icmp_seq=2 ttl=63 time=22.410 ms
84 bytes from 192.168.100.11 icmp_seq=3 ttl=63 time=29.922 ms
84 bytes from 192.168.100.11 icmp_seq=4 ttl=63 time=25.474 ms
84 bytes from 192.168.100.11 icmp_seq=5 ttl=63 time=26.860 ms

VPC_4>
```
### Проверка доступности после падения линка между leaf-2 и Server-2

- Остались линки между leaf-1 - Server2 и leaf-2 - Server-1

```
leaf-2(config)#interface ethernet 4
leaf-2(config-if-Et4)#
leaf-2(config-if-Et4)#shutdown
leaf-2(config-if-Et4)#
```

- Проверка связанности

```
VPC_5> ping 192.168.200.14

84 bytes from 192.168.200.14 icmp_seq=1 ttl=63 time=34.357 ms
84 bytes from 192.168.200.14 icmp_seq=2 ttl=63 time=61.008 ms
84 bytes from 192.168.200.14 icmp_seq=3 ttl=63 time=37.442 ms
84 bytes from 192.168.200.14 icmp_seq=4 ttl=63 time=45.399 ms
84 bytes from 192.168.200.14 icmp_seq=5 ttl=63 time=62.800 ms

VPC_5> ping 192.168.100.13

84 bytes from 192.168.100.13 icmp_seq=1 ttl=64 time=21.811 ms
84 bytes from 192.168.100.13 icmp_seq=2 ttl=64 time=29.406 ms
84 bytes from 192.168.100.13 icmp_seq=3 ttl=64 time=32.396 ms
84 bytes from 192.168.100.13 icmp_seq=4 ttl=64 time=18.878 ms
84 bytes from 192.168.100.13 icmp_seq=5 ttl=64 time=25.371 ms

VPC_5> ping 192.168.100.11

84 bytes from 192.168.100.11 icmp_seq=1 ttl=64 time=46.788 ms
84 bytes from 192.168.100.11 icmp_seq=2 ttl=64 time=13.355 ms
84 bytes from 192.168.100.11 icmp_seq=3 ttl=64 time=12.995 ms
84 bytes from 192.168.100.11 icmp_seq=4 ttl=64 time=18.185 ms
84 bytes from 192.168.100.11 icmp_seq=5 ttl=64 time=19.167 ms

VPC_5> ping 192.168.200.11

84 bytes from 192.168.200.11 icmp_seq=1 ttl=63 time=259.863 ms
84 bytes from 192.168.200.11 icmp_seq=2 ttl=63 time=511.900 ms
84 bytes from 192.168.200.11 icmp_seq=3 ttl=63 time=17.274 ms
84 bytes from 192.168.200.11 icmp_seq=4 ttl=63 time=17.179 ms
84 bytes from 192.168.200.11 icmp_seq=5 ttl=63 time=18.313 ms

VPC_5>
```
```
VPC_3> ping 192.168.100.12

84 bytes from 192.168.100.12 icmp_seq=1 ttl=64 time=27.595 ms
84 bytes from 192.168.100.12 icmp_seq=2 ttl=64 time=21.218 ms
84 bytes from 192.168.100.12 icmp_seq=3 ttl=64 time=17.442 ms
84 bytes from 192.168.100.12 icmp_seq=4 ttl=64 time=22.786 ms
84 bytes from 192.168.100.12 icmp_seq=5 ttl=64 time=22.296 ms

VPC_3> ping 192.168.100.11

84 bytes from 192.168.100.11 icmp_seq=1 ttl=64 time=30.844 ms
84 bytes from 192.168.100.11 icmp_seq=2 ttl=64 time=21.470 ms
84 bytes from 192.168.100.11 icmp_seq=3 ttl=64 time=24.761 ms
84 bytes from 192.168.100.11 icmp_seq=4 ttl=64 time=30.948 ms
84 bytes from 192.168.100.11 icmp_seq=5 ttl=64 time=30.096 ms

VPC_3> ping 192.168.200.11

84 bytes from 192.168.200.11 icmp_seq=1 ttl=62 time=32.192 ms
84 bytes from 192.168.200.11 icmp_seq=2 ttl=62 time=31.601 ms
84 bytes from 192.168.200.11 icmp_seq=3 ttl=62 time=36.384 ms
84 bytes from 192.168.200.11 icmp_seq=4 ttl=62 time=35.285 ms
84 bytes from 192.168.200.11 icmp_seq=5 ttl=62 time=38.968 ms

VPC_3> ping 192.168.200.12

84 bytes from 192.168.200.12 icmp_seq=1 ttl=62 time=267.362 ms
84 bytes from 192.168.200.12 icmp_seq=2 ttl=62 time=32.531 ms
84 bytes from 192.168.200.12 icmp_seq=3 ttl=62 time=29.129 ms
84 bytes from 192.168.200.12 icmp_seq=4 ttl=62 time=67.953 ms
84 bytes from 192.168.200.12 icmp_seq=5 ttl=62 time=33.624 ms

VPC_3> ping 192.168.100.12

84 bytes from 192.168.100.12 icmp_seq=1 ttl=64 time=20.436 ms
84 bytes from 192.168.100.12 icmp_seq=2 ttl=64 time=22.803 ms
84 bytes from 192.168.100.12 icmp_seq=3 ttl=64 time=20.932 ms
84 bytes from 192.168.100.12 icmp_seq=4 ttl=64 time=19.919 ms
84 bytes from 192.168.100.12 icmp_seq=5 ttl=64 time=22.725 ms

VPC_3>
```

 Проверка прошла успешно. Теперь включаем лин между leaf-2 и Server-2 и выключаем leaf-1.
 
 - Проверка наличия vtep на leaf-3

```
leaf-3#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.0.3.2       flood

Total number of remote VTEPS:  1
leaf-3#
```
Как видно, leaf-1 не доступен

- Проверка ping с VPC_3

```
VPC_3> ping 192.168.100.12

84 bytes from 192.168.100.12 icmp_seq=1 ttl=64 time=218.651 ms
84 bytes from 192.168.100.12 icmp_seq=2 ttl=64 time=22.837 ms
84 bytes from 192.168.100.12 icmp_seq=3 ttl=64 time=25.374 ms
84 bytes from 192.168.100.12 icmp_seq=4 ttl=64 time=29.556 ms
84 bytes from 192.168.100.12 icmp_seq=5 ttl=64 time=19.154 ms

VPC_3> ping 192.168.200.12

84 bytes from 192.168.200.12 icmp_seq=1 ttl=62 time=228.218 ms
84 bytes from 192.168.200.12 icmp_seq=2 ttl=62 time=20.050 ms
84 bytes from 192.168.200.12 icmp_seq=3 ttl=62 time=19.098 ms
84 bytes from 192.168.200.12 icmp_seq=4 ttl=62 time=24.765 ms
84 bytes from 192.168.200.12 icmp_seq=5 ttl=62 time=23.887 ms

VPC_3> ping 192.168.200.11

84 bytes from 192.168.200.11 icmp_seq=1 ttl=62 time=202.418 ms
84 bytes from 192.168.200.11 icmp_seq=2 ttl=62 time=20.408 ms
84 bytes from 192.168.200.11 icmp_seq=3 ttl=62 time=31.841 ms
84 bytes from 192.168.200.11 icmp_seq=4 ttl=62 time=18.122 ms
84 bytes from 192.168.200.11 icmp_seq=5 ttl=62 time=31.634 ms

VPC_3> ping 192.168.100.11

192.168.100.11 icmp_seq=1 timeout
84 bytes from 192.168.100.11 icmp_seq=2 ttl=64 time=19.171 ms
84 bytes from 192.168.100.11 icmp_seq=3 ttl=64 time=21.032 ms
84 bytes from 192.168.100.11 icmp_seq=4 ttl=64 time=44.465 ms
84 bytes from 192.168.100.11 icmp_seq=5 ttl=64 time=18.028 ms

VPC_3>
```

В таблице маршрутизации так же можно видеть, что хосты стали доступны через leaf-2

```
leaf-3#show ip route vrf OTUS

VRF: OTUS
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B E      192.168.100.11/32 [200/0] via VTEP 10.0.3.2 VNI 333 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B E      192.168.100.12/32 [200/0] via VTEP 10.0.3.2 VNI 333 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 C        192.168.100.0/24 is directly connected, Vlan100
 B E      192.168.200.11/32 [200/0] via VTEP 10.0.3.2 VNI 333 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B E      192.168.200.12/32 [200/0] via VTEP 10.0.3.2 VNI 333 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 C        192.168.200.0/24 is directly connected, Vlan200

leaf-3#
```




