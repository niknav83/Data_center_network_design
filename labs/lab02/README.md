### Underlay OSPF.

## Цель:

- Настроить OSPF для Underlay сети


## В этой самостоятельной работе мы ожидаем, что вы самостоятельно:
  
1. Настроите OSPF в Underlay сети, для IP связанности между всеми устройствами NXOS


### Описание/Пошаговая инструкция выполнения домашнего задания:

1. Настроить IPv4 адресацию на всех устройствах.
2. Настроить OSPF.
3. Проверить работу протокола OSPF.  


## Схема стенда 
![img_1.png](https://github.com/niknav83/Data_center_network_design/blob/main/labs/lab01/img_3.png)

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
|         | E1/1      | 192.168.1.2  | 255.255.255.252 |                 |
|         | E1/2      | 192.168.2.2  | 255.255.255.252 |                 |
|         | E1/3      | 192.168.10.1 | 255.255.255.0   |                 |
| Leaf 2  | lo        | 192.168.0.12 | 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.22 | 255.255.255.252 |                 |
|         | E1/2      | 192.168.2.22 | 255.255.255.252 |                 |
|         | E1/3      | 192.168.20.1 | 255.255.255.252 |                 |
| Leaf 3  | lo        | 192.168.0.13 | 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.34 | 255.255.255.252 |                 |
|         | E1/2      | 192.168.2.34 | 255.255.255.255 |                 |
|         | E1/3      | 192.168.30.1 | 255.255.255.0   |                 |
|         | E1/3      | 192.168.30.1 | 255.255.255.0   |                 |
| VPC1    | eth0      | 192.168.10.2 | 255.255.255.0   | 192.168.10.1    |
| VPC2    | eth0      | 192.168.20.2 | 255.255.255.0   | 192.168.20.1    |
| VPC3    | eth0      | 192.168.30.2 | 255.255.255.0   | 192.168.30.1    |
| VPC4    | eth0      | 192.168.30.3 | 255.255.255.0   | 192.168.30.1    |


## Приступаем к настрйке сети:

### Настроим интерфейсы и IP адреса на всех устройствах Underlay-сети.

 Конфигурация интерфейсов для Spine-1:

```
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.1/30
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.1.21/30
  no shutdown

interface Ethernet1/3
  mtu 9216
  medium p2p
  ip address 192.168.1.33/30
  no shutdown

interface loopback0
  ip address 192.168.0.1/32
!
```
 Конфигурация интерфейсов для Spine-2:

```
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.2.1/30
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.21/30
  no shutdown

interface Ethernet1/3
  mtu 9216
  medium p2p
  ip address 192.168.2.33/30
  no shutdown

interface loopback0
  ip address 192.168.0.2/32
!
```

 Конфигурация интерфейсов для Leaf-1:

```
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.2/30
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.2/30
  no shutdown

interface Ethernet1/3
  mtu 9216
  ip address 192.168.10.1/24
  no shutdown

interface loopback0
  ip address 192.168.0.11/32
!
```

 Конфигурация интерфейсов для Leaf-2:

```
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.22/30
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.22/30
  no shutdown

interface Ethernet1/3
  mtu 9216
  ip address 192.168.20.1/24
  no shutdown

interface loopback0
  ip address 192.168.0.12/32
!
```

 Конфигурация интерфейсов для Leaf-3:

```
interface Vlan130
  no shutdown
  ip address 192.168.30.1/24

interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.34/30
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.34/30
  no shutdown

interface Ethernet1/3
  switchport
  switchport access vlan 130
  mtu 9216
  no shutdown

interface Ethernet1/4
  switchport
  switchport access vlan 130
  mtu 9216
  no shutdown

interface loopback0
  ip address 192.168.0.13/32
!
```

### Далее для общей связанности между всеми устройствами настроим протокол OSPF.

На Nexus необходимо для начала включить функцию OSPF

```
feature ospf
```

 Конфигурация OSPF для Spine-1:

```
feature ospf

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

interface loopback0
  ip address 192.168.0.1/32
  ip router ospf Underlay area 0.0.0.0

router ospf Underlay
  router-id 192.168.0.1
  passive-interface default
!
```
 Конфигурация OSPF для Spine-2:

```
feature ospf

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

interface loopback0
  ip address 192.168.0.2/32
  ip router ospf Underlay area 0.0.0.0

router ospf Underlay
  router-id 192.168.0.2
  passive-interface default
!
```

 Конфигурация OSPF для Leaf-1:

```
feature ospf

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
  mtu 9216
  ip address 192.168.10.1/24
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface loopback0
  ip address 192.168.0.11/32
  ip router ospf Underlay area 0.0.0.0

router ospf Underlay
  router-id 192.168.0.11
  passive-interface default
!
```

 Конфигурация OSPF для Leaf-2:

```
feature ospf

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
  mtu 9216
  ip address 192.168.20.1/24
  ip router ospf Underlay area 0.0.0.0
  no shutdown

interface loopback0
  ip address 192.168.0.12/32
  ip router ospf Underlay area 0.0.0.0

router ospf Underlay
  router-id 192.168.0.12
  passive-interface default
!
```

 Конфигурация OSPF для Leaf-3:

```
feature ospf

interface Vlan130
  no shutdown
  ip address 192.168.30.1/24
  ip router ospf Underlay area 0.0.0.0

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
  switchport access vlan 130
  mtu 9216
  no shutdown

interface Ethernet1/4
  switchport
  switchport access vlan 130
  mtu 9216
  no shutdown

interface loopback0
  ip address 192.168.0.13/32
  ip router ospf Underlay area 0.0.0.0

router ospf Underlay
  router-id 192.168.0.13
  passive-interface default
!
```

## Проверяем работу протокола OSPF:

 Вывод команды show ip ospf neighbors для Spine-1:

```
Spine-1# show ip ospf neighbors
 OSPF Process ID Underlay VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 192.168.0.11      1 FULL/ -          00:37:45 192.168.1.2     Eth1/1
 192.168.0.12      1 FULL/ -          00:24:20 192.168.1.22    Eth1/2
 192.168.0.13      1 FULL/ -          00:11:40 192.168.1.34    Eth1/3
```

 Вывод команды show ip ospf neighbors для Spine-2:

```
Spine-2# show ip ospf neighbors
 OSPF Process ID Underlay VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 192.168.0.11      1 FULL/ -          00:42:08 192.168.2.2     Eth1/1
 192.168.0.12      1 FULL/ -          00:28:41 192.168.2.22    Eth1/2
 192.168.0.13      1 FULL/ -          00:16:02 192.168.2.34    Eth1/3
```

 Вывод команды show ip ospf neighbors для Leaf-1:

```
Leaf-1# show ip ospf neighbors
 OSPF Process ID Underlay VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 192.168.0.1       1 FULL/ -          00:43:00 192.168.1.1     Eth1/1
 192.168.0.2       1 FULL/ -          00:43:00 192.168.2.1     Eth1/2

```

 Вывод команды show ip ospf neighbors для Leaf-2:

```
Leaf-2# show ip ospf neighbors
 OSPF Process ID Underlay VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 192.168.0.1       1 FULL/ -          00:30:27 192.168.1.21    Eth1/1
 192.168.0.2       1 FULL/ -          00:30:26 192.168.2.21    Eth1/2
```
 
 Вывод команды show ip ospf neighbors для Leaf-3:

```
Leaf-3# show ip ospf neighbors
 OSPF Process ID Underlay VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 192.168.0.1       1 FULL/ -          00:18:48 192.168.1.33    Eth1/1
 192.168.0.2       1 FULL/ -          00:18:49 192.168.2.33    Eth1/2
```


Вывод команды show ip int brief для Spine-1:

```
Spine-1# show ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo0                  192.168.0.1     protocol-up/link-up/admin-up
Eth1/1               192.168.1.1     protocol-up/link-up/admin-up
Eth1/2               192.168.1.21    protocol-up/link-up/admin-up
Eth1/3               192.168.1.33    protocol-up/link-up/admin-up

```

 Вывод команды show ip int brief для Spine-2:

```
Spine-2# show ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo0                  192.168.0.2     protocol-up/link-up/admin-up
Eth1/1               192.168.2.1     protocol-up/link-up/admin-up
Eth1/2               192.168.2.21    protocol-up/link-up/admin-up
Eth1/3               192.168.2.33    protocol-up/link-up/admin-up

```

 Вывод команды show ip int brief для Leaf-1:

```
Leaf-1# show ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo0                  192.168.0.11    protocol-up/link-up/admin-up
Eth1/1               192.168.1.2     protocol-up/link-up/admin-up
Eth1/2               192.168.2.2     protocol-up/link-up/admin-up
Eth1/3               192.168.10.1    protocol-up/link-up/admin-up


```

 Вывод команды show ip int brief для Leaf-2:

```
Leaf-2# show ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo0                  192.168.0.12    protocol-up/link-up/admin-up
Eth1/1               192.168.1.22    protocol-up/link-up/admin-up
Eth1/2               192.168.2.22    protocol-up/link-up/admin-up
Eth1/3               192.168.20.1    protocol-up/link-up/admin-up

```
 
 Вывод команды show ip int brief для Leaf-3:

```
Leaf-3# show ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Vlan130              192.168.30.1    protocol-up/link-up/admin-up
Lo0                  192.168.0.13    protocol-up/link-up/admin-up
Eth1/1               192.168.1.34    protocol-up/link-up/admin-up
Eth1/2               192.168.2.34    protocol-up/link-up/admin-up

```

Вывод команды show ip ospf database для Spine-1:

```
Spine-1# show ip ospf database
        OSPF Router with ID (192.168.0.1) (Process ID Underlay VRF default)

                Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age        Seq#       Checksum Link Count
192.168.0.1     192.168.0.1     1635       0x8000000b 0x205c   7
192.168.0.2     192.168.0.2     1637       0x8000000c 0x571b   7
192.168.0.11    192.168.0.11    1375       0x8000000d 0xf0ef   6
192.168.0.12    192.168.0.12    555        0x8000000b 0x2263   6
192.168.0.13    192.168.0.13    1592       0x80000006 0xcb81   6
```

 Вывод команды show ip ospf database для Spine-2:

```
Spine-2# show ip ospf database
        OSPF Router with ID (192.168.0.2) (Process ID Underlay VRF default)

                Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age        Seq#       Checksum Link Count
192.168.0.1     192.168.0.1     1659       0x8000000b 0x205c   7
192.168.0.2     192.168.0.2     1657       0x8000000c 0x571b   7
192.168.0.11    192.168.0.11    1397       0x8000000d 0xf0ef   6
192.168.0.12    192.168.0.12    577        0x8000000b 0x2263   6
192.168.0.13    192.168.0.13    1614       0x80000006 0xcb81   6
```

 Вывод команды show ip ospf database для Leaf-1:

```
Leaf-1# show ip ospf database
        OSPF Router with ID (192.168.0.11) (Process ID Underlay VRF default)

                Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age        Seq#       Checksum Link Count
192.168.0.1     192.168.0.1     1646       0x8000000b 0x205c   7
192.168.0.2     192.168.0.2     1646       0x8000000c 0x571b   7
192.168.0.11    192.168.0.11    1384       0x8000000d 0xf0ef   6
192.168.0.12    192.168.0.12    566        0x8000000b 0x2263   6
192.168.0.13    192.168.0.13    1603       0x80000006 0xcb81   6
```

 Вывод команды show ip ospf database для Leaf-2:

```
Leaf-2# show ip ospf database
        OSPF Router with ID (192.168.0.12) (Process ID Underlay VRF default)

                Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age        Seq#       Checksum Link Count
192.168.0.1     192.168.0.1     1673       0x8000000b 0x205c   7
192.168.0.2     192.168.0.2     1674       0x8000000c 0x571b   7
192.168.0.11    192.168.0.11    1413       0x8000000d 0xf0ef   6
192.168.0.12    192.168.0.12    592        0x8000000b 0x2263   6
192.168.0.13    192.168.0.13    1631       0x80000006 0xcb81   6
```
 
 Вывод команды show ip ospf database для Leaf-3:

```
Leaf-3# show ip ospf database
        OSPF Router with ID (192.168.0.13) (Process ID Underlay VRF default)

                Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age        Seq#       Checksum Link Count
192.168.0.1     192.168.0.1     1667       0x8000000b 0x205c   7
192.168.0.2     192.168.0.2     1670       0x8000000c 0x571b   7
192.168.0.11    192.168.0.11    1407       0x8000000d 0xf0ef   6
192.168.0.12    192.168.0.12    588        0x8000000b 0x2263   6
192.168.0.13    192.168.0.13    1623       0x80000006 0xcb81   6
```

Вывод команды show ip route ospf для Spine-1:

```

```

 Вывод команды show ip route ospf для Spine-2:

```

```

 Вывод команды show ip route ospf для Leaf-1:

```

```

 Вывод команды show ip route ospf для Leaf-2:

```

```
 
 Вывод команды show ip route ospf для Leaf-3:

```

```


















