# Проект на тему "Проектирование L3 сети ЦОД без внедрения L2"

## Задача:
   - Спроектировать сеть ЦОД с возможностью масштабирования и обеспечением отказоустойчивости на уровне L3.

## Используемые технологии:
   - VxLAN, EVPN, eBGP, MLAG, LACP.

## Схема сети:
![](img/Schema.png)

## Конфигурации устройств:
  - [spine-1](Config/spine-1.cfg)
  - [spine-2](Config/spine-2.cfg)
  - [leaf-1](Config/leaf-1.cfg)
  - [leaf-2](Config/leaf-2.cfg)
  - [leaf-3](Config/leaf-3.cfg)
  - [leaf-4](Config/leaf-4.cfg)
  - [GW1](Config/GW1.cfg)
  - [GW2](Config/GW2.cfg)
  - [spine-2-1](Config/spine-2-1.cfg)
  - [spine-2-2](Config/spine-2-2.cfg)
  - [leaf-2-1](Config/leaf-2-1.cfg)
  - [leaf-2-2](Config/leaf-2-2.cfg)
  - [leaf-2-3](Config/leaf-2-3.cfg)
  - [leaf-2-4](Config/leaf-2-4.cfg)
  - VRF RED:
      VPC_vlan100 ip - 192.168.10.254/24 gw 192.168.10.1
      VPC_vlan110 ip - 192.168.11.254/24 gw 192.168.11.1
  - VRF BLUE:
      VPC_vlan200 ip - 192.168.20.254/24 gw 192.168.20.1
      VPC_vlan210 ip - 192.168.21.254/24 gw 192.168.21.1
  - VRF YELLOW:
      VPC_vlan300 ip - 192.168.30.254/24 gw 192.168.30.1
      VPC_vlan310 ip - 192.168.31.254/24 gw 192.168.31.1
  - VRF BLACK:
      VPC_vlan101 ip - 192.168.110.254/24 gw 192.168.110.1
      VPC_vlan111 ip - 192.168.111.254/24 gw 192.168.111.1
  - VRF BROWN:
      VPC_vlan120 ip - 192.168.120.254/24 gw 192.168.120.1
      VPC_vlan121 ip - 192.168.121.254/24 gw 192.168.121.1
  - VRF PURPLE:
      VPC_vlan130 ip - 192.168.130.254/24 gw 192.168.130.1
      VPC_vlan131 ip - 192.168.131.254/24 gw 192.168.131.1

## Выборочная проверка доступности хостов между собой

- VPC_vlan100
```
VPC_vlan100 : 192.168.10.254 255.255.255.0 gateway 192.168.10.1

VPC_vlan100> ping 192.168.11.254

84 bytes from 192.168.11.254 icmp_seq=1 ttl=62 time=60.885 ms
84 bytes from 192.168.11.254 icmp_seq=2 ttl=62 time=91.547 ms
84 bytes from 192.168.11.254 icmp_seq=3 ttl=62 time=56.250 ms
84 bytes from 192.168.11.254 icmp_seq=4 ttl=62 time=47.998 ms
84 bytes from 192.168.11.254 icmp_seq=5 ttl=62 time=36.538 ms

VPC_vlan100> trace 192.168.11.254
trace to 192.168.11.254, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   10.212 ms  8.531 ms  7.363 ms
 2   10.2.4.2   21.111 ms  15.988 ms  26.909 ms
 3   *192.168.11.254   45.324 ms (ICMP type:3, code:3, Destination port unreachable)

VPC_vlan100> ping 192.168.120.254

84 bytes from 192.168.120.254 icmp_seq=1 ttl=60 time=152.857 ms
84 bytes from 192.168.120.254 icmp_seq=2 ttl=60 time=58.556 ms
84 bytes from 192.168.120.254 icmp_seq=3 ttl=60 time=77.542 ms
84 bytes from 192.168.120.254 icmp_seq=4 ttl=60 time=58.461 ms
84 bytes from 192.168.120.254 icmp_seq=5 ttl=60 time=52.484 ms

VPC_vlan100> trace 192.168.120.254
trace to 192.168.120.254, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   4.595 ms  3.633 ms  4.208 ms
 2   10.2.4.3   15.702 ms  18.704 ms  39.845 ms
 3   10.2.4.1   54.201 ms  47.740 ms  70.553 ms
 4   10.10.4.10   98.594 ms  68.173 ms  63.851 ms
 5   *192.168.120.254   50.292 ms (ICMP type:3, code:3, Destination port unreachable)

VPC_vlan100> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=62 time=40.283 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=62 time=85.794 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=62 time=40.213 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=62 time=52.473 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=62 time=42.382 ms

VPC_vlan100>
```
  
 - VPC_vlan121 

 ```
 VPC_vlan121 : 192.168.121.254 255.255.255.0 gateway 192.168.121.1

VPC_vlan121> ping 192.168.11.254

84 bytes from 192.168.11.254 icmp_seq=1 ttl=60 time=90.546 ms
84 bytes from 192.168.11.254 icmp_seq=2 ttl=60 time=57.547 ms
84 bytes from 192.168.11.254 icmp_seq=3 ttl=60 time=60.837 ms
84 bytes from 192.168.11.254 icmp_seq=4 ttl=60 time=69.866 ms
84 bytes from 192.168.11.254 icmp_seq=5 ttl=60 time=61.102 ms

VPC_vlan121> trace 192.168.11.254
trace to 192.168.11.254, 8 hops max, press Ctrl+C to stop
 1   192.168.121.1   8.469 ms  4.105 ms  4.284 ms
 2   192.168.120.1   14.009 ms  12.050 ms  22.259 ms
 3   10.10.4.9   43.230 ms  34.511 ms  54.925 ms
 4   10.2.4.2   86.137 ms  107.517 ms  68.323 ms
 5   *192.168.11.254   58.961 ms (ICMP type:3, code:3, Destination port unreachable)

VPC_vlan121> ping 192.168.120.254

84 bytes from 192.168.120.254 icmp_seq=1 ttl=62 time=44.161 ms
84 bytes from 192.168.120.254 icmp_seq=2 ttl=62 time=42.516 ms
84 bytes from 192.168.120.254 icmp_seq=3 ttl=62 time=31.201 ms
84 bytes from 192.168.120.254 icmp_seq=4 ttl=62 time=45.957 ms
84 bytes from 192.168.120.254 icmp_seq=5 ttl=62 time=36.446 ms

VPC_vlan121> trace 192.168.120.254
trace to 192.168.120.254, 8 hops max, press Ctrl+C to stop
 1   192.168.121.1   21.780 ms  10.362 ms  8.101 ms
 2   192.168.120.1   16.431 ms  56.896 ms  15.213 ms
 3   *192.168.120.254   30.915 ms (ICMP type:3, code:3, Destination port unreachable)

VPC_vlan121> ping 192.168.131.254

84 bytes from 192.168.131.254 icmp_seq=1 ttl=59 time=81.507 ms
84 bytes from 192.168.131.254 icmp_seq=2 ttl=59 time=51.269 ms
84 bytes from 192.168.131.254 icmp_seq=3 ttl=59 time=62.187 ms
84 bytes from 192.168.131.254 icmp_seq=4 ttl=59 time=63.464 ms
84 bytes from 192.168.131.254 icmp_seq=5 ttl=59 time=70.283 ms

VPC_vlan121> trace 192.168.131.254
trace to 192.168.131.254, 8 hops max, press Ctrl+C to stop
 1   192.168.121.1   9.151 ms  8.624 ms  14.411 ms
 2   192.168.120.1   17.703 ms  14.386 ms  12.701 ms
 3   10.10.4.9   54.798 ms  34.559 ms  32.528 ms
 4   10.10.4.18   44.837 ms  37.095 ms  35.737 ms
 5   192.168.131.1   62.086 ms  50.763 ms  66.116 ms
 6   *192.168.131.254   52.429 ms (ICMP type:3, code:3, Destination port unreachable)

VPC_vlan121>
 ```

## Тест при отключении маршрутизатора

![](img/VPC131.png)
