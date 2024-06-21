# Домашнее задание №6
## Построение Overlay сети VxLAN L3

### Задание:
- Настроите каждого клиента в своем VNI;
- Настроите маршрутизацию между клиентами.

## Решение:

### Схема сети
![](img/Schema_VxLAN_L3.png)

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
vlan 100

vrf instance OTUS

interface Loopback1
   description EVPN_VxLAN
   ip address 10.0.3.1/32

interface Loopback2
   description router-id for BGP
   ip address 10.1.0.1/32

interface Vlan100
   vrf OTUS
   ip address 192.168.100.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 100 vni 10010
   vxlan vrf OTUS vni 333
   vxlan learn-restrict any

ip prefix-list BGP_out seq 10 permit 10.0.3.1/32
ip prefix-list BGP_out seq 20 permit 10.1.0.1/32

route-map BGP_out_map permit 10
   match ip address prefix-list BGP_out

router bgp 65001
   router-id 10.1.0.1
   bgp default ipv4-unicast transport ipv6
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES bfd
   neighbor SPINES auto-local-addr
   neighbor SPINES password 7 P1GCpTlgo6U9in80t4X0Rw==
   neighbor SPINES send-community
   neighbor 10.0.1.0 peer group EVPN
   neighbor 10.0.2.0 peer group EVPN
   neighbor fe80::1%Et1 peer group SPINES
   neighbor fe80::2%Et2 peer group SPINES
   redistribute connected route-map BGP_out_map
   
   vlan 100
      rd 65001:10010
      route-target both 10:10010
      redistribute learned
   
   address-family evpn
      neighbor EVPN activate
   
   address-family ipv4
      neighbor SPINES activate
   
   address-family ipv6
      neighbor SPINES activate
   
   vrf OTUS
      rd 65001:333
      route-target import evpn 33:333
      route-target export evpn 33:333

```
- [leaf-2](Config/leaf-2.cfg)

```
vlan 200

vrf instance OTUS

interface Loopback1
   description EVPN_VxLAN
   ip address 10.0.3.2/32

interface Loopback2
   description router-id for BGP
   ip address 10.1.0.2/32

interface Vlan200
   vrf OTUS
   ip address 192.168.200.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 200 vni 10020
   vxlan vrf OTUS vni 333
   vxlan learn-restrict any

ip routing
ip routing vrf OTUS

ip prefix-list BGP_out seq 10 permit 10.0.3.2/32
ip prefix-list BGP_out seq 20 permit 10.1.0.2/32

route-map BGP_out_map permit 10
   match ip address prefix-list BGP_out

router bgp 65002
   router-id 10.1.0.2
   bgp default ipv4-unicast transport ipv6
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES bfd
   neighbor SPINES auto-local-addr
   neighbor SPINES password 7 P1GCpTlgo6U9in80t4X0Rw==
   neighbor SPINES send-community
   neighbor 10.0.1.0 peer group EVPN
   neighbor 10.0.2.0 peer group EVPN
   neighbor fe80::1%Et1 peer group SPINES
   neighbor fe80::2%Et2 peer group SPINES
   redistribute connected route-map BGP_out_map
   
   vlan 200
      rd 65002:10020
      route-target both 20:10020
      redistribute learned
   
   address-family evpn
      neighbor EVPN activate
   
   address-family ipv4
      neighbor SPINES activate
   
   address-family ipv6
      neighbor SPINES activate
   
   vrf OTUS
      rd 65002:333
      route-target import evpn 33:333
      route-target export evpn 33:333

```
- [leaf-3](Config/leaf-3.cfg)

```
vlan 110,120

vrf instance OTUS

interface Loopback1
   description EVPN_VxLAN
   ip address 10.0.3.3/32

interface Loopback2
   description router-id for BGP
   ip address 10.1.0.3/32

interface Vlan110
   vrf OTUS
   ip address 192.168.100.1/24

interface Vlan120
   vrf OTUS
   ip address 192.168.200.1/24

interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 110 vni 10010
   vxlan vlan 120 vni 10020
   vxlan vrf OTUS vni 333
   vxlan learn-restrict any

ip prefix-list BGP_out seq 10 permit 10.0.3.3/32
ip prefix-list BGP_out seq 20 permit 10.1.0.3/32

route-map BGP_out_map permit 10
   match ip address prefix-list BGP_out

router bgp 65003
   router-id 10.1.0.3
   bgp default ipv4-unicast transport ipv6
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor EVPN peer group
   neighbor EVPN remote-as 65000
   neighbor EVPN update-source Loopback1
   neighbor EVPN ebgp-multihop 3
   neighbor EVPN send-community extended
   neighbor SPINES peer group
   neighbor SPINES remote-as 65000
   neighbor SPINES bfd
   neighbor SPINES auto-local-addr
   neighbor SPINES password 7 P1GCpTlgo6U9in80t4X0Rw==
   neighbor SPINES send-community
   neighbor 10.0.1.0 peer group EVPN
   neighbor 10.0.2.0 peer group EVPN
   neighbor fe80::1%Et1 peer group SPINES
   neighbor fe80::2%Et2 peer group SPINES
   redistribute connected route-map BGP_out_map
   
   vlan 110
      rd 65003:10010
      route-target both 10:10010
      redistribute learned
   
   vlan 120
      rd 65003:10020
      route-target both 20:10020
      redistribute learned
   
   address-family evpn
      neighbor EVPN activate
   
   address-family ipv4
      neighbor SPINES activate
   
   address-family ipv6
      neighbor SPINES activate
   
   vrf OTUS
      rd 65003:333
      route-target import evpn 33:333
      route-target export evpn 33:333

```

## Проверка

- spine-1

```
spine-1#
spine-1#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.1.0, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10010 imet 10.0.3.1
                                 10.0.3.1              -       100     0       65001 i
 * >      RD: 65002:10020 imet 10.0.3.2
                                 10.0.3.2              -       100     0       65002 i
 * >      RD: 65003:10010 imet 10.0.3.3
                                 10.0.3.3              -       100     0       65003 i
 * >      RD: 65003:10020 imet 10.0.3.3
                                 10.0.3.3              -       100     0       65003 i
spine-1#
spine-1#show ip route

VRF: default
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

 C        10.0.1.0/32 is directly connected, Loopback1
 B E      10.0.3.1/32 [200/0] via 10.2.1.1, Ethernet1
 B E      10.0.3.2/32 [200/0] via 10.2.1.3, Ethernet2
 B E      10.0.3.3/32 [200/0] via 10.2.1.5, Ethernet3
 B E      10.1.0.1/32 [200/0] via 10.2.1.1, Ethernet1
 B E      10.1.0.2/32 [200/0] via 10.2.1.3, Ethernet2
 B E      10.1.0.3/32 [200/0] via 10.2.1.5, Ethernet3
 C        10.2.1.0/31 is directly connected, Ethernet1
 C        10.2.1.2/31 is directly connected, Ethernet2
 C        10.2.1.4/31 is directly connected, Ethernet3

spine-1#
spine-1#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.0.1.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.3.1 4 65001            107        96    0    0 00:02:50 Estab   1      1
  10.0.3.2 4 65002           2160      2134    0    0 01:23:06 Estab   1      1
  10.0.3.3 4 65003           2151      2063    0    0 01:23:22 Estab   2      2
spine-1#

```
- spine-2

```
spine-2#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.2.0, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10010 imet 10.0.3.1
                                 10.0.3.1              -       100     0       65001 i
 * >      RD: 65002:10020 imet 10.0.3.2
                                 10.0.3.2              -       100     0       65002 i
 * >      RD: 65003:10010 imet 10.0.3.3
                                 10.0.3.3              -       100     0       65003 i
 * >      RD: 65003:10020 imet 10.0.3.3
                                 10.0.3.3              -       100     0       65003 i
spine-2#
spine-2#show ip route

VRF: default
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

 C        10.0.2.0/32 is directly connected, Loopback1
 B E      10.0.3.1/32 [200/0] via 10.2.2.1, Ethernet1
 B E      10.0.3.2/32 [200/0] via 10.2.2.3, Ethernet2
 B E      10.0.3.3/32 [200/0] via 10.2.2.5, Ethernet3
 B E      10.1.0.1/32 [200/0] via 10.2.2.1, Ethernet1
 B E      10.1.0.2/32 [200/0] via 10.2.2.3, Ethernet2
 B E      10.1.0.3/32 [200/0] via 10.2.2.5, Ethernet3
 C        10.2.2.0/31 is directly connected, Ethernet1
 C        10.2.2.2/31 is directly connected, Ethernet2
 C        10.2.2.4/31 is directly connected, Ethernet3

spine-2#
spine-2#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.0.2.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.3.1 4 65001            102       101    0    0 00:03:50 Estab   1      1
  10.0.3.2 4 65002            136       135    0    0 00:04:24 Estab   1      1
  10.0.3.3 4 65003            105       101    0    0 00:03:59 Estab   2      2
spine-2#

```
- leaf-1

```
leaf-1#show ip route

VRF: default
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

 B E      10.0.1.0/32 [200/0] via 10.2.1.0, Ethernet1
 B E      10.0.2.0/32 [200/0] via 10.2.2.0, Ethernet2
 C        10.0.3.1/32 is directly connected, Loopback1
 B E      10.0.3.2/32 [200/0] via 10.2.1.0, Ethernet1
                              via 10.2.2.0, Ethernet2
 B E      10.0.3.3/32 [200/0] via 10.2.1.0, Ethernet1
                              via 10.2.2.0, Ethernet2
 C        10.1.0.1/32 is directly connected, Loopback2
 B E      10.1.0.2/32 [200/0] via 10.2.1.0, Ethernet1
                              via 10.2.2.0, Ethernet2
 B E      10.1.0.3/32 [200/0] via 10.2.1.0, Ethernet1
                              via 10.2.2.0, Ethernet2
 C        10.2.1.0/31 is directly connected, Ethernet1
 C        10.2.2.0/31 is directly connected, Ethernet2

leaf-1#
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
 B E      192.168.200.11/32 [200/0] via VTEP 10.0.3.2 VNI 333 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B E      192.168.200.14/32 [200/0] via VTEP 10.0.3.3 VNI 333 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1

leaf-1#
leaf-1#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.0.3.2       unicast
10.0.3.3       flood, unicast

Total number of remote VTEPS:  2
leaf-1#

```
- leaf-2

```
leaf-2#show ip route

VRF: default
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

 B E      10.0.1.0/32 [200/0] via 10.2.1.2, Ethernet1
 B E      10.0.2.0/32 [200/0] via 10.2.2.2, Ethernet2
 B E      10.0.3.1/32 [200/0] via 10.2.1.2, Ethernet1
                              via 10.2.2.2, Ethernet2
 C        10.0.3.2/32 is directly connected, Loopback1
 B E      10.0.3.3/32 [200/0] via 10.2.1.2, Ethernet1
                              via 10.2.2.2, Ethernet2
 B E      10.1.0.1/32 [200/0] via 10.2.1.2, Ethernet1
                              via 10.2.2.2, Ethernet2
 C        10.1.0.2/32 is directly connected, Loopback2
 B E      10.1.0.3/32 [200/0] via 10.2.1.2, Ethernet1
                              via 10.2.2.2, Ethernet2
 C        10.2.1.2/31 is directly connected, Ethernet1
 C        10.2.2.2/31 is directly connected, Ethernet2

leaf-2#
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

 B E      192.168.100.11/32 [200/0] via VTEP 10.0.3.1 VNI 333 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      192.168.100.13/32 [200/0] via VTEP 10.0.3.3 VNI 333 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 B E      192.168.200.14/32 [200/0] via VTEP 10.0.3.3 VNI 333 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        192.168.200.0/24 is directly connected, Vlan200

leaf-2#
leaf-2#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.0.3.1       unicast
10.0.3.3       unicast, flood

Total number of remote VTEPS:  2
leaf-2#

```
- leaf-3

```
leaf-3#show ip route

VRF: default
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

 B E      10.0.1.0/32 [200/0] via 10.2.1.4, Ethernet1
 B E      10.0.2.0/32 [200/0] via 10.2.2.4, Ethernet2
 B E      10.0.3.1/32 [200/0] via 10.2.1.4, Ethernet1
                              via 10.2.2.4, Ethernet2
 B E      10.0.3.2/32 [200/0] via 10.2.1.4, Ethernet1
                              via 10.2.2.4, Ethernet2
 C        10.0.3.3/32 is directly connected, Loopback1
 B E      10.1.0.1/32 [200/0] via 10.2.1.4, Ethernet1
                              via 10.2.2.4, Ethernet2
 B E      10.1.0.2/32 [200/0] via 10.2.1.4, Ethernet1
                              via 10.2.2.4, Ethernet2
 C        10.1.0.3/32 is directly connected, Loopback2
 C        10.2.1.4/31 is directly connected, Ethernet1
 C        10.2.2.4/31 is directly connected, Ethernet2

leaf-3#
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
 C        192.168.100.0/24 is directly connected, Vlan110
 B E      192.168.200.11/32 [200/0] via VTEP 10.0.3.2 VNI 333 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 C        192.168.200.0/24 is directly connected, Vlan120

leaf-3#
leaf-3#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP           Tunnel Type(s)
-------------- --------------
10.0.3.1       flood, unicast
10.0.3.2       flood, unicast

Total number of remote VTEPS:  2
leaf-3#

```
- Связанность между хостами

```
VPC_1> ping 192.168.100.13

84 bytes from 192.168.100.13 icmp_seq=1 ttl=64 time=24.197 ms
84 bytes from 192.168.100.13 icmp_seq=2 ttl=64 time=15.970 ms
84 bytes from 192.168.100.13 icmp_seq=3 ttl=64 time=26.795 ms
84 bytes from 192.168.100.13 icmp_seq=4 ttl=64 time=22.776 ms
84 bytes from 192.168.100.13 icmp_seq=5 ttl=64 time=79.386 ms

VPC_1> ping 192.168.200.11

84 bytes from 192.168.200.11 icmp_seq=1 ttl=62 time=30.569 ms
84 bytes from 192.168.200.11 icmp_seq=2 ttl=62 time=18.723 ms
84 bytes from 192.168.200.11 icmp_seq=3 ttl=62 time=20.399 ms
84 bytes from 192.168.200.11 icmp_seq=4 ttl=62 time=30.375 ms
84 bytes from 192.168.200.11 icmp_seq=5 ttl=62 time=53.237 ms

VPC_1> ping 192.168.200.14

84 bytes from 192.168.200.14 icmp_seq=1 ttl=62 time=23.982 ms
84 bytes from 192.168.200.14 icmp_seq=2 ttl=62 time=20.396 ms
84 bytes from 192.168.200.14 icmp_seq=3 ttl=62 time=22.334 ms
84 bytes from 192.168.200.14 icmp_seq=4 ttl=62 time=23.140 ms
84 bytes from 192.168.200.14 icmp_seq=5 ttl=62 time=27.502 ms

VPC_1>
```

```
VPC_2> ping 192.168.100.11

84 bytes from 192.168.100.11 icmp_seq=1 ttl=62 time=36.348 ms
84 bytes from 192.168.100.11 icmp_seq=2 ttl=62 time=26.633 ms
84 bytes from 192.168.100.11 icmp_seq=3 ttl=62 time=22.058 ms
84 bytes from 192.168.100.11 icmp_seq=4 ttl=62 time=20.507 ms
84 bytes from 192.168.100.11 icmp_seq=5 ttl=62 time=24.109 ms

VPC_2> ping 192.168.100.13

84 bytes from 192.168.100.13 icmp_seq=1 ttl=62 time=44.711 ms
84 bytes from 192.168.100.13 icmp_seq=2 ttl=62 time=26.374 ms
84 bytes from 192.168.100.13 icmp_seq=3 ttl=62 time=49.446 ms
84 bytes from 192.168.100.13 icmp_seq=4 ttl=62 time=19.130 ms
84 bytes from 192.168.100.13 icmp_seq=5 ttl=62 time=19.495 ms

VPC_2> ping 192.168.200.14

84 bytes from 192.168.200.14 icmp_seq=1 ttl=64 time=26.621 ms
84 bytes from 192.168.200.14 icmp_seq=2 ttl=64 time=14.094 ms
84 bytes from 192.168.200.14 icmp_seq=3 ttl=64 time=23.672 ms
84 bytes from 192.168.200.14 icmp_seq=4 ttl=64 time=16.402 ms
84 bytes from 192.168.200.14 icmp_seq=5 ttl=64 time=17.309 ms

VPC_2>
```

```
VPC_3> ping 192.168.200.14

84 bytes from 192.168.200.14 icmp_seq=1 ttl=63 time=17.861 ms
84 bytes from 192.168.200.14 icmp_seq=2 ttl=63 time=7.533 ms
84 bytes from 192.168.200.14 icmp_seq=3 ttl=63 time=6.612 ms
84 bytes from 192.168.200.14 icmp_seq=4 ttl=63 time=8.139 ms
84 bytes from 192.168.200.14 icmp_seq=5 ttl=63 time=6.541 ms

VPC_3> ping 192.168.200.11

84 bytes from 192.168.200.11 icmp_seq=1 ttl=62 time=26.360 ms
84 bytes from 192.168.200.11 icmp_seq=2 ttl=62 time=36.362 ms
84 bytes from 192.168.200.11 icmp_seq=3 ttl=62 time=38.038 ms
84 bytes from 192.168.200.11 icmp_seq=4 ttl=62 time=18.509 ms
84 bytes from 192.168.200.11 icmp_seq=5 ttl=62 time=107.824 ms

VPC_3> ping 192.168.100.11

84 bytes from 192.168.100.11 icmp_seq=1 ttl=64 time=45.311 ms
84 bytes from 192.168.100.11 icmp_seq=2 ttl=64 time=19.536 ms
84 bytes from 192.168.100.11 icmp_seq=3 ttl=64 time=14.796 ms
84 bytes from 192.168.100.11 icmp_seq=4 ttl=64 time=15.515 ms
84 bytes from 192.168.100.11 icmp_seq=5 ttl=64 time=22.878 ms

VPC_3>
```

```
VPC_4> ping 192.168.100.13

84 bytes from 192.168.100.13 icmp_seq=1 ttl=63 time=25.929 ms
84 bytes from 192.168.100.13 icmp_seq=2 ttl=63 time=11.567 ms
84 bytes from 192.168.100.13 icmp_seq=3 ttl=63 time=8.973 ms
84 bytes from 192.168.100.13 icmp_seq=4 ttl=63 time=6.771 ms
84 bytes from 192.168.100.13 icmp_seq=5 ttl=63 time=6.208 ms

VPC_4> ping 192.168.100.11

84 bytes from 192.168.100.11 icmp_seq=1 ttl=62 time=30.749 ms
84 bytes from 192.168.100.11 icmp_seq=2 ttl=62 time=25.787 ms
84 bytes from 192.168.100.11 icmp_seq=3 ttl=62 time=24.513 ms
84 bytes from 192.168.100.11 icmp_seq=4 ttl=62 time=17.458 ms
84 bytes from 192.168.100.11 icmp_seq=5 ttl=62 time=17.998 ms

VPC_4> ping 192.168.200.11

84 bytes from 192.168.200.11 icmp_seq=1 ttl=64 time=22.226 ms
84 bytes from 192.168.200.11 icmp_seq=2 ttl=64 time=35.130 ms
84 bytes from 192.168.200.11 icmp_seq=3 ttl=64 time=32.428 ms
84 bytes from 192.168.200.11 icmp_seq=4 ttl=64 time=15.353 ms
84 bytes from 192.168.200.11 icmp_seq=5 ttl=64 time=18.681 ms

VPC_4>
```

