### Underlay BGP.

## Цель:

- Настроить BGP для Underlay сети


## В этой самостоятельной работе мы ожидаем, что вы самостоятельно:
  
1. Настроить BGP в Underlay сети, для IP связанности между всеми устройствами NXOS


### Описание/Пошаговая инструкция выполнения домашнего задания:

1. Настроить IPv4 адресацию на всех устройствах.
2. Настроить BGP.
3. Проверить работу протокола BGP.  

## Схема стенда 
![img_1.png](img_1.PNG)


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


### [Файлы конфигураций устройст и сама работа выполненная в EVE-NG ](https://github.com/niknav83/Data_center_network_design/tree/main/labs/lab04/configs)

В данной работе применялса образ nxosv9k-9500-9.3.8

Логин и пароль: admin 

## Приступаем к настрйке сети:

### Настроим интерфейсы и IP адреса на всех устройствах Underlay-сети.

<details>

<summary> Конфигурация интерфейсов для Spine-1: </summary>

```
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.0/31
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.1.20/31
  no shutdown

interface Ethernet1/3
  mtu 9216
  medium p2p
  ip address 192.168.1.30/31
  no shutdown

interface loopback0
  ip address 192.168.0.1/32
```
</details>


<details>

<summary> Конфигурация интерфейсов для Spine-2: </summary>

```
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.2.0/31
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.20/31
  no shutdown

interface Ethernet1/3
  mtu 9216
  medium p2p
  ip address 192.168.2.30/31
  no shutdown

interface loopback0
  ip address 192.168.0.2/32
```
</details>


<details>

<summary> Конфигурация интерфейсов для Leaf-1: </summary>

```
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.1/31
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.1/31
  no shutdown

interface Ethernet1/3
  mtu 9216
  ip address 192.168.10.1/24
  no shutdown

interface loopback0
  ip address 192.168.0.11/32
```
</details>


<details>

<summary> Конфигурация интерфейсов для Leaf-2: </summary>

```
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.21/31
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.21/31
  no shutdown

interface Ethernet1/3
  mtu 9216
  ip address 192.168.20.1/24
  no shutdown

interface loopback0
  ip address 192.168.0.12/32
```
</details>


<details>

<summary> Конфигурация интерфейсов для Leaf-3: </summary>

```
interface Ethernet1/1
  mtu 9216
  medium p2p
  ip address 192.168.1.31/31
  no shutdown

interface Ethernet1/2
  mtu 9216
  medium p2p
  ip address 192.168.2.31/31
  no shutdown

interface Ethernet1/3
  mtu 9216
  ip address 192.168.30.1/24
  no shutdown

interface Ethernet1/4
  mtu 9216
  ip address 192.168.40.1/24
  no shutdown

interface loopback0
  ip address 192.168.0.13/32
```
</details>


<details>
  
<summary> Конфигурация интерфейсов для VPC1: </summary>

```
ip 192.168.10.2 192.168.10.1 24
```
</details>

<details>
  
<summary> Конфигурация интерфейсов для VPC2: </summary>

```
ip 192.168.20.2 192.168.20.1 24
```
</details>
  
<details>
    
<summary> Конфигурация интерфейсов для VPC3: </summary>

```
ip 192.168.30.2 192.168.30.1 24
```
</details>

<details>
  
<summary> Конфигурация интерфейсов для VPC4: </summary>

```
ip 192.168.40.2 192.168.40.1 24
```
</details>


### Далее для общей связанности между всеми устройствами настроим протокол BGP.

На Nexus необходимо для начала включить функцию BGP

```
feature bgp
```

Конфигурация BGP для Spine-1:

```
feature bgp
feature bfd

router bgp 65000
  router-id 192.168.0.1
  bestpath as-path multipath-relax
  reconnect-interval 10
  address-family ipv4 unicast
    network 192.168.0.1/32
    maximum-paths 10
  neighbor 192.168.1.1
    bfd
    remote-as 65001
    address-family ipv4 unicast
  neighbor 192.168.1.21
    bfd
    remote-as 65002
    address-family ipv4 unicast
  neighbor 192.168.1.31
    bfd
    remote-as 65003
    address-family ipv4 unicast
```

 Конфигурация BGP для Spine-2:

```
feature bgp
feature bfd

router bgp 65000
  router-id 192.168.0.2
  bestpath as-path multipath-relax
  reconnect-interval 10
  address-family ipv4 unicast
    network 192.168.0.2/32
    maximum-paths 10
  neighbor 192.168.2.1
    bfd
    remote-as 65001
    address-family ipv4 unicast
  neighbor 192.168.2.21
    bfd
    remote-as 65002
    address-family ipv4 unicast
  neighbor 192.168.2.31
    bfd
    remote-as 65003
    address-family ipv4 unicast
```

 Конфигурация BGP для Leaf-1:

```
feature bgp
feature bfd

route-map REDISTRIBUTE_CONNECTED permit 10
  match interface Ethernet1/3 

router bgp 65001
  router-id 192.168.0.11
  bestpath as-path multipath-relax
  reconnect-interval 10
  log-neighbor-changes
  address-family ipv4 unicast
    network 192.168.0.11/32
    redistribute direct route-map REDISTRIBUTE_CONNECTED
    maximum-paths 10
  template peer SPINES
    bfd
    remote-as 65000
    timers 3 9
    address-family ipv4 unicast
  neighbor 192.168.1.0
    inherit peer SPINES
  neighbor 192.168.2.0
    inherit peer SPINES
```

 Конфигурация BGP для Leaf-2:

```
feature bgp
feature bfd

route-map REDISTRIBUTE_CONNECTED permit 10
  match interface Ethernet1/3

router bgp 65002
  router-id 192.168.0.12
  bestpath as-path multipath-relax
  reconnect-interval 10
  log-neighbor-changes
  address-family ipv4 unicast
    network 192.168.0.12/32
    redistribute direct route-map REDISTRIBUTE_CONNECTED
    maximum-paths 10
  template peer SPINES
    bfd
    remote-as 65000
    timers 3 9
    address-family ipv4 unicast
  neighbor 192.168.1.20
    inherit peer SPINES
  neighbor 192.168.2.20
    inherit peer SPINES
```

 Конфигурация BGP для Leaf-3:

```
feature bgp
feature bfd

route-map REDISTRIBUTE_CONNECTED permit 10
  match interface Ethernet1/3 Ethernet1/4

router bgp 65003
  router-id 192.168.0.13
  bestpath as-path multipath-relax
  reconnect-interval 10
  log-neighbor-changes
  address-family ipv4 unicast
    network 192.168.0.13/32
    redistribute direct route-map REDISTRIBUTE_CONNECTED
    maximum-paths 10
  template peer SPINES
    bfd
    remote-as 65000
    timers 3 9
    address-family ipv4 unicast
  neighbor 192.168.1.30
    inherit peer SPINES
  neighbor 192.168.2.30
    inherit peer SPINES
```

## Проверяем работу протокола BGP:


<details>
  
<summary>Вывод команды show ip bgp summary:</summary>

Spine-1

```
Spine-1# show ip
ip     ipv6
Spine-1# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 192.168.0.1, local AS number 65000
BGP table version is 470, IPv4 Unicast config peers 3, capable peers 3
8 network entries and 8 paths using 1952 bytes of memory
BGP attribute entries [7/1204], BGP AS path entries [3/18]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.1.1     4 65001    4717    4727      470    0    0 00:26:05 2
192.168.1.21    4 65002    4810    4787      470    0    0 00:26:12 2
192.168.1.31    4 65003    3395    3411      470    0    0 00:31:57 3
```

Spine-2

```
Spine-2# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 192.168.0.2, local AS number 65000
BGP table version is 525, IPv4 Unicast config peers 3, capable peers 3
8 network entries and 8 paths using 1952 bytes of memory
BGP attribute entries [7/1204], BGP AS path entries [3/18]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.2.1     4 65001    4771    4741      525    0    0 00:16:45 2
192.168.2.21    4 65002    5395    5345      525    0    0 00:26:51 2
192.168.2.31    4 65003    3366    3356      525    0    0 00:26:52 3
```

Leaf-1

```
Leaf-1# show ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 192.168.0.11, local AS number 65001
BGP table version is 747, IPv4 Unicast config peers 2, capable peers 2
9 network entries and 14 paths using 2796 bytes of memory
BGP attribute entries [7/1204], BGP AS path entries [3/26]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.1.0     4 65000    4878    4723      747    0    0 00:27:17 6
192.168.2.0     4 65000    4879    4759      747    0    0 00:17:18 6
```

Leaf-2

```
Leaf-2# show  ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 192.168.0.12, local AS number 65002
BGP table version is 754, IPv4 Unicast config peers 2, capable peers 1
9 network entries and 14 paths using 2796 bytes of memory
BGP attribute entries [7/1204], BGP AS path entries [3/26]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.1.20    4 65000    4935    4816        0    0    0 00:00:18 Idle
192.168.2.20    4 65000    5325    5225      754    0    0 00:00:01 1
```

Leaf-3

```
Leaf-3# show  ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 192.168.0.13, local AS number 65003
BGP table version is 346, IPv4 Unicast config peers 2, capable peers 2
9 network entries and 13 paths using 2676 bytes of memory
BGP attribute entries [7/1204], BGP AS path entries [3/26]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.1.30    4 65000    1902    1821      346    0    0 00:00:28 5
192.168.2.30    4 65000    1842    1782      346    0    0 00:00:26 5
```
</details>

<details>
  
<summary>Вывод команды show ip bgp :</summary>

Spine-1

```
Spine-1# show ip bgp
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 481, Local Router ID is 192.168.0.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>l192.168.0.1/32     0.0.0.0                           100      32768 i
*>e192.168.0.11/32    192.168.1.1                                    0 65001 i
*>e192.168.0.12/32    192.168.1.21                                   0 65002 i
*>e192.168.0.13/32    192.168.1.31                                   0 65003 i
*>e192.168.10.0/24    192.168.1.1              0                     0 65001 ?
*>e192.168.20.0/24    192.168.1.21             0                     0 65002 ?
*>e192.168.30.0/24    192.168.1.31             0                     0 65003 ?
*>e192.168.40.0/24    192.168.1.31             0                     0 65003 ?
```

Spine-2

```
Spine-2# show ip bgp
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 538, Local Router ID is 192.168.0.2
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>l192.168.0.2/32     0.0.0.0                           100      32768 i
*>e192.168.0.11/32    192.168.2.1                                    0 65001 i
*>e192.168.0.12/32    192.168.2.21                                   0 65002 i
*>e192.168.0.13/32    192.168.2.31                                   0 65003 i
*>e192.168.10.0/24    192.168.2.1              0                     0 65001 ?
*>e192.168.20.0/24    192.168.2.21             0                     0 65002 ?
*>e192.168.30.0/24    192.168.2.31             0                     0 65003 ?
*>e192.168.40.0/24    192.168.2.31             0                     0 65003 ?
```

Leaf-1

```
Leaf-1# show  ip bgp
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 768, Local Router ID is 192.168.0.11
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>e192.168.0.1/32     192.168.1.0                                    0 65000 i
*>e192.168.0.2/32     192.168.2.0                                    0 65000 i
*>l192.168.0.11/32    0.0.0.0                           100      32768 i
*|e192.168.0.12/32    192.168.2.0                                    0 65000 65002 i
*>e                   192.168.1.0                                    0 65000 65002 i
*|e192.168.0.13/32    192.168.2.0                                    0 65000 65003 i
*>e                   192.168.1.0                                    0 65000 65003 i
*>r192.168.10.0/24    0.0.0.0                  0        100      32768 ?
*|e192.168.20.0/24    192.168.2.0                                    0 65000 65002 ?
*>e                   192.168.1.0                                    0 65000 65002 ?
*|e192.168.30.0/24    192.168.2.0                                    0 65000 65003 ?
*>e                   192.168.1.0                                    0 65000 65003 ?
*|e192.168.40.0/24    192.168.2.0                                    0 65000 65003 ?
*>e                   192.168.1.0                                    0 65000 65003 ?
```

Leaf-2

```
Leaf-2# show ip bgp
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 766, Local Router ID is 192.168.0.12
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>e192.168.0.1/32     192.168.1.20                                   0 65000 i
*>e192.168.0.2/32     192.168.2.20                                   0 65000 i
*|e192.168.0.11/32    192.168.2.20                                   0 65000 65001 i
*>e                   192.168.1.20                                   0 65000 65001 i
*>l192.168.0.12/32    0.0.0.0                           100      32768 i
*|e192.168.0.13/32    192.168.2.20                                   0 65000 65003 i
*>e                   192.168.1.20                                   0 65000 65003 i
*|e192.168.10.0/24    192.168.2.20                                   0 65000 65001 ?
*>e                   192.168.1.20                                   0 65000 65001 ?
*>r192.168.20.0/24    0.0.0.0                  0        100      32768 ?
*|e192.168.30.0/24    192.168.2.20                                   0 65000 65003 ?
*>e                   192.168.1.20                                   0 65000 65003 ?
*|e192.168.40.0/24    192.168.2.20                                   0 65000 65003 ?
*>e                   192.168.1.20                                   0 65000 65003 ?
```

Leaf-3

```
Leaf-3# show ip bgp
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 346, Local Router ID is 192.168.0.13
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - best2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>e192.168.0.1/32     192.168.1.30                                   0 65000 i
*>e192.168.0.2/32     192.168.2.30                                   0 65000 i
*|e192.168.0.11/32    192.168.2.30                                   0 65000 65001 i
*>e                   192.168.1.30                                   0 65000 65001 i
*|e192.168.0.12/32    192.168.2.30                                   0 65000 65002 i
*>e                   192.168.1.30                                   0 65000 65002 i
*>l192.168.0.13/32    0.0.0.0                           100      32768 i
*|e192.168.10.0/24    192.168.2.30                                   0 65000 65001 ?
*>e                   192.168.1.30                                   0 65000 65001 ?
*|e192.168.20.0/24    192.168.2.30                                   0 65000 65002 ?
*>e                   192.168.1.30                                   0 65000 65002 ?
*>r192.168.30.0/24    0.0.0.0                  0        100      32768 ?
*>r192.168.40.0/24    0.0.0.0                  0        100      32768 ?
```

</details>


<details>
  
<summary>Проверяем работу протокола BGP c VPC1</summary>


```
VPCS> trace 192.168.20.2
trace to 192.168.20.2, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   4.310 ms  2.555 ms  2.040 ms
 2   192.168.1.0   8.882 ms  8.532 ms  9.048 ms
 3   192.168.1.21   23.194 ms  18.865 ms  15.733 ms
 4   *192.168.20.2   17.505 ms (ICMP type:3, code:3, Destination port unreachable)
```
```
VPCS> trace 192.168.30.2
trace to 192.168.30.2, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   17.805 ms  3.482 ms  2.910 ms
 2   192.168.2.0   34.159 ms  9.189 ms  9.953 ms
 3   192.168.2.31   41.007 ms  16.350 ms  17.324 ms
 4   *192.168.30.2   176.200 ms (ICMP type:3, code:3, Destination port unreachable)
```
```
VPCS> trace 192.168.40.2
trace to 192.168.40.2, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   556.853 ms  4.271 ms  6.732 ms
 2   192.168.1.0   118.544 ms  14.024 ms  20.746 ms
 3   192.168.1.31   121.204 ms  87.945 ms  82.185 ms
 4   *192.168.40.2   27.213 ms (ICMP type:3, code:3, Destination port unreachable)
```
</details>


<details>
  
<summary>Проверяем работу протокола BGP c VPC2</summary>

```
VPCS> trace 192.168.10.2
trace to 192.168.10.2, 8 hops max, press Ctrl+C to stop
 1   192.168.20.1   6.563 ms  2.608 ms  2.388 ms
 2   192.168.1.20   9.187 ms  9.580 ms  9.075 ms
 3   192.168.1.1   17.433 ms  19.850 ms  14.354 ms
 4   *192.168.10.2   19.470 ms (ICMP type:3, code:3, Destination port unreachable)
```
```
VPCS> trace 192.168.30.2
trace to 192.168.30.2, 8 hops max, press Ctrl+C to stop
 1   192.168.20.1   25.520 ms  3.631 ms  3.570 ms
 2   192.168.1.20   30.366 ms  12.539 ms  8.708 ms
 3   192.168.1.31   37.788 ms  18.830 ms  17.902 ms
 4   *192.168.30.2   37.535 ms (ICMP type:3, code:3, Destination port unreachable)
```
```
VPCS> trace 192.168.40.2
trace to 192.168.40.2, 8 hops max, press Ctrl+C to stop
 1   192.168.20.1   17.410 ms  3.166 ms  3.266 ms
 2   192.168.1.20   16.140 ms  9.485 ms  9.919 ms
 3   192.168.1.31   21.156 ms  16.812 ms  16.359 ms
 4   *192.168.40.2   27.120 ms (ICMP type:3, code:3, Destination port unreachable)
```
</details>

<details>
  
<summary>Проверяем работу протокола BGP c VPC3</summary>

```
VPCS> trace 192.168.10.2
trace to 192.168.10.2, 8 hops max, press Ctrl+C to stop
 1   192.168.30.1   4.696 ms  3.340 ms  2.695 ms
 2   192.168.1.30   13.151 ms  7.634 ms  12.126 ms
 3   192.168.1.1   23.183 ms  13.899 ms  14.657 ms
 4   *192.168.10.2   22.178 ms (ICMP type:3, code:3, Destination port unreachable)
```
```
VPCS> trace 192.168.20.2
trace to 192.168.20.2, 8 hops max, press Ctrl+C to stop
 1   192.168.30.1   8.810 ms  4.125 ms  3.650 ms
 2   192.168.2.30   19.170 ms  12.517 ms  13.197 ms
 3   192.168.2.21   35.171 ms  19.884 ms  18.574 ms
 4   *192.168.20.2   62.428 ms (ICMP type:3, code:3, Destination port unreachable)
```
```
VPCS> trace 192.168.40.2
trace to 192.168.40.2, 8 hops max, press Ctrl+C to stop
 1   192.168.30.1   15.907 ms  3.016 ms  2.712 ms
 2   *192.168.40.2   25.709 ms (ICMP type:3, code:3, Destination port unreachable)
```
</details>

<details>
  
<summary>Проверяем работу протокола BGP c VPC4</summary>

```
VPCS> tracer 192.168.10.2
trace to 192.168.10.2, 8 hops max, press Ctrl+C to stop
 1   192.168.40.1   54.723 ms  6.486 ms  5.207 ms
 2   192.168.1.30   32.646 ms  15.992 ms  12.487 ms
 3   192.168.1.1   33.962 ms  15.948 ms  16.261 ms
 4   *192.168.10.2   19.860 ms (ICMP type:3, code:3, Destination port unreachable)
```

```
VPCS> tracer 192.168.20.2
trace to 192.168.20.2, 8 hops max, press Ctrl+C to stop
 1   192.168.40.1   16.588 ms  3.727 ms  3.900 ms
 2   192.168.2.30   30.168 ms  12.853 ms  12.358 ms
 3   192.168.2.21   60.215 ms  22.323 ms  25.593 ms
 4   *192.168.20.2   40.418 ms (ICMP type:3, code:3, Destination port unreachable)
```

```
VPCS> tracer 192.168.30.2
trace to 192.168.30.2, 8 hops max, press Ctrl+C to stop
 1   192.168.40.1   26.546 ms  3.906 ms  16.005 ms
 2   *192.168.30.2   34.029 ms (ICMP type:3, code:3, Destination port unreachable)
```

</details>










