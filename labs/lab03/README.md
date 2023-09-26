### Underlay IS-IS.

## Цель:

- Настроить IS-IS для Underlay сети


## В этой самостоятельной работе мы ожидаем, что вы самостоятельно:
  
1. настроить IS-IS в Underlay сети, для IP связанности между всеми устройствами NXOS


### Описание/Пошаговая инструкция выполнения домашнего задания:

1. Настроить IPv4 адресацию на всех устройствах.
2. Настроить IS-IS.
3. Проверить работу протокола IS-IS.  


## Схема стенда 
![img_1.png](img_1.png)

## Таблица адресов

| Device  | Interface | IP Address   | Subnet Mask     | Default Gateway |
|---------|-----------|--------------|-----------------|-----------------|
| Spine 1 | lo        | 192.168.0.1  | 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.0  | 255.255.255.254 |                 |
|         | E1/2      | 192.168.1.20 | 255.255.255.254 |                 |
|         | E1/3      | 192.168.1.30 | 255.255.255.254 |                 |
| Spine 1 | lo        | 192.168.0.2  | 255.255.255.255 |                 |
|         | E1/1      | 192.168.2.0  | 255.255.255.254 |                 |
|         | E1/2      | 192.168.2.20 | 255.255.255.254 |                 |
|         | E1/3      | 192.168.2.30 | 255.255.255.254 |                 |
| Leaf 1  | lo        | 192.168.0.11 | 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.1  | 255.255.255.254 |                 |
|         | E1/2      | 192.168.2.1  | 255.255.255.254 |                 |
|         | E1/3      | 192.168.10.1 | 255.255.255.0   |                 |
| Leaf 2  | lo        | 192.168.0.12 | 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.21 | 255.255.255.254 |                 |
|         | E1/2      | 192.168.2.21 | 255.255.255.254 |                 |
|         | E1/3      | 192.168.20.1 | 255.255.255.0   |                 |
| Leaf 3  | lo        | 192.168.0.13 | 255.255.255.255 |                 |
|         | E1/1      | 192.168.1.31 | 255.255.255.254 |                 |
|         | E1/2      | 192.168.2.31 | 255.255.255.254 |                 |
|         | E1/3      | 192.168.30.1 | 255.255.255.0   |                 |
|         | E1/4      | 192.168.40.1 | 255.255.255.0   |                 |
| VPC1    | eth0      | 192.168.10.2 | 255.255.255.0   | 192.168.10.1    |
| VPC2    | eth0      | 192.168.20.2 | 255.255.255.0   | 192.168.20.1    |
| VPC3    | eth0      | 192.168.30.2 | 255.255.255.0   | 192.168.30.1    |
| VPC4    | eth0      | 192.168.40.2 | 255.255.255.0   | 192.168.40.1    |


### [Файлы конфигураций устройст и сама работа выполненная в EVE-NG ](https://github.com/niknav83/Data_center_network_design/tree/main/labs/lab03/configs)

В данной работе применялса образ nxosv9k-9500-9.3.7

Логин и пароль: admin 

## Приступаем к настрйке сети:

### Настроим интерфейсы и IP адреса на всех устройствах Underlay-сети.



 Конфигурация интерфейсов для Spine-1:

```
interface Ethernet1/1
  medium p2p
  ip address 192.168.1.0/31
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.1.20/31
  no shutdown

interface Ethernet1/3
  medium p2p
  ip address 192.168.1.30/31
  no shutdown

interface loopback0
  ip address 192.168.0.1/32
!
```
 Конфигурация интерфейсов для Spine-2:

```
interface Ethernet1/1
  medium p2p
  ip address 192.168.2.0/31
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.20/31
  no shutdown

interface Ethernet1/3
  medium p2p
  ip address 192.168.2.30/31
  no shutdown

interface loopback0
  ip address 192.168.0.2/32
!
```

 Конфигурация интерфейсов для Leaf-1:

```
interface Ethernet1/1
  medium p2p
  ip address 192.168.1.1/31
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.1/31
  no shutdown

interface Ethernet1/3
  ip address 192.168.10.1/24
  no shutdown

interface loopback0
  ip address 192.168.0.11/32
!
```

 Конфигурация интерфейсов для Leaf-2:

```
interface Ethernet1/1
  medium p2p
  ip address 192.168.1.21/31
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.21/31
  no shutdown

interface Ethernet1/3
  ip address 192.168.20.1/24
  no shutdown

interface loopback0
  ip address 192.168.0.12/32
!
```

 Конфигурация интерфейсов для Leaf-3:

```
interface Ethernet1/1
  medium p2p
  ip address 192.168.1.31/31
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.31/31
  no shutdown

interface Ethernet1/3
  ip address 192.168.30.1/24
  no shutdown

interface Ethernet1/4
  ip address 192.168.40.1/24
  no shutdown

interface loopback0
  ip address 192.168.0.13/32
!
```

Конфигурация интерфейсов для VPC1:

```
ip 192.168.10.2 192.168.10.1 24
```

Конфигурация интерфейсов для VPC2:

```
ip 192.168.20.2 192.168.20.1 24
```

Конфигурация интерфейсов для VPC3:

```
ip 192.168.30.2 192.168.30.1 24
```

Конфигурация интерфейсов для VPC4:

```
ip 192.168.40.2 192.168.40.1 24
```


### Далее для общей связанности между всеми устройствами настроим протокол IS-IS.

На Nexus необходимо для начала включить функцию IS-IS

```
feature isis
```

 Конфигурация IS-IS для Spine-1:

```
feature isis

interface Ethernet1/1
  medium p2p
  ip address 192.168.1.0/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.1.20/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface Ethernet1/3
  medium p2p
  ip address 192.168.1.30/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface loopback0
  ip address 192.168.0.1/32
  ip router isis Underlay
  isis passive-interface level-1-2

router isis Underlay
  net 49.0001.1921.6800.0001.00
  is-type level-1
!
```
 Конфигурация IS-IS для Spine-2:

```
feature isis

interface Ethernet1/1
  medium p2p
  ip address 192.168.2.0/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.20/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface Ethernet1/3
  medium p2p
  ip address 192.168.2.30/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface loopback0
  ip address 192.168.0.2/32
  ip router isis Underlay
  isis passive-interface level-1-2

router isis Underlay
  net 49.0001.1921.6800.0002.00
  is-type level-1
!
```

 Конфигурация IS-IS для Leaf-1:

```
feature isis

interface Ethernet1/1
  medium p2p
  ip address 192.168.1.1/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.1/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface Ethernet1/3
  ip address 192.168.10.1/24
  ip router isis Underlay
  isis passive-interface level-1-2
  no shutdown

interface loopback0
  ip address 192.168.0.11/32
  ip router isis Underlay
  isis passive-interface level-1-2

router isis Underlay
  net 49.0001.1921.6800.0011.00
  is-type level-1
!
```

 Конфигурация IS-IS для Leaf-2:

```
feature isis

interface Ethernet1/1
  medium p2p
  ip address 192.168.1.21/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.21/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface Ethernet1/3
  ip address 192.168.20.1/24
  ip router isis Underlay
  isis passive-interface level-1-2
  no shutdown

interface loopback0
  ip address 192.168.0.12/32
  ip router isis Underlay
  isis passive-interface level-1-2

router isis Underlay
  net 49.0001.1921.6800.0012.00
  is-type level-1
!
```

 Конфигурация IS-IS для Leaf-3:

```
feature isis

interface Ethernet1/1
  medium p2p
  ip address 192.168.1.31/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface Ethernet1/2
  medium p2p
  ip address 192.168.2.31/31
  isis network point-to-point
  ip router isis Underlay
  no shutdown

interface Ethernet1/3
  ip address 192.168.30.1/24
  ip router isis Underlay
  isis passive-interface level-1-2
  no shutdown

interface Ethernet1/4
  ip address 192.168.40.1/24
  ip router isis Underlay
  isis passive-interface level-1-2
  no shutdown

interface loopback0
  ip address 192.168.0.13/32
  ip router isis Underlay
  isis passive-interface level-1-2

router isis Underlay
  net 49.0001.1921.6800.0013.00
  is-type level-1
!
```

## Проверяем работу протокола IS-IS:

 Вывод команды show isis hostname для Spine-1:

```
Spine-1# show isis hostname
IS-IS Process: Underlay dynamic hostname table VRF: default
  Level  System ID       Dynamic hostname
  1      1921.6800.0001* Spine-1
  1      1921.6800.0002  Spine-2
  1      1921.6800.0011  Liaf-1
  1      1921.6800.0012  Leaf-2
  1      1921.6800.0013  Leaf-3
```

 Вывод команды show isis hostname для Spine-2:

```
Spine-2# show isis hostname
IS-IS Process: Underlay dynamic hostname table VRF: default
  Level  System ID       Dynamic hostname
  1      1921.6800.0001  Spine-1
  1      1921.6800.0002* Spine-2
  1      1921.6800.0011  Liaf-1
  1      1921.6800.0012  Leaf-2
  1      1921.6800.0013  Leaf-3
```

 Вывод команды show isis hostname для Leaf-1:

```


```

 Вывод команды show isis hostname для Leaf-2:

```

```
 
 Вывод команды show isis hostname для Leaf-3:

```

```

 Вывод команды  для Spine-1:

```

```

 Вывод команды  для Spine-2:

```

```

 Вывод команды  для Leaf-1:

```


```

 Вывод команды  для Leaf-2:

```

```
 
 Вывод команды  для Leaf-3:

```

```






