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
| VPC1    | eth0      | 192.168.10.2 | 255.255.255.0   |                 |
| VPC2    | eth0      | 192.168.20.2 | 255.255.255.0   |                 |
| VPC3    | eth0      | 192.168.30.2 | 255.255.255.0   |                 |
| VPC4    | eth0      | 192.168.30.3 | 255.255.255.0   |                 |


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



!
```

 Конфигурация OSPF для Leaf-3:

```
feature ospf



!
```


