## VxLAN. Оптимизация таблиц маршрутизации

  
### Цель:

- Реализовать передачу суммарных префиксов через EVPN Route Type 5
  

## В этой самостоятельной работе мы ожидаем, что вы самостоятельно:
  
- Реализовать передачу суммарных префиксов через EVPN Route Type 5

### Описание/Пошаговая инструкция выполнения домашнего задания:

- Underlay и Overlay настроены, согласно  [предыдущей](https://github.com/niknav83/Data_center_network_design/tree/main/labs/lab07) лабораторной работе
- Настроить устройство DNS в AS 65005:
    - Интерфейсы, IP-адресация
    - eBGP
- На DNS создать интерфейс Loopback 1, Loopback 2 и задать им IP-адреса: они будет имитировать узел из внешней сети - 77.88.8.0/24
- Анонсировать на DNS в BGP суммарный маршрут с префиксом: 77.88.8.0/24
- LEAF3 выбран в качестве Border. На нём необходимо настроить eBGP с DNS
- Интерфейс LEAF3 для соединения с DNS добавить в VRF PROD
- Настройки для eBGP на LEAF3 выполнить в VRF PROD:
  - Анонсировать суммарный маршрут для всех клиентских сетей - 192.168.0.0/16
- Выполнить проверку наличия Route Type 5 и общую проверку работоспособности фабрики

## Схема стенда 
![img_1.png](img_1.PNG)

## Таблица адресов

| Device  | Interface | IP Address   | Subnet Mask     | Default Gateway |
|---------|-----------|--------------|-----------------|-----------------|
| Spine 1 | lo        | 192.168.0.1  | 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.1  | 255.255.255.252 |                 |
|         | E1/2      | 192.168.1.21 | 255.255.255.252 |                 |
|         | E1/3      | 192.168.1.33 | 255.255.255.252 |                 |
| Spine 1 | lo        | 192.168.0.2  | 255.255.255.255 |                 |
|         | E1/1      | 192.168.2.1  | 255.255.255.252 |                 |
|         | E1/2      | 192.168.2.21 | 255.255.255.252 |                 |
|         | E1/3      | 192.168.2.33 | 255.255.255.252 |                 |
| Leaf 1  | lo        | 192.168.0.11 | 255.255.255.255 |                 |
|         | lo10      | 192.168.0.111| 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.2  | 255.255.255.252 |                 |
|         | E1/2      | 192.168.2.2  | 255.255.255.252 |                 |
| Leaf 2  | lo        | 192.168.0.12 | 255.255.255.255 |                 |
|         | lo10      | 192.168.0.112| 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.22 | 255.255.255.252 |                 |
|         | E1/2      | 192.168.2.22 | 255.255.255.252 |                 |
|         | E1/7      | 192.168.20.1 | 255.255.255.252 |                 |
| Leaf 3  | lo        | 192.168.0.13 | 255.255.255.252 |                 |
|         | lo10      | 192.168.0.113| 255.255.255.252 |                 |
|         | E1/1      | 192.168.1.34 | 255.255.255.252 |                 |
|         | E1/2      | 192.168.2.34 | 255.255.255.255 |                 |
|         | E1/7      | 192.168.20.1 | 255.255.255.0   |                 |
|         | E1/8      | 192.168.10.1 | 255.255.255.0   |                 |
|         | E1/5      | 1.1.1.2      | 255.255.255.252 |                 |
| vEOS6   | vlan 10   | 192.168.10.10| 255.255.255.0   | 192.168.10.1    |
|         | vlan 20   | 192.168.20.20| 255.255.255.0   | 192.168.20.1    |
| R7      | eth0/0    | 192.168.20.2 | 255.255.255.0   | 192.168.20.1    |
| R8      | eth0/0    | 192.168.20.3 | 255.255.255.0   | 192.168.20.1    |
| R9      | eth0/0    | 192.168.10.2 | 255.255.255.0   | 192.168.10.1    |
| DNS     | lo        | 5.5.5.5      | 255.255.255.255 |                 |
|         | lo1       | 77.88.8.1    | 255.255.255.255 |                 |
|         | lo2       | 77.88.8.8    | 255.255.255.255 |                 |
|         | eth0/0    | 1.1.1.1      | 255.255.255.252 |                 |


### [Файлы конфигураций устройст и сама работа выполненная в EVE-NG ](https://github.com/niknav83/Data_center_network_design/tree/main/labs/lab08/configs)

В данной работе применялса образ Arista (vEOS, EOS-4.21.1.1F) 

Логин: admin 

## Приступаем к настрйке сети:

### Настройка сети из предыдущей работы.

<details>

<summary> Конфигурация для Spine-1: </summary>

```
service routing protocols model multi-agent
!
hostname SPINE-1
!
interface Ethernet1
   no switchport
   ip address 192.168.1.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 192.168.1.21/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   no switchport
   ip address 192.168.1.33/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 192.168.0.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router bgp 65000
   router-id 192.168.0.1
   maximum-paths 4
   neighbor 192.168.0.11 remote-as 65000
   neighbor 192.168.0.11 update-source Loopback0
   neighbor 192.168.0.11 route-reflector-client
   neighbor 192.168.0.11 send-community
   neighbor 192.168.0.11 maximum-routes 12000 
   neighbor 192.168.0.12 remote-as 65000
   neighbor 192.168.0.12 update-source Loopback0
   neighbor 192.168.0.12 route-reflector-client
   neighbor 192.168.0.12 send-community
   neighbor 192.168.0.12 maximum-routes 12000 
   neighbor 192.168.0.13 remote-as 65000
   neighbor 192.168.0.13 update-source Loopback0
   neighbor 192.168.0.13 route-reflector-client
   neighbor 192.168.0.13 send-community
   neighbor 192.168.0.13 maximum-routes 12000 
   !
   address-family evpn
      neighbor 192.168.0.11 activate
      neighbor 192.168.0.12 activate
      neighbor 192.168.0.13 activate
   !
   address-family ipv4
      no neighbor 192.168.0.11 activate
      no neighbor 192.168.0.12 activate
      no neighbor 192.168.0.13 activate
!
router ospf 1
   router-id 192.168.0.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000

```
</details>

<details>

<summary>Конфигурация для Spine-2: </summary>

```
service routing protocols model multi-agent
!
hostname SPINE-2
!
interface Ethernet1
   no switchport
   ip address 192.168.2.1/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 192.168.2.21/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   no switchport
   ip address 192.168.2.33/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 192.168.0.2/32
   ip ospf area 0.0.0.0
!
ip routing
!
router bgp 65000
   router-id 192.168.0.2
   maximum-paths 4
   neighbor 192.168.0.11 remote-as 65000
   neighbor 192.168.0.11 update-source Loopback0
   neighbor 192.168.0.11 route-reflector-client
   neighbor 192.168.0.11 send-community
   neighbor 192.168.0.11 maximum-routes 12000 
   neighbor 192.168.0.12 remote-as 65000
   neighbor 192.168.0.12 update-source Loopback0
   neighbor 192.168.0.12 route-reflector-client
   neighbor 192.168.0.12 send-community
   neighbor 192.168.0.12 maximum-routes 12000 
   neighbor 192.168.0.13 remote-as 65000
   neighbor 192.168.0.13 update-source Loopback0
   neighbor 192.168.0.13 route-reflector-client
   neighbor 192.168.0.13 send-community
   neighbor 192.168.0.13 maximum-routes 12000 
   !
   address-family evpn
      neighbor 192.168.0.11 activate
      neighbor 192.168.0.12 activate
      neighbor 192.168.0.13 activate
   !
   address-family ipv4
      no neighbor 192.168.0.11 activate
      no neighbor 192.168.0.12 activate
      no neighbor 192.168.0.13 activate
!
router ospf 1
   router-id 192.168.0.2
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000

```
</details>

<details>

<summary> Конфигурация для Leaf-1: </summary>

```
service routing protocols model multi-agent
!
hostname Leaf-1
!
vlan 10,20
!
vrf definition PROD
!
interface Port-Channel1
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:1111:2222:3333:4444
      route-target import 12:23:34:45:56:67
   lacp system-id 1111.2222.3333
   spanning-tree portfast
!
interface Ethernet1
   no switchport
   ip address 192.168.1.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 192.168.2.2/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet8
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   channel-group 1 mode active
   spanning-tree portfast
!
interface Loopback0
   ip address 192.168.0.11/32
   ip ospf area 0.0.0.0
!
interface Loopback10
   ip address 192.168.0.111/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding PROD
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback10
   vxlan udp-port 4789
   vxlan vlan 10,20 vni 10010,10020
   vxlan vrf PROD vni 100999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:11:11:22:22
!
ip routing
ip routing vrf PROD
!
router bgp 65000
   router-id 192.168.0.11
   maximum-paths 4
   neighbor 192.168.0.1 remote-as 65000
   neighbor 192.168.0.1 update-source Loopback0
   neighbor 192.168.0.1 send-community
   neighbor 192.168.0.1 maximum-routes 12000 
   neighbor 192.168.0.2 remote-as 65000
   neighbor 192.168.0.2 update-source Loopback0
   neighbor 192.168.0.2 send-community
   neighbor 192.168.0.2 maximum-routes 12000 
   !
   vlan 10
      rd auto
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor 192.168.0.1 activate
      neighbor 192.168.0.2 activate
   !
   address-family ipv4
      no neighbor 192.168.0.1 activate
      no neighbor 192.168.0.2 activate
   !
   vrf PROD
      rd 192.168.0.11:1
      route-target import evpn 65000:100999
      route-target export evpn 65000:100999
!
router ospf 1
   router-id 192.168.0.11
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>


<details>

<summary> Конфигурация для Leaf-2: </summary>

```
service routing protocols model multi-agent
!
hostname Leaf-2
!
vlan 10,20
!
vrf definition PROD
!
interface Port-Channel1
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   !
   evpn ethernet-segment
      identifier 0000:1111:2222:3333:4444
      route-target import 12:23:34:45:56:67
   lacp system-id 1111.2222.3333
   spanning-tree portfast
!
interface Ethernet1
   no switchport
   ip address 192.168.1.22/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 192.168.2.22/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet7
   switchport access vlan 20
!
interface Ethernet8
   switchport trunk allowed vlan 10,20
   switchport mode trunk
   channel-group 1 mode active
   spanning-tree portfast
!
interface Loopback0
   ip address 192.168.0.12/32
   ip ospf area 0.0.0.0
!
interface Loopback10
   ip address 192.168.0.112/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding PROD
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback10
   vxlan udp-port 4789
   vxlan vlan 10,20 vni 10010,10020
   vxlan vrf PROD vni 100999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:11:11:22:22
!
ip routing
ip routing vrf PROD
!
router bgp 65000
   router-id 192.168.0.12
   maximum-paths 4
   neighbor 192.168.0.1 remote-as 65000
   neighbor 192.168.0.1 update-source Loopback0
   neighbor 192.168.0.1 send-community
   neighbor 192.168.0.1 maximum-routes 12000 
   neighbor 192.168.0.2 remote-as 65000
   neighbor 192.168.0.2 update-source Loopback0
   neighbor 192.168.0.2 send-community
   neighbor 192.168.0.2 maximum-routes 12000 
   !
   vlan 10
      rd auto
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor 192.168.0.1 activate
      neighbor 192.168.0.2 activate
   !
   address-family ipv4
      no neighbor 192.168.0.1 activate
      no neighbor 192.168.0.2 activate
   !
   vrf PROD
      rd 192.168.0.12:1
      route-target import evpn 65000:100999
      route-target export evpn 65000:100999
!
router ospf 1
   router-id 192.168.0.12
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>


<details>

<summary> Конфигурация для Leaf-3: </summary>

```
service routing protocols model multi-agent
!
hostname Leaf-3
!
vlan 10,20
!
vrf definition PROD
!
interface Ethernet1
   no switchport
   ip address 192.168.1.34/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   no switchport
   ip address 192.168.2.34/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet5
   no switchport
   vrf forwarding PROD
   ip address 1.1.1.2/30
!
interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding PROD
   ip address virtual 192.168.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback10
   vxlan udp-port 4789
   vxlan vlan 10,20 vni 10010,10020
   vxlan vrf PROD vni 100999
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:11:11:22:22
!
ip routing
ip routing vrf PROD
!
router bgp 65000
   router-id 192.168.0.13
   maximum-paths 4
   neighbor 192.168.0.1 remote-as 65000
   neighbor 192.168.0.1 update-source Loopback0
   neighbor 192.168.0.1 send-community
   neighbor 192.168.0.1 maximum-routes 12000 
   neighbor 192.168.0.2 remote-as 65000
   neighbor 192.168.0.2 update-source Loopback0
   neighbor 192.168.0.2 send-community
   neighbor 192.168.0.2 maximum-routes 12000 
   !
   vlan 10
      rd auto
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor 192.168.0.1 activate
      neighbor 192.168.0.2 activate
   !
   address-family ipv4
      no neighbor 192.168.0.1 activate
      no neighbor 192.168.0.2 activate
   !
   vrf PROD
      rd 192.168.0.13:1
      route-target import evpn 65000:100999
      route-target export evpn 65000:100999
  !
router ospf 1
   router-id 192.168.0.13
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>


### Далее произведем необходимые настройки.

 Конфигурация для Leaf-3:

```
router bgp 65000
   router-id 192.168.0.13
   maximum-paths 4
   neighbor 192.168.0.1 remote-as 65000
   neighbor 192.168.0.1 update-source Loopback0
   neighbor 192.168.0.1 send-community
   neighbor 192.168.0.1 maximum-routes 12000 
   neighbor 192.168.0.2 remote-as 65000
   neighbor 192.168.0.2 update-source Loopback0
   neighbor 192.168.0.2 send-community
   neighbor 192.168.0.2 maximum-routes 12000 
   !
   vlan 10
      rd auto
      route-target both 65000:10010
      redistribute learned
   !
   vlan 20
      rd auto
      route-target both 65000:10020
      redistribute learned
   !
   address-family evpn
      neighbor 192.168.0.1 activate
      neighbor 192.168.0.2 activate
   !
   address-family ipv4
      no neighbor 192.168.0.1 activate
      no neighbor 192.168.0.2 activate
   !
   vrf PROD
      rd 192.168.0.13:1
      route-target import evpn 65000:100999
      route-target export evpn 65000:100999
      neighbor 1.1.1.1 remote-as 65005
      neighbor 1.1.1.1 update-source Ethernet5
      neighbor 1.1.1.1 send-community
      neighbor 1.1.1.1 maximum-routes 12000 
      network 1.1.1.0/30
      aggregate-address 192.168.0.0/16 as-set summary-only
```

 Конфигурация для DNS:

```
interface Loopback0
 no shutdown
 ip address 5.5.5.5 255.255.255.255
!
interface Loopback1
 no shutdown
 ip address 77.88.8.1 255.255.255.255
!
interface Loopback2
 no shutdown
 ip address 77.88.8.8 255.255.255.255
!
interface Ethernet0/0
 no shutdown
 ip address 1.1.1.1 255.255.255.252
!
interface Ethernet0/1
 no shutdown
 no ip address
 shutdown
!
interface Ethernet0/2
 no shutdown
 no ip address
 shutdown
!
interface Ethernet0/3
 no shutdown
 no ip address
 shutdown
!
router bgp 65005
 bgp log-neighbor-changes
 neighbor 1.1.1.2 remote-as 65000
 !
 address-family ipv4
  network 1.1.1.0 mask 255.255.255.252
  network 5.5.5.5 mask 255.255.255.255
  network 77.88.8.1 mask 255.255.255.255
  network 77.88.8.8 mask 255.255.255.255
  aggregate-address 77.88.8.0 255.255.255.0 as-set summary-only
  neighbor 1.1.1.2 activate
 exit-address-family
```

### Проверка работоспособности EVPN / VxLAN и Route Type 5


<details>
  
<summary>Вывод команд для Leaf-1 :</summary>

```
Leaf-1#show bgp evpn summary
BGP summary information for VRF default
Router identifier 192.168.0.11, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State  PfxRcd PfxAcc
  192.168.0.1      4  65000           3960      4000    0    0    2d06h Estab  14     14
  192.168.0.2      4  65000           3948      4008    0    0    2d06h Estab  14     14
```
```
Leaf-1#show bgp evpn
BGP routing table information for VRF default
Router identifier 192.168.0.11, local AS number 65000
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network             Next Hop         Metric  LocPref Weight Path
 * >Ec   RD: 192.168.0.112:20 mac-ip aabb.cc00.7000
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.1
 *  ec   RD: 192.168.0.112:20 mac-ip aabb.cc00.7000
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.2
 * >Ec   RD: 192.168.0.112:20 mac-ip aabb.cc00.7000 192.168.20.2
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.112:20 mac-ip aabb.cc00.7000 192.168.20.2
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.113:20 mac-ip aabb.cc00.8000
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:20 mac-ip aabb.cc00.8000
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.113:20 mac-ip aabb.cc00.8000 192.168.20.3
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:20 mac-ip aabb.cc00.8000 192.168.20.3
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.113:10 mac-ip aabb.cc00.9000
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 *  ec   RD: 192.168.0.113:10 mac-ip aabb.cc00.9000
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 * >Ec   RD: 192.168.0.113:10 mac-ip aabb.cc00.9000 192.168.10.2
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:10 mac-ip aabb.cc00.9000 192.168.10.2
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >     RD: 192.168.0.111:10 imet 192.168.0.111
                             -                -       -       0       i
 * >     RD: 192.168.0.111:20 imet 192.168.0.111
                             -                -       -       0       i
 * >Ec   RD: 192.168.0.112:10 imet 192.168.0.112
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.112:10 imet 192.168.0.112
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.112:20 imet 192.168.0.112
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.112:20 imet 192.168.0.112
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.113:10 imet 192.168.0.113
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:10 imet 192.168.0.113
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.113:20 imet 192.168.0.113
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:20 imet 192.168.0.113
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 5.5.5.5/32
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 5.5.5.5/32
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 77.88.8.0/24
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 77.88.8.0/24
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 192.168.0.0/16
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 192.168.0.0/16
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
```
```
Leaf-1#show ip bgp vrf PROD
BGP routing table information for VRF PROD
Router identifier 192.168.20.1, local AS number 65000
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network             Next Hop         Metric  LocPref Weight Path
 * >Ec   1.1.1.0/30          192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   1.1.1.0/30          192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   5.5.5.5/32          192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   5.5.5.5/32          192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   77.88.8.0/24        192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   77.88.8.0/24        192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   192.168.0.0/16      192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   192.168.0.0/16      192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   192.168.10.2/32     192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   192.168.10.2/32     192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   192.168.20.2/32     192.168.0.112    -       100     0      i Or-ID: 192.168.0.12 C-LST: 192.168.0.2
 *  ec   192.168.20.2/32     192.168.0.112    -       100     0      i Or-ID: 192.168.0.12 C-LST: 192.168.0.1
 * >Ec   192.168.20.3/32     192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   192.168.20.3/32     192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
```
```
Leaf-1#show bgp evpn route-type ?
  auto-discovery    Filter by Ethernet Auto-Discovery (A-D) route (Type 1)
  ethernet-segment  Filter by Ethernet Segment Route (Type 4)
  imet              Filter by Inclusive Multicast Ethernet Tag Route (Type 3)
  ip-prefix         Filter by IP Prefix Route (Type 5)
  mac-ip            Filter by MAC/IP advertisement route (Type 2)

Leaf-1#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 192.168.0.11, local AS number 65000
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network             Next Hop         Metric  LocPref Weight Path
 * >Ec   RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 5.5.5.5/32
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 5.5.5.5/32
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 77.88.8.0/24
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 77.88.8.0/24
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 192.168.0.0/16
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 192.168.0.0/16
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
```

</details>

<details>
  
<summary>Вывод команд для Leaf-2 :</summary>

```
Leaf-2#show bgp evpn summary
BGP summary information for VRF default
Router identifier 192.168.0.12, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State  PfxRcd PfxAcc
  192.168.0.1      4  65000           4071      3919    0    0    2d07h Estab  12     12
  192.168.0.2      4  65000           4077      3916    0    0    2d07h Estab  12     12
```
```
Leaf-2#show bgp evpn
BGP routing table information for VRF default
Router identifier 192.168.0.12, local AS number 65000
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network             Next Hop         Metric  LocPref Weight Path
 * >     RD: 192.168.0.112:20 mac-ip aabb.cc00.7000
                             -                -       -       0       i
 * >     RD: 192.168.0.112:20 mac-ip aabb.cc00.7000 192.168.20.2
                             -                -       -       0       i
 * >Ec   RD: 192.168.0.113:20 mac-ip aabb.cc00.8000
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:20 mac-ip aabb.cc00.8000
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.113:20 mac-ip aabb.cc00.8000 192.168.20.3
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:20 mac-ip aabb.cc00.8000 192.168.20.3
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.113:10 mac-ip aabb.cc00.9000
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:10 mac-ip aabb.cc00.9000
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.113:10 mac-ip aabb.cc00.9000 192.168.10.2
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:10 mac-ip aabb.cc00.9000 192.168.10.2
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.111:10 imet 192.168.0.111
                             192.168.0.111    -       100     0       i Or-ID: 192.168.0.11 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.111:10 imet 192.168.0.111
                             192.168.0.111    -       100     0       i Or-ID: 192.168.0.11 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.111:20 imet 192.168.0.111
                             192.168.0.111    -       100     0       i Or-ID: 192.168.0.11 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.111:20 imet 192.168.0.111
                             192.168.0.111    -       100     0       i Or-ID: 192.168.0.11 C-LST: 192.168.0.1
 * >     RD: 192.168.0.112:10 imet 192.168.0.112
                             -                -       -       0       i
 * >     RD: 192.168.0.112:20 imet 192.168.0.112
                             -                -       -       0       i
 * >Ec   RD: 192.168.0.113:10 imet 192.168.0.113
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:10 imet 192.168.0.113
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.113:20 imet 192.168.0.113
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.113:20 imet 192.168.0.113
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 *  ec   RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 * >Ec   RD: 192.168.0.13:1 ip-prefix 5.5.5.5/32
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 5.5.5.5/32
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 77.88.8.0/24
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 77.88.8.0/24
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 192.168.0.0/16
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 192.168.0.0/16
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
```
```
Leaf-2#show ip bgp vrf PROD
BGP routing table information for VRF PROD
Router identifier 192.168.20.1, local AS number 65000
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network             Next Hop         Metric  LocPref Weight Path
 * >Ec   1.1.1.0/30          192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 *  ec   1.1.1.0/30          192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 * >Ec   5.5.5.5/32          192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   5.5.5.5/32          192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   77.88.8.0/24        192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   77.88.8.0/24        192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   192.168.0.0/16      192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   192.168.0.0/16      192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   192.168.10.2/32     192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   192.168.10.2/32     192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   192.168.20.3/32     192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   192.168.20.3/32     192.168.0.113    -       100     0      i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
```
```
Leaf-2#show bgp evpn route-type ?
  auto-discovery    Filter by Ethernet Auto-Discovery (A-D) route (Type 1)
  ethernet-segment  Filter by Ethernet Segment Route (Type 4)
  imet              Filter by Inclusive Multicast Ethernet Tag Route (Type 3)
  ip-prefix         Filter by IP Prefix Route (Type 5)
  mac-ip            Filter by MAC/IP advertisement route (Type 2)

Leaf-2#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 192.168.0.12, local AS number 65000
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network             Next Hop         Metric  LocPref Weight Path
 * >Ec   RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 *  ec   RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 * >Ec   RD: 192.168.0.13:1 ip-prefix 5.5.5.5/32
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 5.5.5.5/32
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 77.88.8.0/24
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 77.88.8.0/24
                             192.168.0.113    0       100     0      65005 i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.13:1 ip-prefix 192.168.0.0/16
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.13:1 ip-prefix 192.168.0.0/16
                             192.168.0.113    -       100     0       i Or-ID: 192.168.0.13 C-LST: 192.168.0.1
```

</details>

<details>
  
<summary>Вывод команд для Leaf-3 :</summary>

```
Leaf-3#show bgp evpn summary
BGP summary information for VRF default
Router identifier 192.168.0.13, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State  PfxRcd PfxAcc
  192.168.0.1      4  65000           4039      3998    0    0    2d07h Estab  6      6
  192.168.0.2      4  65000           4063      4002    0    0    2d07h Estab  6      6
```
```
Leaf-3#show bgp evpn
BGP routing table information for VRF default
Router identifier 192.168.0.13, local AS number 65000
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network             Next Hop         Metric  LocPref Weight Path
 * >Ec   RD: 192.168.0.112:20 mac-ip aabb.cc00.7000
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.1
 *  ec   RD: 192.168.0.112:20 mac-ip aabb.cc00.7000
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.2
 * >Ec   RD: 192.168.0.112:20 mac-ip aabb.cc00.7000 192.168.20.2
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.112:20 mac-ip aabb.cc00.7000 192.168.20.2
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.1
 * >     RD: 192.168.0.113:20 mac-ip aabb.cc00.8000
                             -                -       -       0       i
 * >     RD: 192.168.0.113:20 mac-ip aabb.cc00.8000 192.168.20.3
                             -                -       -       0       i
 * >     RD: 192.168.0.113:10 mac-ip aabb.cc00.9000
                             -                -       -       0       i
 * >     RD: 192.168.0.113:10 mac-ip aabb.cc00.9000 192.168.10.2
                             -                -       -       0       i
 * >Ec   RD: 192.168.0.111:10 imet 192.168.0.111
                             192.168.0.111    -       100     0       i Or-ID: 192.168.0.11 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.111:10 imet 192.168.0.111
                             192.168.0.111    -       100     0       i Or-ID: 192.168.0.11 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.111:20 imet 192.168.0.111
                             192.168.0.111    -       100     0       i Or-ID: 192.168.0.11 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.111:20 imet 192.168.0.111
                             192.168.0.111    -       100     0       i Or-ID: 192.168.0.11 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.112:10 imet 192.168.0.112
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.112:10 imet 192.168.0.112
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.1
 * >Ec   RD: 192.168.0.112:20 imet 192.168.0.112
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.2
 *  ec   RD: 192.168.0.112:20 imet 192.168.0.112
                             192.168.0.112    -       100     0       i Or-ID: 192.168.0.12 C-LST: 192.168.0.1
 * >     RD: 192.168.0.113:10 imet 192.168.0.113
                             -                -       -       0       i
 * >     RD: 192.168.0.113:20 imet 192.168.0.113
                             -                -       -       0       i
 * >     RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             -                -       -       0       i
 *       RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             -                0       100     0      65005 i
 * >     RD: 192.168.0.13:1 ip-prefix 5.5.5.5/32
                             -                0       100     0      65005 i
 * >     RD: 192.168.0.13:1 ip-prefix 77.88.8.0/24
                             -                0       100     0      65005 i
 * >     RD: 192.168.0.13:1 ip-prefix 192.168.0.0/16
                             -                -       -       0       i
```
```
Leaf-3#show ip bgp vrf PROD
BGP routing table information for VRF PROD
Router identifier 192.168.20.1, local AS number 65000
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network             Next Hop         Metric  LocPref Weight Path
 * >     1.1.1.0/30          -                -       -       0      i
 *       1.1.1.0/30          1.1.1.1          0       100     0      65005 i
 * >     5.5.5.5/32          1.1.1.1          0       100     0      65005 i
 * >     77.88.8.0/24        1.1.1.1          0       100     0      65005 i
 * >     192.168.0.0/16      -                -       -       0      i
 *s>Ec   192.168.20.2/32     192.168.0.112    -       100     0      i Or-ID: 192.168.0.12 C-LST: 192.168.0.2
 *s ec   192.168.20.2/32     192.168.0.112    -       100     0      i Or-ID: 192.168.0.12 C-LST: 192.168.0.1

```
```
Leaf-3#show bgp evpn route-type ?
  auto-discovery    Filter by Ethernet Auto-Discovery (A-D) route (Type 1)
  ethernet-segment  Filter by Ethernet Segment Route (Type 4)
  imet              Filter by Inclusive Multicast Ethernet Tag Route (Type 3)
  ip-prefix         Filter by IP Prefix Route (Type 5)
  mac-ip            Filter by MAC/IP advertisement route (Type 2)

Leaf-3#show bgp evpn route-type ip-prefix ipv4
BGP routing table information for VRF default
Router identifier 192.168.0.13, local AS number 65000
Route status codes: s - suppressed, * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network             Next Hop         Metric  LocPref Weight Path
 * >     RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             -                -       -       0       i
 *       RD: 192.168.0.13:1 ip-prefix 1.1.1.0/30
                             -                0       100     0      65005 i
 * >     RD: 192.168.0.13:1 ip-prefix 5.5.5.5/32
                             -                0       100     0      65005 i
 * >     RD: 192.168.0.13:1 ip-prefix 77.88.8.0/24
                             -                0       100     0      65005 i
 * >     RD: 192.168.0.13:1 ip-prefix 192.168.0.0/16
                             -                -       -       0       i
```

</details>


<details>
<summary>Вывод команд для DNS :</summary>

```
DNS#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                1.1.1.1         YES NVRAM  up                    up
Ethernet0/1                unassigned      YES NVRAM  administratively down down
Ethernet0/2                unassigned      YES NVRAM  administratively down down
Ethernet0/3                unassigned      YES NVRAM  administratively down down
Loopback0                  5.5.5.5         YES NVRAM  up                    up
Loopback1                  77.88.8.1       YES NVRAM  up                    up
Loopback2                  77.88.8.8       YES NVRAM  up                    up
```
```
DNS#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      1.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        1.1.1.0/30 is directly connected, Ethernet0/0
L        1.1.1.1/32 is directly connected, Ethernet0/0
      5.0.0.0/32 is subnetted, 1 subnets
C        5.5.5.5 is directly connected, Loopback0
      77.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
B        77.88.8.0/24 [200/0] via 0.0.0.0, 01:30:01, Null0
C        77.88.8.1/32 is directly connected, Loopback1
C        77.88.8.8/32 is directly connected, Loopback2
B     192.168.0.0/16 [20/0] via 1.1.1.2, 01:16:05
```
```
DNS#show ip bgp all
For address family: IPv4 Unicast

BGP table version is 11, local router ID is 77.88.8.8
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   1.1.1.0/30       1.1.1.2                                0 65000 i
 *>                   0.0.0.0                  0         32768 i
 *>  5.5.5.5/32       0.0.0.0                  0         32768 i
 *>  77.88.8.0/24     0.0.0.0                       100  32768 i
 s>  77.88.8.1/32     0.0.0.0                  0         32768 i
 s>  77.88.8.8/32     0.0.0.0                  0         32768 i
 *>  192.168.0.0/16   1.1.1.2                                0 65000 i

For address family: IPv4 Multicast

For address family: L2VPN E-VPN

For address family: MVPNv4 Unicast

```

</details>
