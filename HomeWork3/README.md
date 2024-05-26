# Домашнее задание №3
## Построение Underlay сети IS-IS

### Задание:
- Настроите IS-IS в Underlay сети, для IP связанности между всеми сетевыми устройствами;
- Убедитесь в наличии IP связанности между устройствами в IS-IS домене.

## Решение:

### Схема сети
![](img/SchemaISIS_area0001.png)

## Конфигурации:

- [spine-1](Config/spine-1.cfg)

```
router isis 0001
   net 49.0001.0100.0100.0002.00
   is-type level-1
   !
   address-family ipv4 unicast

interface Ethernet1
   description to-leaf-1
   no switchport
   ip address 10.2.1.0/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

interface Ethernet2
   description to-leaf-2
   no switchport
   ip address 10.2.1.2/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

interface Ethernet3
   description to-leaf-3
   no switchport
   ip address 10.2.1.4/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==
   
```

- [spine-2](Config/spine-2.cfg)

```
router isis 0001
   net 49.0001.0100.0000.1000.00
   is-type level-1
   !
   address-family ipv4 unicast

interface Ethernet1
   description to-leaf-1
   no switchport
   ip address 10.2.2.0/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

interface Ethernet2
   description to-leaf-2
   no switchport
   ip address 10.2.2.2/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

interface Ethernet3
   description to-leaf-3
   no switchport
   ip address 10.2.2.4/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

```

- [leaf-1](Config/leaf-1.cfg)

```
router isis 0001
   net 49.0001.0100.0000.1000.00
   is-type level-1
   !
   address-family ipv4 unicast

interface Ethernet1
   description to-spine-1
   no switchport
   ip address 10.2.1.1/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

interface Ethernet2
   description to-spine-2
   no switchport
   ip address 10.2.2.1/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

```

- [leaf-2](Config/leaf-2.cfg)

```

router isis 0001
   net 49.0001.0100.0000.1000.00
   is-type level-1
   !
   address-family ipv4 unicast

interface Ethernet1
   description to-spine-1
   no switchport
   ip address 10.2.1.3/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

interface Ethernet2
   description to-spine-2
   no switchport
   ip address 10.2.2.3/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

```

- [leaf-3](Config/leaf-3.cfg)

```
router isis 0001
   net 49.0001.0100.0000.1000.00
   is-type level-1
   !
   address-family ipv4 unicast

interface Ethernet1
   description to-spine-1
   no switchport
   ip address 10.2.1.5/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

interface Ethernet2
   description to-spine-2
   no switchport
   ip address 10.2.2.5/31
   bfd interval 100 min-rx 100 multiplier 3
   isis enable 0001
   isis bfd
   isis circuit-type level-1
   isis network point-to-point
   isis authentication key 7 kkH/BpnEzGmNpyc0bNYQKA==

```

## Таблицы маршрутизации:

- spine-1

```
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
 C        10.2.1.0/31 is directly connected, Ethernet1
 C        10.2.1.2/31 is directly connected, Ethernet2
 C        10.2.1.4/31 is directly connected, Ethernet3
 I L1     10.2.2.0/31 [115/20] via 10.2.1.1, Ethernet1
 I L1     10.2.2.2/31 [115/20] via 10.2.1.3, Ethernet2
 I L1     10.2.2.4/31 [115/20] via 10.2.1.5, Ethernet3

spine-1#

```

- spine-2

```
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
 I L1     10.2.1.0/31 [115/20] via 10.2.2.1, Ethernet1
 I L1     10.2.1.2/31 [115/20] via 10.2.2.3, Ethernet2
 I L1     10.2.1.4/31 [115/20] via 10.2.2.5, Ethernet3
 C        10.2.2.0/31 is directly connected, Ethernet1
 C        10.2.2.2/31 is directly connected, Ethernet2
 C        10.2.2.4/31 is directly connected, Ethernet3

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

 C        10.1.0.1/32 is directly connected, Loopback2
 C        10.2.1.0/31 is directly connected, Ethernet1
 I L1     10.2.1.2/31 [115/20] via 10.2.1.0, Ethernet1
 I L1     10.2.1.4/31 [115/20] via 10.2.1.0, Ethernet1
 C        10.2.2.0/31 is directly connected, Ethernet2
 I L1     10.2.2.2/31 [115/20] via 10.2.2.0, Ethernet2
 I L1     10.2.2.4/31 [115/20] via 10.2.2.0, Ethernet2

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

 C        10.1.0.2/32 is directly connected, Loopback2
 I L1     10.2.1.0/31 [115/20] via 10.2.1.2, Ethernet1
 C        10.2.1.2/31 is directly connected, Ethernet1
 I L1     10.2.1.4/31 [115/20] via 10.2.1.2, Ethernet1
 I L1     10.2.2.0/31 [115/20] via 10.2.2.2, Ethernet2
 C        10.2.2.2/31 is directly connected, Ethernet2
 I L1     10.2.2.4/31 [115/20] via 10.2.2.2, Ethernet2

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

 C        10.1.0.3/32 is directly connected, Loopback2
 I L1     10.2.1.0/31 [115/20] via 10.2.1.4, Ethernet1
 I L1     10.2.1.2/31 [115/20] via 10.2.1.4, Ethernet1
 C        10.2.1.4/31 is directly connected, Ethernet1
 I L1     10.2.2.0/31 [115/20] via 10.2.2.4, Ethernet2
 I L1     10.2.2.2/31 [115/20] via 10.2.2.4, Ethernet2
 C        10.2.2.4/31 is directly connected, Ethernet2

leaf-3#

```
## Проверка доступности устройств в area 0:

- leaf-1

```
leaf-1#ping 10.2.1.3
PING 10.2.1.3 (10.2.1.3) 72(100) bytes of data.
80 bytes from 10.2.1.3: icmp_seq=1 ttl=63 time=57.5 ms
80 bytes from 10.2.1.3: icmp_seq=2 ttl=63 time=53.9 ms
80 bytes from 10.2.1.3: icmp_seq=3 ttl=63 time=63.7 ms
80 bytes from 10.2.1.3: icmp_seq=4 ttl=63 time=76.2 ms
80 bytes from 10.2.1.3: icmp_seq=5 ttl=63 time=71.4 ms

--- 10.2.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 56ms
rtt min/avg/max/mdev = 53.924/64.586/76.295/8.350 ms, pipe 5, ipg/ewma 14.054/61.601 ms
leaf-1#
leaf-1#ping 10.2.1.5
PING 10.2.1.5 (10.2.1.5) 72(100) bytes of data.
80 bytes from 10.2.1.5: icmp_seq=1 ttl=63 time=79.9 ms
80 bytes from 10.2.1.5: icmp_seq=2 ttl=63 time=75.4 ms
80 bytes from 10.2.1.5: icmp_seq=3 ttl=63 time=80.9 ms
80 bytes from 10.2.1.5: icmp_seq=4 ttl=63 time=79.9 ms
80 bytes from 10.2.1.5: icmp_seq=5 ttl=63 time=73.0 ms

--- 10.2.1.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 51ms
rtt min/avg/max/mdev = 73.018/77.862/80.954/3.094 ms, pipe 5, ipg/ewma 12.865/78.793 ms
leaf-1#
leaf-1#ping 10.2.2.3
PING 10.2.2.3 (10.2.2.3) 72(100) bytes of data.
80 bytes from 10.2.2.3: icmp_seq=1 ttl=63 time=31.1 ms
80 bytes from 10.2.2.3: icmp_seq=2 ttl=63 time=33.0 ms
80 bytes from 10.2.2.3: icmp_seq=3 ttl=63 time=36.0 ms
80 bytes from 10.2.2.3: icmp_seq=4 ttl=63 time=31.8 ms

--- 10.2.2.3 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 109ms
rtt min/avg/max/mdev = 31.121/33.010/36.060/1.892 ms, pipe 3, ipg/ewma 27.488/31.933 ms
leaf-1#
leaf-1#ping 10.2.2.5
PING 10.2.2.5 (10.2.2.5) 72(100) bytes of data.
80 bytes from 10.2.2.5: icmp_seq=1 ttl=63 time=101 ms
80 bytes from 10.2.2.5: icmp_seq=2 ttl=63 time=100 ms
80 bytes from 10.2.2.5: icmp_seq=3 ttl=63 time=107 ms
80 bytes from 10.2.2.5: icmp_seq=4 ttl=63 time=111 ms
80 bytes from 10.2.2.5: icmp_seq=5 ttl=63 time=116 ms

--- 10.2.2.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 59ms
rtt min/avg/max/mdev = 100.541/107.522/116.463/6.008 ms, pipe 5, ipg/ewma 14.837/105.036 ms
leaf-1#

```

- leaf-2

```
leaf-2#ping 10.2.1.1
PING 10.2.1.1 (10.2.1.1) 72(100) bytes of data.
80 bytes from 10.2.1.1: icmp_seq=1 ttl=63 time=36.8 ms
80 bytes from 10.2.1.1: icmp_seq=2 ttl=63 time=49.7 ms
80 bytes from 10.2.1.1: icmp_seq=3 ttl=63 time=46.4 ms
80 bytes from 10.2.1.1: icmp_seq=4 ttl=63 time=45.8 ms
80 bytes from 10.2.1.1: icmp_seq=5 ttl=63 time=28.1 ms

--- 10.2.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 106ms
rtt min/avg/max/mdev = 28.119/41.382/49.716/7.895 ms, pipe 3, ipg/ewma 26.534/38.715 ms
leaf-2#
leaf-2#ping 10.2.2.1
PING 10.2.2.1 (10.2.2.1) 72(100) bytes of data.
80 bytes from 10.2.2.1: icmp_seq=1 ttl=63 time=141 ms
80 bytes from 10.2.2.1: icmp_seq=2 ttl=63 time=169 ms
80 bytes from 10.2.2.1: icmp_seq=3 ttl=63 time=174 ms
80 bytes from 10.2.2.1: icmp_seq=4 ttl=63 time=181 ms
80 bytes from 10.2.2.1: icmp_seq=5 ttl=63 time=179 ms

--- 10.2.2.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 50ms
rtt min/avg/max/mdev = 141.626/169.253/181.930/14.487 ms, pipe 5, ipg/ewma 12.556/156.169 ms
leaf-2#
leaf-2#ping 10.2.1.5
PING 10.2.1.5 (10.2.1.5) 72(100) bytes of data.
80 bytes from 10.2.1.5: icmp_seq=1 ttl=63 time=83.8 ms
80 bytes from 10.2.1.5: icmp_seq=2 ttl=63 time=74.2 ms
80 bytes from 10.2.1.5: icmp_seq=3 ttl=63 time=80.1 ms
80 bytes from 10.2.1.5: icmp_seq=4 ttl=63 time=79.4 ms
80 bytes from 10.2.1.5: icmp_seq=5 ttl=63 time=79.1 ms

--- 10.2.1.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 50ms
rtt min/avg/max/mdev = 74.244/79.385/83.888/3.100 ms, pipe 5, ipg/ewma 12.721/81.649 ms
leaf-2#
leaf-2#ping 10.2.2.5
PING 10.2.2.5 (10.2.2.5) 72(100) bytes of data.
80 bytes from 10.2.2.5: icmp_seq=1 ttl=63 time=28.0 ms
80 bytes from 10.2.2.5: icmp_seq=2 ttl=63 time=30.4 ms
80 bytes from 10.2.2.5: icmp_seq=3 ttl=63 time=36.5 ms
80 bytes from 10.2.2.5: icmp_seq=4 ttl=63 time=50.1 ms
80 bytes from 10.2.2.5: icmp_seq=5 ttl=63 time=37.3 ms

--- 10.2.2.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 89ms
rtt min/avg/max/mdev = 28.060/36.504/50.167/7.685 ms, pipe 3, ipg/ewma 22.293/32.645 ms
leaf-2#

```

- leaf-3

```
leaf-3#ping 10.2.1.1
PING 10.2.1.1 (10.2.1.1) 72(100) bytes of data.
80 bytes from 10.2.1.1: icmp_seq=1 ttl=63 time=41.6 ms
80 bytes from 10.2.1.1: icmp_seq=2 ttl=63 time=53.0 ms
80 bytes from 10.2.1.1: icmp_seq=3 ttl=63 time=51.6 ms
80 bytes from 10.2.1.1: icmp_seq=4 ttl=63 time=58.2 ms
80 bytes from 10.2.1.1: icmp_seq=5 ttl=63 time=29.3 ms

--- 10.2.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 101ms
rtt min/avg/max/mdev = 29.311/46.792/58.283/10.271 ms, pipe 4, ipg/ewma 25.498/43.830 ms
leaf-3#
leaf-3#ping 10.2.2.1
PING 10.2.2.1 (10.2.2.1) 72(100) bytes of data.
80 bytes from 10.2.2.1: icmp_seq=1 ttl=63 time=39.2 ms
80 bytes from 10.2.2.1: icmp_seq=2 ttl=63 time=39.4 ms
80 bytes from 10.2.2.1: icmp_seq=3 ttl=63 time=38.8 ms
80 bytes from 10.2.2.1: icmp_seq=4 ttl=63 time=98.4 ms
80 bytes from 10.2.2.1: icmp_seq=5 ttl=63 time=71.2 ms

--- 10.2.2.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 108ms
rtt min/avg/max/mdev = 38.869/57.467/98.499/23.971 ms, pipe 3, ipg/ewma 27.209/49.734 ms
leaf-3#
leaf-3#ping 10.2.1.3
PING 10.2.1.3 (10.2.1.3) 72(100) bytes of data.
80 bytes from 10.2.1.3: icmp_seq=1 ttl=63 time=30.6 ms
80 bytes from 10.2.1.3: icmp_seq=2 ttl=63 time=32.7 ms
80 bytes from 10.2.1.3: icmp_seq=3 ttl=63 time=28.2 ms
80 bytes from 10.2.1.3: icmp_seq=4 ttl=63 time=33.6 ms
80 bytes from 10.2.1.3: icmp_seq=5 ttl=63 time=37.4 ms

--- 10.2.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 93ms
rtt min/avg/max/mdev = 28.296/32.560/37.499/3.072 ms, pipe 3, ipg/ewma 23.412/31.799 ms
leaf-3#
leaf-3#ping 10.2.2.3
PING 10.2.2.3 (10.2.2.3) 72(100) bytes of data.
80 bytes from 10.2.2.3: icmp_seq=1 ttl=63 time=55.9 ms
80 bytes from 10.2.2.3: icmp_seq=2 ttl=63 time=56.3 ms
80 bytes from 10.2.2.3: icmp_seq=3 ttl=63 time=54.0 ms
80 bytes from 10.2.2.3: icmp_seq=4 ttl=63 time=53.0 ms
80 bytes from 10.2.2.3: icmp_seq=5 ttl=63 time=50.4 ms

--- 10.2.2.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 50.482/53.983/56.348/2.129 ms, pipe 5, ipg/ewma 11.661/54.823 ms
leaf-3#

```

- spine-1

```
spine-1#ping 10.2.2.0
PING 10.2.2.0 (10.2.2.0) 72(100) bytes of data.
80 bytes from 10.2.2.0: icmp_seq=1 ttl=63 time=25.2 ms
80 bytes from 10.2.2.0: icmp_seq=2 ttl=63 time=33.0 ms
80 bytes from 10.2.2.0: icmp_seq=3 ttl=63 time=41.1 ms
80 bytes from 10.2.2.0: icmp_seq=4 ttl=63 time=23.7 ms
80 bytes from 10.2.2.0: icmp_seq=5 ttl=63 time=24.9 ms

--- 10.2.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 99ms
rtt min/avg/max/mdev = 23.797/29.631/41.142/6.619 ms, pipe 2, ipg/ewma 24.947/27.224 ms
spine-1#
spine-1#ping 10.2.2.2
PING 10.2.2.2 (10.2.2.2) 72(100) bytes of data.
80 bytes from 10.2.2.2: icmp_seq=1 ttl=63 time=46.4 ms
80 bytes from 10.2.2.2: icmp_seq=2 ttl=63 time=43.7 ms
80 bytes from 10.2.2.2: icmp_seq=3 ttl=63 time=62.7 ms
80 bytes from 10.2.2.2: icmp_seq=4 ttl=63 time=60.7 ms
80 bytes from 10.2.2.2: icmp_seq=5 ttl=63 time=82.8 ms

--- 10.2.2.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44ms
rtt min/avg/max/mdev = 43.702/59.288/82.848/13.976 ms, pipe 5, ipg/ewma 11.110/53.887 ms
spine-1#
spine-1#ping 10.2.2.4
PING 10.2.2.4 (10.2.2.4) 72(100) bytes of data.
80 bytes from 10.2.2.4: icmp_seq=1 ttl=63 time=81.2 ms
80 bytes from 10.2.2.4: icmp_seq=2 ttl=63 time=80.3 ms
80 bytes from 10.2.2.4: icmp_seq=3 ttl=63 time=81.8 ms
80 bytes from 10.2.2.4: icmp_seq=4 ttl=63 time=80.8 ms
80 bytes from 10.2.2.4: icmp_seq=5 ttl=63 time=79.7 ms

--- 10.2.2.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 79.750/80.818/81.803/0.795 ms, pipe 5, ipg/ewma 11.544/81.019 ms
spine-1#

```

- spine-2

```
spine-2#ping 10.2.1.0
PING 10.2.1.0 (10.2.1.0) 72(100) bytes of data.
80 bytes from 10.2.1.0: icmp_seq=1 ttl=63 time=48.8 ms
80 bytes from 10.2.1.0: icmp_seq=2 ttl=63 time=51.8 ms
80 bytes from 10.2.1.0: icmp_seq=3 ttl=63 time=51.1 ms
80 bytes from 10.2.1.0: icmp_seq=4 ttl=63 time=51.7 ms
80 bytes from 10.2.1.0: icmp_seq=5 ttl=63 time=45.2 ms

--- 10.2.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 45ms
rtt min/avg/max/mdev = 45.266/49.748/51.806/2.494 ms, pipe 5, ipg/ewma 11.426/49.163 ms
spine-2#
spine-2#ping 10.2.1.2
PING 10.2.1.2 (10.2.1.2) 72(100) bytes of data.
80 bytes from 10.2.1.2: icmp_seq=1 ttl=63 time=34.1 ms
80 bytes from 10.2.1.2: icmp_seq=2 ttl=63 time=32.1 ms
80 bytes from 10.2.1.2: icmp_seq=3 ttl=63 time=37.4 ms
80 bytes from 10.2.1.2: icmp_seq=4 ttl=63 time=29.8 ms
80 bytes from 10.2.1.2: icmp_seq=5 ttl=63 time=39.8 ms

--- 10.2.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 97ms
rtt min/avg/max/mdev = 29.850/34.690/39.881/3.592 ms, pipe 3, ipg/ewma 24.406/34.528 ms
spine-2#
spine-2#ping 10.2.1.4
PING 10.2.1.4 (10.2.1.4) 72(100) bytes of data.
80 bytes from 10.2.1.4: icmp_seq=1 ttl=63 time=56.2 ms
80 bytes from 10.2.1.4: icmp_seq=2 ttl=63 time=46.7 ms
80 bytes from 10.2.1.4: icmp_seq=3 ttl=63 time=54.1 ms
80 bytes from 10.2.1.4: icmp_seq=4 ttl=63 time=68.1 ms
80 bytes from 10.2.1.4: icmp_seq=5 ttl=63 time=76.4 ms

--- 10.2.1.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 46.757/60.350/76.467/10.603 ms, pipe 5, ipg/ewma 11.748/59.065 ms
spine-2#

```

