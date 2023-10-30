### VxLAN. vPC.

## Цель:

- Настроить отказоустойчивое подключение клиентов с использованием vPC


## В этой самостоятельной работе мы ожидаем, что вы самостоятельно:
  
- Настроить отказоустойчивое подключение клиентов с использованием vPC


### Описание/Пошаговая инструкция выполнения домашнего задания:

- Underlay и Overlay настроены, согласно  лабораторной работе
- Задаем Loopback-адреса, согласно плану сети
- Назначаем на LEAF1 и LEAF2 интерфейсу Loopback 10 также одинаковый secondary IP-адрес - 
- Проверяем, чтобы все коммутаторы обменивались маршрутной инофрмацией
- Настраиваем VLAN ID и VNI на LEAF1 и LEAF2 идентично друг другу
- Создаём агрегированные логические интерфесы на LEAF1 и LEAF2:
    - Cоздаём Port-Channel 1 (Ethernet 8), переводим в режим mode trunk
    - создаём VLAN 10 и 20.
    - Интерфейсы переводим в mode trunk
- На коомутаторе vEOS6 также создаём Port-Channel 1 (Ethernet 1 и Ethernet 2), переводим в режим mode trunk
- На коомутаторе vEOS6 также создаём VLAN 10 и 20, задаём IP-адреса. Интерфейс будет использоваться в качестве клиента
- На LEAF1 и LEAF2 настраиваем LAG и проверяем его работу
- Выполняем общую проверку работосопобности Overlay-сети

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
| vEOS6   | vlan 10   | 192.168.10.10| 255.255.255.0   | 192.168.10.1    |
|         | vlan 20   | 192.168.20.20| 255.255.255.0   | 192.168.20.1    |
| R7      | eth0/0    | 192.168.20.2 | 255.255.255.0   | 192.168.20.1    |
| R8      | eth0/0    | 192.168.20.3 | 255.255.255.0   | 192.168.20.1    |
| R9      | eth0/0    | 192.168.10.2 | 255.255.255.0   | 192.168.10.1    |

### [Файлы конфигураций устройст и сама работа выполненная в EVE-NG ](https://github.com/niknav83/Data_center_network_design/tree/main/labs/lab07/configs)

В данной работе применялса образ Arista (vEOS, EOS-4.21.1.1F) 

Логин: admin 

## Приступаем к настрйке сети:

### Настроим интерфейсы, IP адреса и OSPF на всех устройствах Underlay-сети.

<details>

<summary> Конфигурация интерфейсов и OSPF для Spine-1: </summary>

```
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

interface Loopback0
   ip address 192.168.0.1/32
   ip ospf area 0.0.0.0

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

<summary>Конфигурация интерфейсов и OSPF для Spine-2: </summary>

```
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

interface Loopback0
   ip address 192.168.0.2/32
   ip ospf area 0.0.0.0

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

<summary> Конфигурация интерфейсов и OSPF для Leaf-1: </summary>

```
vlan 10,20

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

interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding PROD
   ip address virtual 192.168.20.1/24

interface Loopback0
   ip address 192.168.0.11/32
   ip ospf area 0.0.0.0
!
interface Loopback10
   ip address 192.168.0.111/32
   ip ospf area 0.0.0.0

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

<summary> Конфигурация интерфейсов и OSPF для Leaf-2: </summary>

```
vlan 10,20

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

interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding PROD
   ip address virtual 192.168.20.1/24

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

<summary> Конфигурация интерфейсов и OSPF для Leaf-3: </summary>

```
vlan 10,20

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

interface Ethernet7
   switchport access vlan 20
!
interface Ethernet8
   switchport access vlan 10
!
interface Loopback0
   ip address 192.168.0.13/32
   ip ospf area 0.0.0.0
!
interface Loopback10
   ip address 192.168.0.113/32
   ip ospf area 0.0.0.0

interface Vlan10
   vrf forwarding PROD
   ip address virtual 192.168.10.1/24
!
interface Vlan20
   vrf forwarding PROD
   ip address virtual 192.168.20.1/24

router ospf 1
   router-id 192.168.0.13
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   network 0.0.0.0/0 area 0.0.0.0
   max-lsa 12000
```
</details>


### Далее на всех устройствах произведем необходимые настройки.


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

### Проверка работоспособности EVPN / VxLAN. Проверяем соседство по L2VPN между устройствами и таблицу маршрутизации. На LEAF-коммутаторах проверяем также NVE Peers:


<details>
  
<summary>Вывод команд для Spine-1 :</summary>

```


```
```

```
</details>

<details>
  
<summary>Вывод команд для Spine-2 :</summary>

```

```
```

```
</details>

<details>
  
<summary>Вывод команд для Leaf-1 :</summary>

```


```
```

```
```

```
```


```

</details>

<details>
  
<summary>Вывод команд для Leaf-2 :</summary>

```

```
```

```
```

```
```



```
</details>

<details>
  
<summary>Вывод команд для Leaf-3 :</summary>

```

```
```

```
```


```
```

```

</details>




