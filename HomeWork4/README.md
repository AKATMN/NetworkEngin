# Домашнее задание №4
## Построение Underlay сети eBGP

### Задание:
- Настроите BGP в Underlay сети, для IP связанности между всеми сетевыми устройствами;
- Убедитесь в наличии IP связанности между устройствами в BGP домене.

## Решение:

### Схема сети
![](img/Schema_eBGP.png)

## Конфигурации:

- [spine-1](Config/spine-1.cfg)

```
interface Ethernet1
   description to-leaf-1
   no switchport
   ip address 10.2.1.0/31
   ipv6 enable
   ipv6 address fe80::1/64 link-local

interface Ethernet2
   description to-leaf-2
   no switchport
   ip address 10.2.1.2/31
   ipv6 enable
   ipv6 address fe80::1/64 link-local

interface Ethernet3
   description to-leaf-3
   no switchport
   ip address 10.2.1.4/31
   ipv6 enable
   ipv6 address fe80::1/64 link-local

router bgp 65000
   router-id 10.0.1.0
   bgp default ipv4-unicast transport ipv6
   neighbor fe80::11%Et1 remote-as 65001
   neighbor fe80::11%Et1 bfd
   neighbor fe80::11%Et1 auto-local-addr
   neighbor fe80::12%Et2 remote-as 65002
   neighbor fe80::12%Et2 bfd
   neighbor fe80::12%Et2 auto-local-addr
   neighbor fe80::13%Et3 remote-as 65003
   neighbor fe80::13%Et3 bfd
   neighbor fe80::13%Et3 auto-local-addr
   
   address-family ipv4
      neighbor fe80::11%Et1 activate
      neighbor fe80::12%Et2 activate
      neighbor fe80::13%Et3 activate
      network 10.2.1.0/31
      network 10.2.1.2/31
      network 10.2.1.4/31
   
   address-family ipv6
      neighbor fe80::11%Et1 activate
      neighbor fe80::12%Et2 activate
      neighbor fe80::13%Et3 activate

```

- [spine-2](Config/spine-2.cfg)

```
interface Ethernet1
   description to-leaf-1
   no switchport
   ip address 10.2.2.0/31
   ipv6 enable
   ipv6 address fe80::2/64 link-local

interface Ethernet2
   description to-leaf-2
   no switchport
   ip address 10.2.2.2/31
   ipv6 enable
   ipv6 address fe80::2/64 link-local

interface Ethernet3
   description to-leaf-3
   no switchport
   ip address 10.2.2.4/31
   ipv6 enable
   ipv6 address fe80::2/64 link-local

router bgp 65000
   router-id 10.0.2.0
   bgp default ipv4-unicast transport ipv6
   neighbor fe80::11%Et1 remote-as 65001
   neighbor fe80::11%Et1 bfd
   neighbor fe80::11%Et1 auto-local-addr
   neighbor fe80::12%Et2 remote-as 65002
   neighbor fe80::12%Et2 bfd
   neighbor fe80::12%Et2 auto-local-addr
   neighbor fe80::13%Et3 remote-as 65003
   neighbor fe80::13%Et3 bfd
   neighbor fe80::13%Et3 auto-local-addr
   
   address-family ipv4
      neighbor fe80::11%Et1 activate
      neighbor fe80::12%Et2 activate
      neighbor fe80::13%Et3 activate
      network 10.2.2.0/31
      network 10.2.2.2/31
      network 10.2.2.4/31
   
   address-family ipv6
      neighbor fe80::11%Et1 activate
      neighbor fe80::12%Et2 activate
      neighbor fe80::13%Et3 activate

```

- [leaf-1.cfg](Config/leaf-1.cfg)

```
interface Ethernet1
   description to-spine-1
   no switchport
   ip address 10.2.1.1/31
   ipv6 enable
   ipv6 address fe80::11/64 link-local

interface Ethernet2
   description to-spine-2
   no switchport
   ip address 10.2.2.1/31
   ipv6 enable
   ipv6 address fe80::11/64 link-local

router bgp 65001
   router-id 10.1.0.1
   bgp default ipv4-unicast transport ipv6
   neighbor spines peer group
   neighbor spines remote-as 65000
   neighbor spines bfd
   neighbor spines auto-local-addr
   neighbor fe80::1%Et1 peer group spines
   neighbor fe80::2%Et2 peer group spines
   
   address-family ipv4
      neighbor spines activate
      network 10.2.1.0/31
      network 10.2.2.0/31
   
   address-family ipv6
      neighbor spines activate

```

- [leaf-2.cfg](Config/leaf-2.cfg)

```
interface Ethernet1
   description to-spine-1
   no switchport
   ip address 10.2.1.3/31
   ipv6 enable
   ipv6 address fe80::12/64 link-local

interface Ethernet2
   description to-spine-2
   no switchport
   ip address 10.2.2.3/31
   ipv6 enable
   ipv6 address fe80::12/64 link-local

router bgp 65002
   router-id 10.1.0.2
   bgp default ipv4-unicast transport ipv6
   neighbor spines peer group
   neighbor spines remote-as 65000
   neighbor spines bfd
   neighbor spines auto-local-addr
   neighbor fe80::1%Et1 peer group spines
   neighbor fe80::2%Et2 peer group spines
   
   address-family ipv4
      neighbor spines activate
      network 10.2.1.2/31
      network 10.2.2.2/31
   
   address-family ipv6
      neighbor spines activate

```

- [leaf-3.cfg](Config/leaf-3.cfg)

```
interface Ethernet1
   description to-spine-1
   no switchport
   ip address 10.2.1.5/31
   ipv6 enable
   ipv6 address fe80::13/64 link-local

interface Ethernet2
   description to-spine-2
   no switchport
   ip address 10.2.2.5/31
   ipv6 enable
   ipv6 address fe80::13/64 link-local

router bgp 65003
   router-id 10.1.0.3
   bgp default ipv4-unicast transport ipv6
   neighbor spines peer group
   neighbor spines remote-as 65000
   neighbor spines bfd
   neighbor spines auto-local-addr
   neighbor fe80::1%Et1 peer group spines
   neighbor fe80::2%Et2 peer group spines
   !
   address-family ipv4
      neighbor spines activate
      network 10.2.1.4/31
      network 10.2.2.4/31
   !
   address-family ipv6
      neighbor spines activate

```
