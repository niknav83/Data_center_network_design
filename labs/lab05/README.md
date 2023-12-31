### VxLAN. EVPN L2.

## Цель:

- Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами


## В этой самостоятельной работе мы ожидаем, что вы самостоятельно:
  
1. Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами


### Описание/Пошаговая инструкция выполнения домашнего задания:

- В качестве Underlay-сети для IP-связанности будем использовать OSPF. 
- Настроить eBGP peering в Address Family L2VPN EVPN, указать BGP-соседство с IP-адресами Loopback 0 соседних устройств
- Выбрать VNI и VLAN ID. Создать VLAN и связать его с VNI на всех LEAF-коммутаторах
- Создать интерфейсы Loopback 10 на LEAF-коммутаторах и задать им IP-адреса
- Создать и настроить интерфейсы VxLAN на всех LEAF-коммутаторах - NVE
- Прописать VLAN на физических интерфейсах в Access Mode, к которым подключены клиентские устройства
- Выполнить проверку работы EVPN и VxLAN на всех устройствах


## Схема стенда 
![img_1.png](img_1.PNG)


## Таблица адресов

| Device  | Interface | IP Address   | Subnet Mask     | Default Gateway |
|---------|-----------|--------------|-----------------|-----------------|
| Spine 1 | lo 0      | 192.168.0.1  | 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.0  | 255.255.255.254 |                 |
|         | E1/2      | 192.168.1.20 | 255.255.255.254 |                 |
|         | E1/3      | 192.168.1.30 | 255.255.255.254 |                 |
| Spine 1 | lo 0      | 192.168.0.2  | 255.255.255.255 |                 |
|         | E1/1      | 192.168.2.0  | 255.255.255.254 |                 |
|         | E1/2      | 192.168.2.20 | 255.255.255.254 |                 |
|         | E1/3      | 192.168.2.30 | 255.255.255.254 |                 |
| Leaf 1  | lo 0      | 192.168.0.11 | 255.255.255.255 |                 |
|         | lo 10     | 192.168.0.111| 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.1  | 255.255.255.254 |                 |
|         | E1/2      | 192.168.2.1  | 255.255.255.254 |                 |
|         | E1/3      | 192.168.10.1 | 255.255.255.0   |                 |
| Leaf 2  | lo 0      | 192.168.0.12 | 255.255.255.255 |                 |
|         | lo 10     | 192.168.0.112| 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.21 | 255.255.255.254 |                 |
|         | E1/2      | 192.168.2.21 | 255.255.255.254 |                 |
|         | E1/3      | 192.168.20.1 | 255.255.255.0   |                 |
| Leaf 3  | lo 0      | 192.168.0.13 | 255.255.255.255 |                 |
|         | lo 10     | 192.168.0.113| 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.31 | 255.255.255.254 |                 |
|         | E1/2      | 192.168.2.31 | 255.255.255.254 |                 |
|         | E1/3      | 192.168.30.1 | 255.255.255.0   |                 |
|         | E1/4      | 192.168.40.1 | 255.255.255.0   |                 |
| VPC1    | eth0      | 192.168.10.1 | 255.255.255.0   |                 |
| VPC2    | eth0      | 192.168.20.1 | 255.255.255.0   |                 |
| VPC3    | eth0      | 192.168.10.2 | 255.255.255.0   |                 |
| VPC4    | eth0      | 192.168.20.2 | 255.255.255.0   |                 |


### [Файлы конфигураций устройст и сама работа выполненная в EVE-NG ](https://github.com/niknav83/Data_center_network_design/tree/main/labs/lab05/configs)

В данной работе применялса образ nxosv9k-9500-9.3.8

Логин и пароль: admin 

## Приступаем к настрйке сети:

### Настроим интерфейсы, IP адреса  и OSPF на всех устройствах Underlay-сети.

<details>

<summary> Конфигурация для Spine-1: </summary>

```
interface loopback0
  ip address 192.168.0.1/32
  ip router ospf Underlay area 0.0.0.0

interface Ethernet1/1
  medium p2p
  ip address 192.168.1.0/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.1.20/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/3
  medium p2p
  ip address 192.168.1.30/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

router ospf Underlay
  router-id 192.168.0.1
  passive-interface default
```
</details>


<details>

<summary> Конфигурация для Spine-2: </summary>

```
interface loopback0
  ip address 192.168.0.2/32
  ip router ospf Underlay area 0.0.0.0

interface Ethernet1/1
  medium p2p
  ip address 192.168.2.0/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.20/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/3
  medium p2p
  ip address 192.168.2.30/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

router ospf Underlay
  router-id 192.168.0.2
  passive-interface default
```
</details>


<details>

<summary> Конфигурация для Leaf-1: </summary>

```
interface loopback0
  ip address 192.168.0.11/32
  ip router ospf Underlay area 0.0.0.0

interface loopback10
  ip address 192.168.0.111/32
  ip router ospf Underlay area 0.0.0.0

interface Ethernet1/1
  medium p2p
  ip address 192.168.1.1/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.1/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/3
  switchport
  switchport access vlan 10
  no shutdown

interface loopback0
  ip address 192.168.0.11/32
  ip router ospf Underlay area 0.0.0.0
```
</details>


<details>

<summary> Конфигурация  для Leaf-2: </summary>

```
interface loopback0
  ip address 192.168.0.12/32
  ip router ospf Underlay area 0.0.0.0

interface loopback10
  ip address 192.168.0.112/32
  ip router ospf Underlay area 0.0.0.0

interface Ethernet1/1
  medium p2p
  ip address 192.168.1.21/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.21/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/3
  switchport
  switchport access vlan 20
  no shutdown

router ospf Underlay
  router-id 192.168.0.12
  passive-interface default
```
</details>


<details>

<summary> Конфигурация  для Leaf-3: </summary>

```
interface loopback0
  ip address 192.168.0.13/32
  ip router ospf Underlay area 0.0.0.0

interface loopback10
  ip address 192.168.0.113/32
  ip router ospf Underlay area 0.0.0.0

interface Ethernet1/1
  medium p2p
  ip address 192.168.1.31/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.31/31
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/3
  switchport
  switchport access vlan 10
  no shutdown

interface Ethernet1/4
  switchport
  switchport access vlan 20
  no shutdown

router ospf Underlay
  router-id 192.168.0.13
  passive-interface default
```
</details>




### Далее на всех устройствах произведем необходимые настройки.


Конфигурация для Spine-1:

```
nv overlay evpn
feature ospf
feature bgp
feature bfd

route-map NEXT-HOP-UNCH permit 10
  set ip next-hop unchanged

router bgp 65000
  router-id 192.168.0.1
  timers bgp 3 9
  reconnect-interval 10
  address-family l2vpn evpn
    maximum-paths 10
    retain route-target all
  template peer LEAVES
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map NEXT-HOP-UNCH out
      rewrite-evpn-rt-asn
  neighbor 192.168.0.11
    inherit peer LEAVES
    remote-as 65001
  neighbor 192.168.0.12
    inherit peer LEAVES
    remote-as 65002
  neighbor 192.168.0.13
    inherit peer LEAVES
    remote-as 65003

```

 Конфигурация для Spine-2:

```
nv overlay evpn
feature ospf
feature bgp
feature bfd

route-map NEXT-HOP-UNCH permit 10
  set ip next-hop unchanged

router bgp 65000
  router-id 192.168.0.2
  timers bgp 3 9
  reconnect-interval 10
  address-family l2vpn evpn
    maximum-paths 10
    retain route-target all
  template peer LEAVES
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map NEXT-HOP-UNCH out
      rewrite-evpn-rt-asn
  neighbor 192.168.0.11
    inherit peer LEAVES
    remote-as 65001
  neighbor 192.168.0.12
    inherit peer LEAVES
    remote-as 65002
  neighbor 192.168.0.13
    inherit peer LEAVES
    remote-as 65003
```

 Конфигурация для Leaf-1:

```
nv overlay evpn
feature ospf
feature bgp
feature vn-segment-vlan-based
feature bfd
feature nv overlay

vlan 1,10
vlan 10
  vn-segment 10010

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback10
  member vni 10010
    ingress-replication protocol bgp

router bgp 65001
  router-id 192.168.0.11
  timers bgp 3 9
  reconnect-interval 10
  address-family l2vpn evpn
    maximum-paths 10
  template peer SPINES
    remote-as 65000
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  neighbor 192.168.0.1
    inherit peer SPINES
  neighbor 192.168.0.2
    inherit peer SPINES
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
```

 Конфигурация для Leaf-2:

```
nv overlay evpn
feature ospf
feature bgp
feature vn-segment-vlan-based
feature bfd
feature nv overlay

vlan 1,20
vlan 20
  vn-segment 10020

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback10
  member vni 10020
    ingress-replication protocol bgp

router bgp 65002
  router-id 192.168.0.12
  timers bgp 3 9
  reconnect-interval 10
  address-family l2vpn evpn
    maximum-paths 10
  template peer SPINES
    remote-as 65000
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  neighbor 192.168.0.1
    inherit peer SPINES
  neighbor 192.168.0.2
    inherit peer SPINES
evpn
  vni 10020 l2
    rd auto
    route-target import auto
    route-target export auto
```

 Конфигурация для Leaf-3:

```
nv overlay evpn
feature ospf
feature bgp
feature vn-segment-vlan-based
feature bfd
feature nv overlay

vlan 1,10,20
vlan 10
  vn-segment 10010
vlan 20
  vn-segment 10020

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback10
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp

router bgp 65003
  router-id 192.168.0.13
  timers bgp 3 9
  reconnect-interval 10
  address-family l2vpn evpn
    maximum-paths 10
  template peer SPINES
    remote-as 65000
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  neighbor 192.168.0.1
    inherit peer SPINES
  neighbor 192.168.0.2
    inherit peer SPINES
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 10020 l2
    rd auto
    route-target import auto
    route-target export auto
```

### Проверка работоспособности EVPN / VxLAN. Проверяем соседство по L2VPN между устройствами и таблицу маршрутизации Route Distinguisher. На LEAF-коммутаторах проверяем также NVE Peers:


<details>
  
<summary>Вывод команд Spine-1 :</summary>

```
Spine-1# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 192.168.0.1, local AS number 65000
BGP table version is 38, L2VPN EVPN config peers 3, capable peers 3
8 network entries and 8 paths using 1952 bytes of memory
BGP attribute entries [8/1376], BGP AS path entries [3/18]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.0.11    4 65001     219     211       38    0    0 00:06:48 2
192.168.0.12    4 65002     221     212       38    0    0 00:06:50 2
192.168.0.13    4 65003     222     212       38    0    0 00:06:56 4
```
```
Spine-1# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 38, Local Router ID is 192.168.0.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 192.168.0.11:32777
*>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                      192.168.0.111                                  0 65001 i
*>e[3]:[0]:[32]:[192.168.0.111]/88
                      192.168.0.111                                  0 65001 i

Route Distinguisher: 192.168.0.12:32787
*>e[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                      192.168.0.112                                  0 65002 i
*>e[3]:[0]:[32]:[192.168.0.112]/88
                      192.168.0.112                                  0 65002 i

Route Distinguisher: 192.168.0.13:32777
*>e[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      192.168.0.113                                  0 65003 i
*>e[3]:[0]:[32]:[192.168.0.113]/88
                      192.168.0.113                                  0 65003 i

Route Distinguisher: 192.168.0.13:32787
*>e[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                      192.168.0.113                                  0 65003 i
*>e[3]:[0]:[32]:[192.168.0.113]/88
                      192.168.0.113                                  0 65003 i
```
</details>

<details>
  
<summary>Вывод команд Spine-2 :</summary>

```
Spine-2# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 192.168.0.2, local AS number 65000
BGP table version is 26, L2VPN EVPN config peers 3, capable peers 3
8 network entries and 8 paths using 1952 bytes of memory
BGP attribute entries [8/1376], BGP AS path entries [3/18]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.0.11    4 65001     262     259       26    0    0 00:09:10 2
192.168.0.12    4 65002     264     264       26    0    0 00:13:14 2
192.168.0.13    4 65003     266     258       26    0    0 00:09:10 4
```
```
Spine-2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 26, Local Router ID is 192.168.0.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 192.168.0.11:32777
*>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                      192.168.0.111                                  0 65001 i
*>e[3]:[0]:[32]:[192.168.0.111]/88
                      192.168.0.111                                  0 65001 i

Route Distinguisher: 192.168.0.12:32787
*>e[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                      192.168.0.112                                  0 65002 i
*>e[3]:[0]:[32]:[192.168.0.112]/88
                      192.168.0.112                                  0 65002 i

Route Distinguisher: 192.168.0.13:32777
*>e[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      192.168.0.113                                  0 65003 i
*>e[3]:[0]:[32]:[192.168.0.113]/88
                      192.168.0.113                                  0 65003 i

Route Distinguisher: 192.168.0.13:32787
*>e[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                      192.168.0.113                                  0 65003 i
*>e[3]:[0]:[32]:[192.168.0.113]/88
                      192.168.0.113                                  0 65003 i
```
</details>

<details>
  
<summary>Вывод команд Leaf-1 :</summary>


```
Leaf-1# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 192.168.0.11, local AS number 65001
BGP table version is 300, L2VPN EVPN config peers 2, capable peers 2
6 network entries and 8 paths using 1464 bytes of memory
BGP attribute entries [5/860], BGP AS path entries [1/10]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.0.1     4 65000    5934    5863      300    0    0 00:10:09 2
192.168.0.2     4 65000   11601   11560      300    0    0 00:10:17 2
```
```
Leaf-1# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 300, Local Router ID is 192.168.0.11
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 192.168.0.11:32777    (L2VNI 10010)
*>l[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                      192.168.0.111                     100      32768 i
*>e[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      192.168.0.113                                  0 65000 65003 i
*>l[3]:[0]:[32]:[192.168.0.111]/88
                      192.168.0.111                     100      32768 i
*>e[3]:[0]:[32]:[192.168.0.113]/88
                      192.168.0.113                                  0 65000 65003 i

Route Distinguisher: 192.168.0.13:32777
* e[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      192.168.0.113                                  0 65000 65003 i
*>e                   192.168.0.113                                  0 65000 65003 i
* e[3]:[0]:[32]:[192.168.0.113]/88
                      192.168.0.113                                  0 65000 65003 i
*>e                   192.168.0.113                                  0 65000 65003 i
```
```
Leaf-1# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      192.168.0.113                           Up    CP        00:03:54 n/a
```

```
Leaf-1# show mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*   10     0050.7966.6806   dynamic  0         F      F    Eth1/3
C   10     0050.7966.6808   dynamic  0         F      F    nve1(192.168.0.113)
G    -     5003.0000.1b08   static   -         F      F    sup-eth1(R)
```
</details>

<details>
  
<summary>Вывод команд Leaf-2 :</summary>

```
Leaf-2# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 192.168.0.12, local AS number 65002
BGP table version is 318, L2VPN EVPN config peers 2, capable peers 2
6 network entries and 10 paths using 1464 bytes of memory
BGP attribute entries [5/860], BGP AS path entries [1/10]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.0.1     4 65000    5951    5900      318    0    0 00:01:08 2
192.168.0.2     4 65000   11639   11622      318    0    0 00:01:09 2
```
```
Leaf-2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 318, Local Router ID is 192.168.0.12
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 192.168.0.12:32787    (L2VNI 10020)
*>l[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                      192.168.0.112                     100      32768 i
*>e[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                      192.168.0.113                                  0 65000 65003 i
*>l[3]:[0]:[32]:[192.168.0.112]/88
                      192.168.0.112                     100      32768 i
*>e[3]:[0]:[32]:[192.168.0.113]/88
                      192.168.0.113                                  0 65000 65003 i

Route Distinguisher: 192.168.0.13:32787
* e[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                      192.168.0.113                                  0 65000 65003 i
*>e                   192.168.0.113                                  0 65000 65003 i
* e[3]:[0]:[32]:[192.168.0.113]/88
                      192.168.0.113                                  0 65000 65003 i
*>e                   192.168.0.113                                  0 65000 65003 i
```
```
Leaf-2# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      192.168.0.113                           Up    CP        00:11:32 n/a
```
```
Leaf-2# show mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*   20     0050.7966.6807   dynamic  0         F      F    Eth1/3
C   20     0050.7966.6809   dynamic  0         F      F    nve1(192.168.0.113)
G    -     5004.0000.1b08   static   -         F      F    sup-eth1(R)

```
</details>

<details>
  
<summary>Вывод команд Leaf-3 :</summary>

```
Leaf-3# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 192.168.0.13, local AS number 65003
BGP table version is 111, L2VPN EVPN config peers 2, capable peers 2
12 network entries and 16 paths using 2928 bytes of memory
BGP attribute entries [10/1720], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.0.1     4 65000     356     342      111    0    0 00:00:07 4
192.168.0.2     4 65000     357     345      111    0    0 00:00:09 4
```
```
Leaf-3# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 111, Local Router ID is 192.168.0.13
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 192.168.0.11:32777
* e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                      192.168.0.111                                  0 65000 65001 i
*>e                   192.168.0.111                                  0 65000 65001 i
* e[3]:[0]:[32]:[192.168.0.111]/88
                      192.168.0.111                                  0 65000 65001 i
*>e                   192.168.0.111                                  0 65000 65001 i

Route Distinguisher: 192.168.0.12:32787
*>e[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                      192.168.0.112                                  0 65000 65002 i
* e                   192.168.0.112                                  0 65000 65002 i
* e[3]:[0]:[32]:[192.168.0.112]/88
                      192.168.0.112                                  0 65000 65002 i
*>e                   192.168.0.112                                  0 65000 65002 i

Route Distinguisher: 192.168.0.13:32777    (L2VNI 10010)
*>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                      192.168.0.111                                  0 65000 65001 i
*>l[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      192.168.0.113                     100      32768 i
*>e[3]:[0]:[32]:[192.168.0.111]/88
                      192.168.0.111                                  0 65000 65001 i
*>l[3]:[0]:[32]:[192.168.0.113]/88
                      192.168.0.113                     100      32768 i

Route Distinguisher: 192.168.0.13:32787    (L2VNI 10020)
*>e[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                      192.168.0.112                                  0 65000 65002 i
*>l[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                      192.168.0.113                     100      32768 i
*>e[3]:[0]:[32]:[192.168.0.112]/88
                      192.168.0.112                                  0 65000 65002 i
*>l[3]:[0]:[32]:[192.168.0.113]/88
                      192.168.0.113                     100      32768 i
```

```
Leaf-3# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      192.168.0.111                           Up    CP        00:02:31 n/a

nve1      192.168.0.112                           Up    CP        00:15:38 n/a
```
```
Leaf-3# sho mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
C   10     0050.7966.6806   dynamic  0         F      F    nve1(192.168.0.111)
*   10     0050.7966.6808   dynamic  0         F      F    Eth1/3
C   20     0050.7966.6807   dynamic  0         F      F    nve1(192.168.0.112)
*   20     0050.7966.6809   dynamic  0         F      F    Eth1/4
G    -     5005.0000.1b08   static   -         F      F    sup-eth1(R)
```

</details>


