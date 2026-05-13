


# Задание:

1. настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.




# Решение:

1. [Создание, документация сети](#создание-сети)
2. [Проверка IP связности](#проверка-доступности)


    
### Создание сети:
Для создания тестовой среды использованы:
- ПО Pnetlab для создания виртуального стенда;
- коммутатор Arista ver. 4.29.2F в роли SPINE в количестве 2 шт (DC01-SSW01-02);
- коммутатор Arista ver. 4.29.2F в роли LEAF в количестве 3 шт (DC01-LSW01-03);
- коммутатор стороннего вендора в роли LEAF в количестве 1 шт (DC01-LSW01-04);
- коммутатор стороннего вендора в роли LEAF в количестве 3 шт (DC01-LSW01-05).

Подключение было выполнено согласно прилагаемой схеме:


![Схема](./images/scheme.png "Визуализация")

#### Описание:
- 10.255.255.X/24 - SPINE loopback IP address, где X - номер SPINE
- 10.255.254.X/24 - LEAF loopback IP address, где X - номер LEAF
- 10.255.253.0/24 - линковая подсеть для связи SPINE-LEAF. Используются /31 подсети. Четный номер - SPINE, нечетный LEAF
- 10.0.X.0/17 - сервисы, где X соотвествует VLAN ID из диапазона 1-99
- AS64512 - номер автономной системы для SPINE'ов
- AS4200000XXX - номера автономных систем для LEAF'ов, где XXX - соотвествует номеру LEAF'а с добавлением нулей в начале
- Хосты имеют именя SRVXX-YZ, где XX - номер VLAN из дипазона 01-99, Y - номер LEAF, Z - порядковый номер, начиная с 0. IP-адрес хоста при этом будет в формате 10.0.XX.YZ/24.
- Используем схему VLAN-AWARE, RT 64512:1 (ASN SPINE:1)
- VNI в формате XXYYYY, где XX - номер ЦОД, YYYY - номер VLAN. Для L3 используем YYYY 4096.


#### Конфигурация сетевого оборудования:
<details>
<summary><b>SPINE 1:</b></summary>

```
router bgp 64512
   router-id 10.255.255.1
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   bgp listen range 10.255.253.0/24 peer-group dyn_leafs peer-filter pf_leafs
   neighbor dyn_leafs peer group
   neighbor dyn_leafs bfd
   neighbor dyn_leafs send-community extended
   !
   vlan-aware-bundle VLAN-AWARE
      rd 64512:1
      route-target both 64512:1
      redistribute learned
      vlan 10,20
   !
   address-family evpn
      neighbor dyn_leafs activate
   !
   address-family ipv4
      neighbor dyn_leafs activate
      redistribute connected route-map from_connected_to_bgp
!

```
</details>
<details>
<summary><b>SPINE 2:</b></summary>

```
router bgp 64512
   router-id 10.255.255.2
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   bgp listen range 10.255.253.0/24 peer-group dyn_leafs peer-filter pf_leafs
   neighbor dyn_leafs peer group
   neighbor dyn_leafs bfd
   neighbor dyn_leafs send-community extended
   !
   vlan-aware-bundle VLAN-AWARE
      rd 64512:2
      route-target both 64512:2
      redistribute learned
      vlan 10,20
   !
   address-family evpn
      neighbor dyn_leafs activate
   !
   address-family ipv4
      neighbor dyn_leafs activate
      redistribute connected route-map from_connected_to_bgp
!
```
</details>

На SPINE'ах нет ни VLAN (кроме VLAN 1), ни интерфейсов vxlan... 
На LEAF'ах в дополнение к вышеописанному добавляем интерфейсы VXLAN

<details>
<summary><b>LEAF 1:</b></summary>

```
vlan 10,20
!
vrf instance PROD
!
interface Loopback0
   ip address 10.255.254.1/32
   isis enable dc01
!
interface Loopback10
   ip address 10.255.254.101/32
!
interface Vlan10
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vxlan1
   vxlan source-interface Loopback10
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf PROD vni 14096
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf PROD
!
router bgp 4200000001
   router-id 10.255.254.1
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   neighbor spines peer group
   neighbor spines remote-as 64512
   neighbor spines bfd
   neighbor spines send-community extended
   neighbor 10.255.253.100 peer group spines
   neighbor 10.255.253.200 peer group spines
   !
   vlan-aware-bundle VLAN-AWARE
      rd 4200000001:1
      route-target both 64512:1
      redistribute learned
      vlan 10,20
   !
   address-family evpn
      neighbor spines activate
   !
   address-family ipv4
      neighbor spines activate
      redistribute connected route-map from_connected_to_bgp
   !
   vrf PROD
      rd 4200000001:4096
      route-target import evpn 64512:4096
      route-target export evpn 64512:4096
      redistribute connected
!

```
</details>




<details>
<summary><b>LEAF 2:</b></summary>

```

vlan 10,20
!
vrf instance PROD
!
interface Loopback0
   ip address 10.255.254.2/32
   isis enable dc01
!
interface Loopback10
   ip address 10.255.254.102/32
!
interface Vlan10
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vxlan1
   vxlan source-interface Loopback10
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf PROD vni 14096
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf PROD
!
router bgp 4200000002
   router-id 10.255.254.2
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   neighbor spines peer group
   neighbor spines remote-as 64512
   neighbor spines bfd
   neighbor spines send-community extended
   neighbor 10.255.253.102 peer group spines
   neighbor 10.255.253.202 peer group spines
   !
   vlan-aware-bundle VLAN-AWARE
      rd 4200000002:1
      route-target both 64512:1
      redistribute learned
      vlan 10,20
   !
   address-family evpn
      neighbor spines activate
   !
   address-family ipv4
      neighbor spines activate
      redistribute connected route-map from_connected_to_bgp
   !
   vrf PROD
      rd 4200000002:4096
      route-target import evpn 64512:4096
      route-target export evpn 64512:4096
      redistribute connected
!



```
</details>




<details>
<summary><b>LEAF 3:</b></summary>
```
vlan 10,20
!
vrf instance PROD
!
interface Loopback0
   ip address 10.255.254.3/32
   isis enable dc01
!
interface Loopback10
   ip address 10.255.254.103/32
!
interface Vlan10
   vrf PROD
   ip address virtual 10.0.10.1/24
!
interface Vlan20
   vrf PROD
   ip address 10.0.20.1/24
!
interface Vxlan1
   vxlan source-interface Loopback10
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf PROD vni 14096
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing
ip routing vrf PROD
!
router bgp 4200000003
   router-id 10.255.254.3
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   neighbor spines peer group
   neighbor spines remote-as 64512
   neighbor spines bfd
   neighbor spines send-community extended
   neighbor 10.255.253.104 peer group spines
   neighbor 10.255.253.204 peer group spines
   !
   vlan-aware-bundle VLAN-AWARE
      rd 4200000003:1
      route-target both 64512:1
      redistribute learned
      vlan 10,20
   !
   address-family evpn
      neighbor spines activate
   !
   address-family ipv4
      neighbor spines activate
      redistribute connected route-map from_connected_to_bgp
   !
   vrf PROD
      rd 4200000003:4096
      route-target import evpn 64512:4096
      route-target export evpn 64512:4096
      redistribute connected
!
```
</details>

#### Конфигурация хостов:
<details>
<summary><b>SRV10-10:</b></summary>
```
NAME   IP/MASK              GATEWAY           MAC                DNS
VPCS1  10.0.10.10/24        10.0.10.1         00:50:79:66:68:29  10.1.0.2
```
</details>
<details>
<summary><b>SRV10-30:</b></summary>
```
NAME   IP/MASK              GATEWAY           MAC                DNS
VPCS1  10.0.10.30/24        10.0.10.1         00:50:79:66:68:2b  10.1.0.10
```
</details>
<details>
<summary><b>SRV20-30:</b></summary>
```
NAME   IP/MASK              GATEWAY           MAC                DNS
VPCS1  10.0.20.30/24        10.0.20.1         00:50:79:66:68:38  10.0.1.10
```
</details>



📥 [Скачать](./configs)  файлы лабы в текстовом формате 


### Проверка доступности: 
Проверка выполняется на каждом из коммутаторов по следующим критериям:
 - просмотр BGP топологии и соседей;
 - просмотр таблицы маршрутизации на каждом коммутаторе;
 - проверка связности посредством icmp echo request.

Под катом находится пример проверки, выполненной на LEAF 1. 
<details>
<summary><b>LEAF 1:</b></summary>
```
DC01-LSW001#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.255.254.1, local AS number 4200000001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 4200000001:1 mac-ip 10010 0050.7966.6829
                                 -                     -       -       0       i
 * >      RD: 4200000001:1 mac-ip 10010 0050.7966.6829 10.0.10.10
                                 -                     -       -       0       i
 * >Ec    RD: 4200000003:1 mac-ip 10010 0050.7966.682b
                                 10.255.254.103        -       100     0       64512 4200000003 i
 *  ec    RD: 4200000003:1 mac-ip 10010 0050.7966.682b
                                 10.255.254.103        -       100     0       64512 4200000003 i
 * >Ec    RD: 4200000003:1 mac-ip 10010 0050.7966.682b 10.0.10.30
                                 10.255.254.103        -       100     0       64512 4200000003 i
 *  ec    RD: 4200000003:1 mac-ip 10010 0050.7966.682b 10.0.10.30
                                 10.255.254.103        -       100     0       64512 4200000003 i
 * >Ec    RD: 4200000003:1 mac-ip 10020 0050.7966.6838
                                 10.255.254.103        -       100     0       64512 4200000003 i
 *  ec    RD: 4200000003:1 mac-ip 10020 0050.7966.6838
                                 10.255.254.103        -       100     0       64512 4200000003 i
 * >Ec    RD: 4200000003:1 mac-ip 10020 0050.7966.6838 10.0.20.30
                                 10.255.254.103        -       100     0       64512 4200000003 i
 *  ec    RD: 4200000003:1 mac-ip 10020 0050.7966.6838 10.0.20.30
                                 10.255.254.103        -       100     0       64512 4200000003 i
 * >      RD: 4200000001:1 imet 10010 10.255.254.101
                                 -                     -       -       0       i
 * >Ec    RD: 4200000003:1 imet 10010 10.255.254.103
                                 10.255.254.103        -       100     0       64512 4200000003 i
 *  ec    RD: 4200000003:1 imet 10010 10.255.254.103
                                 10.255.254.103        -       100     0       64512 4200000003 i
 * >      RD: 4200000001:1 imet 10020 10.255.254.101
                                 -                     -       -       0       i
 * >Ec    RD: 4200000003:1 imet 10020 10.255.254.103
                                 10.255.254.103        -       100     0       64512 4200000003 i
 *  ec    RD: 4200000003:1 imet 10020 10.255.254.103
                                 10.255.254.103        -       100     0       64512 4200000003 i
 * >      RD: 4200000001:4096 ip-prefix 10.0.10.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 4200000003:4096 ip-prefix 10.0.10.0/24
                                 10.255.254.103        -       100     0       64512 4200000003 i
 *  ec    RD: 4200000003:4096 ip-prefix 10.0.10.0/24
                                 10.255.254.103        -       100     0       64512 4200000003 i
 * >Ec    RD: 4200000003:4096 ip-prefix 10.0.20.0/24
                                 10.255.254.103        -       100     0       64512 4200000003 i
 *  ec    RD: 4200000003:4096 ip-prefix 10.0.20.0/24
                                 10.255.254.103        -       100     0       64512 4200000003 i


DC01-LSW001#show vxlan vni 10010
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10010       10         static       Ethernet4       untagged
                                    Vxlan1          10

VNI to dynamic VLAN Mapping for Vxlan1
VNI       VLAN       VRF       Source
--------- ---------- --------- ------------

DC01-LSW001#show vxlan vni 10020
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10020       20         static       Vxlan1          20

VNI to dynamic VLAN Mapping for Vxlan1
VNI       VLAN       VRF       Source
--------- ---------- --------- ------------

DC01-LSW001#show vxlan vni 14096
VNI to VLAN Mapping for Vxlan1
VNI       VLAN       Source       Interface       802.1Q Tag
--------- ---------- ------------ --------------- ----------

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF        Source
----------- ---------- ---------- ------------
14096       4093       PROD       evpn

DC01-LSW001#show mac address-table
          Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports      Moves   Last Move
----    -----------       ----        -----      -----   ---------
   1    0000.0000.0001    STATIC      Cpu
  10    0000.0000.0001    STATIC      Cpu
  10    0050.7966.6829    DYNAMIC     Et4        1       0:04:41 ago
  10    0050.7966.682b    DYNAMIC     Vx1        1       0:04:44 ago
  20    0000.0000.0001    STATIC      Cpu
  20    0050.7966.6838    DYNAMIC     Vx1        1       0:04:21 ago
4093    0000.0000.0001    STATIC      Cpu
4093    505a.5203.e5a1    DYNAMIC     Vx1        1       2:54:44 ago
Total Mac Addresses for this criterion: 8

          Multicast Mac Address Table
------------------------------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       ----        -----

```
</details>



Под катом находится пример проверки, выполненной на SPINE 1. 
<details>
<summary><b>SPINE 1:</b></summary>
```
DC01-SSW001#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.255.255.1, local AS number 64512
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 4200000001:1 mac-ip 10010 0050.7966.6829
                                 10.255.254.101        -       100     0       4200000001 i
 * >      RD: 4200000001:1 mac-ip 10010 0050.7966.6829 10.0.10.10
                                 10.255.254.101        -       100     0       4200000001 i
 * >      RD: 4200000003:1 mac-ip 10010 0050.7966.682b
                                 10.255.254.103        -       100     0       4200000003 i
 * >      RD: 4200000003:1 mac-ip 10010 0050.7966.682b 10.0.10.30
                                 10.255.254.103        -       100     0       4200000003 i
 * >      RD: 4200000003:1 mac-ip 10020 0050.7966.6838
                                 10.255.254.103        -       100     0       4200000003 i
 * >      RD: 4200000003:1 mac-ip 10020 0050.7966.6838 10.0.20.30
                                 10.255.254.103        -       100     0       4200000003 i
 * >      RD: 4200000001:1 imet 10010 10.255.254.101
                                 10.255.254.101        -       100     0       4200000001 i
 * >      RD: 4200000003:1 imet 10010 10.255.254.103
                                 10.255.254.103        -       100     0       4200000003 i
 * >      RD: 4200000001:1 imet 10020 10.255.254.101
                                 10.255.254.101        -       100     0       4200000001 i
 * >      RD: 4200000003:1 imet 10020 10.255.254.103
                                 10.255.254.103        -       100     0       4200000003 i
 * >      RD: 4200000001:4096 ip-prefix 10.0.10.0/24
                                 10.255.254.101        -       100     0       4200000001 i
 * >      RD: 4200000003:4096 ip-prefix 10.0.10.0/24
                                 10.255.254.103        -       100     0       4200000003 i
 * >      RD: 4200000003:4096 ip-prefix 10.0.20.0/24
                                 10.255.254.103        -       100     0       4200000003 i


```
</details>


Проверка между хостами:
<details>
<summary><b>SRV10-10:</b></summary>
```

VPCS> ping 10.0.10.1

84 bytes from 10.0.10.1 icmp_seq=1 ttl=64 time=232.979 ms
^C
VPCS> ping 10.0.10.30

84 bytes from 10.0.10.30 icmp_seq=1 ttl=64 time=99.945 ms
84 bytes from 10.0.10.30 icmp_seq=2 ttl=64 time=44.608 ms
^C
VPCS> ping 10.0.20.30

84 bytes from 10.0.20.30 icmp_seq=1 ttl=62 time=203.967 ms
84 bytes from 10.0.20.30 icmp_seq=2 ttl=62 time=74.800 ms
^C


```
</details>
