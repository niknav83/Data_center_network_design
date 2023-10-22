### VxLAN. EVPN L3.

## Цель:




## В этой самостоятельной работе мы ожидаем, что вы самостоятельно:
  



### Описание/Пошаговая инструкция выполнения домашнего задания:



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
|         | E1/3      | 192.168.10.1 | 255.255.255.0   |                 |
|         | E1/4      | 192.168.20.1 | 255.255.255.0   |                 |
| Leaf 2  | lo        | 192.168.0.12 | 255.255.255.255 |                 |
|         | lo10      | 192.168.0.112| 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.22 | 255.255.255.252 |                 |
|         | E1/2      | 192.168.2.22 | 255.255.255.252 |                 |
|         | E1/3      | 192.168.10.1 | 255.255.255.252 |                 |
|         | E1/4      | 192.168.20.1 | 255.255.255.252 |                 |
| Leaf 3  | lo        | 192.168.0.13 | 255.255.255.252 |                 |
|         | lo10      | 192.168.0.113| 255.255.255.252 |                 |
|         | E1/1      | 192.168.1.34 | 255.255.255.252 |                 |
|         | E1/2      | 192.168.2.34 | 255.255.255.255 |                 |
|         | E1/3      | 192.168.10.1 | 255.255.255.0   |                 |
|         | E1/4      | 192.168.20.1 | 255.255.255.0   |                 |
| VPC1    | eth0      | 192.168.10.2 | 255.255.255.0   | 192.168.10.1    |
| VPC2    | eth0      | 192.168.20.2 | 255.255.255.0   | 192.168.20.1    |
| VPC3    | eth0      | 192.168.10.3 | 255.255.255.0   | 192.168.10.1    |
| VPC4    | eth0      | 192.168.20.3 | 255.255.255.0   | 192.168.20.1    |
| VPC5    | eth0      | 192.168.10.4 | 255.255.255.0   | 192.168.10.1    |
| VPC6    | eth0      | 192.168.20.4 | 255.255.255.0   | 192.168.20.1    |

### [Файлы конфигураций устройст и сама работа выполненная в EVE-NG ](https://github.com/niknav83/Data_center_network_design/tree/main/labs/lab06/configs)

В данной работе применялса образ nxosv9k-9500-9.3.8

Логин и пароль: admin 

## Приступаем к настрйке сети:

### Настроим интерфейсы и IP адреса на всех устройствах Underlay-сети.

<details>

<summary> Конфигурация интерфейсов и OSPF для Spine-1: </summary>

```
interface loopback0
  ip address 192.168.0.1/32
  ip router ospf Underlay area 0.0.0.0

interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.1/30
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.1.21/30
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/3
  mtu 9216
  medium p2p
  ip address 192.168.1.33/30
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

<summary>Конфигурация интерфейсов и OSPF для Spine-2: </summary>

```
interface loopback0
  ip address 192.168.0.2/32
  ip router ospf Underlay area 0.0.0.0

interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.2.1/30
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.21/30
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/3
  mtu 9216
  medium p2p
  ip address 192.168.2.33/30
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

<summary> Конфигурация интерфейсов и OSPF для Leaf-1: </summary>

```
interface loopback0
  ip address 192.168.0.11/32
  ip router ospf Underlay area 0.0.0.0

interface loopback10
  ip address 192.168.0.111/32
  ip router ospf Underlay area 0.0.0.0

interface Vlan10
  no shutdown
  ip address 192.168.10.1/24
 
interface Vlan20
  no shutdown
  ip address 192.168.20.1/24
  
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.2/30
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.2/30
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
  router-id 192.168.0.11
  passive-interface default
```
</details>


<details>

<summary> Конфигурация интерфейсов и OSPF для Leaf-2: </summary>

```
interface loopback0
  ip address 192.168.0.12/32
  ip router ospf Underlay area 0.0.0.0

interface loopback10
  ip address 192.168.0.112/32
  ip router ospf Underlay area 0.0.0.0

interface Vlan10
  no shutdown
  ip address 192.168.10.1/24
  
interface Vlan20
  no shutdown
  ip address 192.168.20.1/24
 
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.22/30
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.22/30
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
  router-id 192.168.0.12
  passive-interface default
```
</details>


<details>

<summary> Конфигурация интерфейсов и OSPF для Leaf-3: </summary>

```
interface loopback0
  ip address 192.168.0.13/32
  ip router ospf Underlay area 0.0.0.0

interface loopback10
  ip address 192.168.0.113/32
  ip router ospf Underlay area 0.0.0.0

interface Vlan10
  no shutdown
  ip address 192.168.10.1/24

interface Vlan20
  no shutdown
  ip address 192.168.20.1/24

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback10
  member vni 10000
    ingress-replication protocol bgp
  member vni 20000
    ingress-replication protocol bgp
  member vni 99000 associate-vrf

interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.34/30
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.34/30
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

На Nexus необходимо для начала включить функции

```

```

Конфигурация для Spine-1:

```

```

 Конфигурация для Spine-2:

```

```

 Конфигурация для Leaf-1:

```

```

 Конфигурация для Leaf-2:

```

```

 Конфигурация для Leaf-3:

```

```

### Проверка работоспособности EVPN / VxLAN. Проверяем соседство по L2VPN между устройствами и таблицу маршрутизации Route Distinguisher. На LEAF-коммутаторах проверяем также NVE Peers:


<details>
  
<summary>Вывод команды :</summary>

Spine-1

```

```
```


```

Spine-2

```

```
```


```

Leaf-1

```


```
```


```

Leaf-2

```

```
```

```

Leaf-3

```

```
```

```

</details>



