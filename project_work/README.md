## Проектная работа "Оптимизация сетевой архитектуры: переход от классической 3-Tier топологии к evpn/vxlan и внедрение функции мультихоминга для эффективного подключения серверов"

### Оглавление
1. [Цель](#цель)
2. [Задачи](#задачи)
3. [Физическая схема](#физическая-схема-сети)

### Цель:
- Спроектировать и реализовать отказоустойчивое, масштабируемое решение сети для подключения серверов и клиентских устройств.
  
### Задачи:
- Проектирование отказоустойчивой и масштабируемой топологии Clos для ЦОД
- Проектирование адресного пространства



### Физическая схема сети:

![img_0.png](img_0.PNG)


### Адресация для построения сети Underlay


| Device  | Interface | IP Address | Subnet Mask     | Default Gateway |
|---------|-----------|------------|-----------------|-----------------|
| Spine 1 | lo        | 10.255.0.1 | 255.255.255.255 |                 |
|         | Eth1      | 10.0.1.1   | 255.255.255.252 |                 |
|         | Eth2      | 10.0.1.5   | 255.255.255.252 |                 |
|         | Eth3      | 10.0.1.9   | 255.255.255.252 |                 |
|         | Eth4      | 10.0.1.13  | 255.255.255.252 |                 |
|         | Eth5      | 10.0.1.17  | 255.255.255.252 |                 |
| Spine 2 | lo        | 10.255.0.2 | 255.255.255.255 |                 |
|         | Eth1      | 10.0.2.1   | 255.255.255.252 |                 |
|         | Eth2      | 10.0.2.5   | 255.255.255.252 |                 |
|         | Eth3      | 10.0.2.9   | 255.255.255.252 |                 |
|         | Eth4      | 10.0.2.13  | 255.255.255.252 |                 |
|         | Eth5      | 10.0.2.17  | 255.255.255.252 |                 |
| Leaf 1  | lo        | 10.255.1.1 | 255.255.255.255 |                 |
|         | Eth1      | 10.0.1.2   | 255.255.255.252 |                 |
|         | Eth2      | 10.0.2.2   | 255.255.255.252 |                 |
| Leaf 2  | lo        | 10.255.1.2 | 255.255.255.255 |                 |
|         | Eth1      | 10.0.1.6   | 255.255.255.252 |                 |
|         | Eth2      | 10.0.2.6   | 255.255.255.252 |                 |
| Leaf 3  | lo        | 10.255.2.1 | 255.255.255.255 |                 |
|         | Eth1      | 10.0.1.10  | 255.255.255.252 |                 |
|         | Eth2      | 10.0.2.10  | 255.255.255.252 |                 |
| Leaf 4  | lo        | 10.255.2.2 | 255.255.255.255 |                 |
|         | Eth1      | 10.0.1.14  | 255.255.255.252 |                 |
|         | Eth2      | 10.0.2.14  | 255.255.255.252 |                 |
| Leaf 5  | lo        | 10.255.3.1 | 255.255.255.255 |                 |
|         | Eth1      | 10.0.1.18  | 255.255.255.252 |                 |
|         | Eth2      | 10.0.2.18  | 255.255.255.252 |                 |


### Адресация для построения сети Overlay


| Device  | Interface | IP Address | Subnet Mask     | Default Gateway |
|---------|-----------|------------|-----------------|-----------------|
| Leaf 1  | lo10      | 10.255.10.1| 255.255.255.255 |                 |
| Leaf 2  | lo10      | 10.255.10.2| 255.255.255.255 |                 |
| Leaf 3  | lo10      | 10.255.10.3| 255.255.255.255 |                 |
| Leaf 4  | lo10      | 10.255.10.4| 255.255.255.255 |                 |
| Leaf 5  | lo10      | 10.255.10.5| 255.255.255.255 |                 |


#### Сетевые зоны
Зоны одинаковы на всех LEAF коммутаторах. 

| Network         | Gateway / Interface VLAN IP Address | VLAN ID | VLAN Name      | VRF   | VNI   | Description |
|-----------------|-------------------------------------|---------|----------------|-------|-------|-------------|
| 192.168.10.0/24 | 192.168.10.1                        | 10      | VMWare_vSphere | PROD  | 10010 |             |
| 192.168.20.0/24 | 192.168.20.1                        | 20      | VoIP           | VoIP  | 10020 |             |
| 192.168.30.0/24 | 192.168.30.1                        | 30      | Video          | Video | 10030 |             |
| 192.168.40.0/24 | 192.168.40.1                        | 40      | PROD           | PROD  | 10040 |             |


## Приступаем к настрйке сети:

### Настроим интерфейсы, IP адреса и OSPF на всех устройствах Underlay-сети.

<details>

<summary> Конфигурация интерфейсов и OSPF для Spine-1: </summary>

```
hostname Spine-1
!
interface Ethernet1
   mtu 9214
   no switchport
   ip address 10.0.1.1/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   mtu 9214
   no switchport
   ip address 10.0.1.5/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   mtu 9214
   no switchport
   ip address 10.0.1.9/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet4
   mtu 9214
   no switchport
   ip address 10.0.1.13/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet5
   mtu 9214
   no switchport
   ip address 10.0.1.17/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.255.0.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.255.0.1
   bfd all-interfaces
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   no passive-interface Ethernet4
   no passive-interface Ethernet5
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>


<details>

<summary>Конфигурация интерфейсов и OSPF для Spine-2: </summary>

```
hostname Spine-2
!
interface Ethernet1
   mtu 9214
   no switchport
   ip address 10.0.2.1/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   mtu 9214
   no switchport
   ip address 10.0.2.5/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   mtu 9214
   no switchport
   ip address 10.0.2.9/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet4
   mtu 9214
   no switchport
   ip address 10.0.2.13/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet5
   mtu 9214
   no switchport
   ip address 10.0.2.17/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.255.0.2/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.255.0.2
   bfd all-interfaces
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   no passive-interface Ethernet4
   no passive-interface Ethernet5
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>


<details>

<summary> Конфигурация интерфейсов и OSPF для Leaf-1: </summary>

```
hostname Leaf-1
!
interface Ethernet1
   mtu 9214
   no switchport
   ip address 10.0.1.2/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   mtu 9214
   no switchport
   ip address 10.0.2.2/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.255.1.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.255.1.1
   bfd all-interfaces
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>


<details>

<summary> Конфигурация интерфейсов и OSPF для Leaf-2: </summary>

```
hostname Leaf-2
!
interface Ethernet1
   mtu 9214
   no switchport
   ip address 10.0.1.6/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   mtu 9214
   no switchport
   ip address 10.0.2.6/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.255.1.2/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.255.1.2
   bfd all-interfaces
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>


<details>

<summary> Конфигурация интерфейсов и OSPF для Leaf-3: </summary>

```
hostname Leaf-3
!
interface Ethernet1
   mtu 9214
   no switchport
   ip address 10.0.1.10/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   mtu 9214
   no switchport
   ip address 10.0.2.10/30
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.255.2.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.255.2.1
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>



<details>

<summary> Конфигурация интерфейсов и OSPF для Leaf-4: </summary>

```
hostname Leaf-4
!
interface Ethernet1
   mtu 9214
   no switchport
   ip address 10.0.1.14/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   mtu 9214
   no switchport
   ip address 10.0.2.14/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.255.2.2/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.255.2.2
   bfd all-interfaces
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>


<details>

<summary> Конфигурация интерфейсов и OSPF для Leaf-5: </summary>

```
hostname Leaf-5
!
interface Ethernet1
   mtu 9214
   no switchport
   ip address 10.0.1.18/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   mtu 9214
   no switchport
   ip address 10.0.2.18/30
   ip ospf bfd
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip address 10.255.3.1/32
   ip ospf area 0.0.0.0
!
ip routing
!
router ospf 1
   router-id 10.255.3.1
   bfd all-interfaces
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>


### Настроим интерфейсы Loopback10, VLAN, VRF на всех устройствах Overlay-сети.

<summary> Конфигурация интерфейсов для Leaf-1: </summary>

```
vlan 10,20,30,40
!
vrf definition PROD
!
vrf definition Video
!
vrf definition VoIP
!
interface Loopback10
   ip address 10.255.10.1/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding VoIP
   ip address virtual 192.168.20.1/24
!
interface Vlan30
   vrf forwarding Video
   ip address virtual 192.168.30.1/24
!
interface Vlan40
   vrf forwarding PROD
   ip address virtual 192.168.40.1/24
!
ip virtual-router mac-address 00:00:11:11:22:22
!
ip routing
ip routing vrf Video
ip routing vrf VoIP
ip routing vrf PROD
!
```
</details>


<details>

<summary> Конфигурация интерфейсов для Leaf-2: </summary>

```
vlan 10,20,30,40
!
vrf definition PROD
!
vrf definition Video
!
vrf definition VoIP
!
interface Loopback10
   ip address 10.255.10.2/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding VoIP
   ip address virtual 192.168.20.1/24
!
interface Vlan30
   vrf forwarding Video
   ip address virtual 192.168.30.1/24
!
interface Vlan40
   vrf forwarding PROD
   ip address virtual 192.168.40.1/24
!
ip virtual-router mac-address 00:00:11:11:22:22
!
ip routing
ip routing vrf Video
ip routing vrf VoIP
ip routing vrf PROD
!
```
</details>


<details>

<summary> Конфигурация интерфейсов для Leaf-3: </summary>

```
vlan 10,20,30,40
!
vrf definition PROD
!
vrf definition Video
!
vrf definition VoIP
!
interface Loopback10
   ip address 10.255.10.3/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding VoIP
   ip address virtual 192.168.20.1/24
!
interface Vlan30
   vrf forwarding Video
   ip address virtual 192.168.30.1/24
!
interface Vlan40
   vrf forwarding PROD
   ip address virtual 192.168.40.1/24
!
ip virtual-router mac-address 00:00:11:11:22:22
!
ip routing
ip routing vrf Video
ip routing vrf VoIP
ip routing vrf PROD
!
```
</details>



<details>

<summary> Конфигурация интерфейсов для Leaf-4: </summary>

```
vlan 10,20,30,40
!
vrf definition PROD
!
vrf definition Video
!
vrf definition VoIP
!
interface Ethernet9

interface Loopback10
   ip address 10.255.10.4/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding VoIP
   ip address virtual 192.168.20.1/24
!
interface Vlan30
   vrf forwarding Video
   ip address virtual 192.168.30.1/24
!
interface Vlan40
   vrf forwarding PROD
   ip address virtual 192.168.40.1/24
!
ip virtual-router mac-address 00:00:11:11:22:22
!
ip routing
ip routing vrf Video
ip routing vrf VoIP
ip routing vrf PROD
!
```
</details>


<details>

<summary> Конфигурация интерфейсов для Leaf-5: </summary>

```
vlan 10,20,30,40
!
vrf definition PROD
!
vrf definition Video
!
vrf definition VoIP
!
interface Loopback10
   ip address 10.255.10.5/32
   ip ospf area 0.0.0.0
!
interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding VoIP
   ip address virtual 192.168.20.1/24
!
interface Vlan30
   vrf forwarding Video
   ip address virtual 192.168.30.1/24
!
interface Vlan40
   vrf forwarding PROD
   ip address virtual 192.168.40.1/24
!
ip virtual-router mac-address 00:00:11:11:22:22
!
ip routing
ip routing vrf Video
ip routing vrf VoIP
ip routing vrf PROD
!
  ```
</details>



## Приступаем к настрйке сети Overlay:

### Настроим интерфейсы Loopback10, BGP, VXLAN на всех устройствах Overlay-сети.

<details>

<summary> Конфигурация интерфейсов и BGP для Spine-1: </summary>

```
router bgp 65000
   router-id 10.255.0.1
   maximum-paths 4
   neighbor LEAVES peer-group
   neighbor LEAVES remote-as 65000
   neighbor LEAVES update-source Loopback0
   neighbor LEAVES route-reflector-client
   neighbor LEAVES send-community
   neighbor LEAVES maximum-routes 12000 
   neighbor 10.255.1.1 peer-group LEAVES
   neighbor 10.255.1.2 peer-group LEAVES
   neighbor 10.255.2.1 peer-group LEAVES
   neighbor 10.255.2.2 peer-group LEAVES
   neighbor 10.255.3.1 peer-group LEAVES
   !
   address-family evpn
      neighbor LEAVES activate
   !
   address-family ipv4
      no neighbor LEAVES activate
!
```
</details>


<details>

<summary>Конфигурация интерфейсов и BGP для Spine-2: </summary>

```
router bgp 65000
   router-id 10.255.0.2
   maximum-paths 4
   neighbor LEAVES peer-group
   neighbor LEAVES remote-as 65000
   neighbor LEAVES update-source Loopback0
   neighbor LEAVES route-reflector-client
   neighbor LEAVES send-community
   neighbor LEAVES maximum-routes 12000 
   neighbor 10.255.1.1 peer-group LEAVES
   neighbor 10.255.1.2 peer-group LEAVES
   neighbor 10.255.2.1 peer-group LEAVES
   neighbor 10.255.2.2 peer-group LEAVES
   neighbor 10.255.3.1 peer-group LEAVES
   !
   address-family evpn
      neighbor LEAVES activate
   !
   address-family ipv4
      no neighbor LEAVES activate
!
```
</details>


<details>

<summary> Конфигурация интерфейсов и BGP для Leaf-1: </summary>

```
router bgp 65000
   router-id 10.255.1.1
   maximum-paths 4
   neighbor 10.255.0.1 remote-as 65000
   neighbor 10.255.0.1 update-source Loopback0
   neighbor 10.255.0.1 send-community
   neighbor 10.255.0.1 maximum-routes 12000 
   neighbor 10.255.0.2 remote-as 65000
   neighbor 10.255.0.2 update-source Loopback0
   neighbor 10.255.0.2 send-community
   neighbor 10.255.0.2 maximum-routes 12000 
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
   vlan 30
      rd auto
      route-target both 65000:10030
      redistribute learned
   !
   vlan 40
      rd auto
      route-target both 65000:10040
      redistribute learned
   !
   address-family evpn
      neighbor 10.255.0.1 activate
      neighbor 10.255.0.2 activate
   !
   address-family ipv4
      no neighbor 10.255.0.1 activate
      no neighbor 10.255.0.2 activate
   !
   vrf PROD
      rd 10.255.1.1:1
      route-target import evpn 65000:100999
      route-target export evpn 65000:100999
   !
   vrf Video
      rd 10.255.1.1:30
      route-target import evpn 65000:100930
      route-target export evpn 65000:100930
   !
   vrf VoIP
      rd 10.255.1.1:20
      route-target import evpn 65000:100920
      route-target export evpn 65000:100920
!
```
</details>


<details>

<summary> Конфигурация интерфейсов и BGP для Leaf-2: </summary>

```
router bgp 65000
   router-id 10.255.1.2
   maximum-paths 4
   neighbor 10.255.0.1 remote-as 65000
   neighbor 10.255.0.1 update-source Loopback0
   neighbor 10.255.0.1 send-community
   neighbor 10.255.0.1 maximum-routes 12000 
   neighbor 10.255.0.2 remote-as 65000
   neighbor 10.255.0.2 update-source Loopback0
   neighbor 10.255.0.2 send-community
   neighbor 10.255.0.2 maximum-routes 12000 
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
   vlan 30
      rd auto
      route-target both 65000:10030
      redistribute learned
   !
   vlan 40
      rd auto
      route-target both 65000:10040
      redistribute learned
   !
   address-family evpn
      neighbor 10.255.0.1 activate
      neighbor 10.255.0.2 activate
   !
   address-family ipv4
      no neighbor 10.255.0.1 activate
      no neighbor 10.255.0.2 activate
   !
   vrf PROD
      rd 10.255.1.2:1
      route-target import evpn 65000:100999
      route-target export evpn 65000:100999
   !
   vrf Video
      rd 10.255.1.2:30
      route-target import evpn 65000:100930
      route-target export evpn 65000:100930
   !
   vrf VoIP
      rd 10.255.1.2:20
      route-target import evpn 65000:100920
      route-target export evpn 65000:100920
!
```
</details>


<details>

<summary> Конфигурация интерфейсов и BGP для Leaf-3: </summary>

```
router bgp 65000
   router-id 10.255.2.1
   maximum-paths 4
   neighbor 10.255.0.1 remote-as 65000
   neighbor 10.255.0.1 update-source Loopback0
   neighbor 10.255.0.1 send-community
   neighbor 10.255.0.1 maximum-routes 12000 
   neighbor 10.255.0.2 remote-as 65000
   neighbor 10.255.0.2 update-source Loopback0
   neighbor 10.255.0.2 send-community
   neighbor 10.255.0.2 maximum-routes 12000 
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
   vlan 30
      rd auto
      route-target both 65000:10030
      redistribute learned
   !
   vlan 40
      rd auto
      route-target both 65000:10040
      redistribute learned
   !
   address-family evpn
      neighbor 10.255.0.1 activate
      neighbor 10.255.0.2 activate
   !
   address-family ipv4
      no neighbor 10.255.0.1 activate
      no neighbor 10.255.0.2 activate
   !
   vrf PROD
      rd 10.255.2.1:1
      route-target import evpn 65000:100999
      route-target export evpn 65000:100999
   !
   vrf Video
      rd 10.255.2.1:30
      route-target import evpn 65000:100930
      route-target export evpn 65000:100930
   !
   vrf VoIP
      rd 10.255.2.1:20
      route-target import evpn 65000:100920
      route-target export evpn 65000:100920
!
```
</details>



<details>

<summary> Конфигурация интерфейсов и BGP для Leaf-4: </summary>

```
router bgp 65000
   router-id 10.255.2.2
   maximum-paths 4
   neighbor 10.255.0.1 remote-as 65000
   neighbor 10.255.0.1 update-source Loopback0
   neighbor 10.255.0.1 send-community
   neighbor 10.255.0.1 maximum-routes 12000 
   neighbor 10.255.0.2 remote-as 65000
   neighbor 10.255.0.2 update-source Loopback0
   neighbor 10.255.0.2 send-community
   neighbor 10.255.0.2 maximum-routes 12000 
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
   vlan 30
      rd auto
      route-target both 65000:10030
      redistribute learned
   !
   vlan 40
      rd auto
      route-target both 65000:10040
      redistribute learned
   !
   address-family evpn
      neighbor 10.255.0.1 activate
      neighbor 10.255.0.2 activate
   !
   address-family ipv4
      no neighbor 10.255.0.1 activate
      no neighbor 10.255.0.2 activate
   !
   vrf PROD
      rd 10.255.2.2:1
      route-target import evpn 65000:100999
      route-target export evpn 65000:100999
   !
   vrf Video
      rd 10.255.2.2:30
      route-target import evpn 65000:100930
      route-target export evpn 65000:100930
   !
   vrf VoIP
      rd 10.255.2.2:20
      route-target import evpn 65000:100920
      route-target export evpn 65000:100920
!
```
</details>


<details>

<summary> Конфигурация интерфейсов и BGP для Leaf-5: </summary>

```
router bgp 65000
   router-id 10.255.3.1
   maximum-paths 4
   neighbor 10.255.0.1 remote-as 65000
   neighbor 10.255.0.1 update-source Loopback0
   neighbor 10.255.0.1 send-community
   neighbor 10.255.0.1 maximum-routes 12000 
   neighbor 10.255.0.2 remote-as 65000
   neighbor 10.255.0.2 update-source Loopback0
   neighbor 10.255.0.2 send-community
   neighbor 10.255.0.2 maximum-routes 12000 
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
   vlan 30
      rd auto
      route-target both 65000:10030
      redistribute learned
   !
   vlan 40
      rd auto
      route-target both 65000:10040
      redistribute learned
   !
   address-family evpn
      neighbor 10.255.0.1 activate
      neighbor 10.255.0.2 activate
   !
   address-family ipv4
      no neighbor 10.255.0.1 activate
      no neighbor 10.255.0.2 activate
   !
   vrf PROD
      rd 10.255.3.1:1
      route-target import evpn 65000:100999
      route-target export evpn 65000:100999
   !
   vrf Video
      rd 10.255.3.1:30
      route-target import evpn 65000:100930
      route-target export evpn 65000:100930
   !
   vrf VoIP
      rd 10.255.3.1:20
      route-target import evpn 65000:100920
      route-target export evpn 65000:100920
!
```
</details>





