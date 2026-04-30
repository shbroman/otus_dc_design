


# Задание:

1. настроить IS-IS для Underlay сети.




# Решение:

1. [Создание, документация сети](#создание-сети)
2. [Проверка IP связности](#проверка-доступности)


    
### Создание сети:
В качестве оборудования было выбрано оборудование Arista ver. 4.29.2F, в качестве среды разработки - Pnetlab. Подключение было выполнено согласно прилагаемой схеме:


![Схема](/lab1/images/scheme.png "Визуализация")

#### Описание:
- 10.255.255.X/24 - SPINE loopback IP address, где X - номер SPINE
- 10.255.254.X/24 - LEAF loopback IP address, где X - номер LEAF
- 10.255.253.0/24 - линковая подсеть для связи SPINE-LEAF. Используются /31 подсети. Четный номер - SPINE, нечетный LEAF
- 10.0.0.0/24 - сервисы

#### Конфигурация:
<details>
<summary><b>SPINE 1:</b></summary>

```
interface Ethernet1
   description # DC01-LSW001 #
   no switchport
   ip address 10.255.253.100/31
   arp aging timeout 300
   bfd interval 2000 min-rx 2000 multiplier 5
   no ip ospf neighbor bfd
   isis enable dc01
   isis bfd
   isis network point-to-point
!
interface Ethernet2
   description # DC01-LSW002 #
   no switchport
   ip address 10.255.253.102/31
   arp aging timeout 300
   bfd interval 2000 min-rx 2000 multiplier 5
   no ip ospf neighbor bfd
   isis enable dc01
   isis bfd
   isis network point-to-point
!
interface Ethernet3
   description # DC01-LSW003 #
   no switchport
   ip address 10.255.253.104/31
   arp aging timeout 300
   bfd interval 2000 min-rx 2000 multiplier 5
   no ip ospf neighbor bfd
   isis enable dc01
   isis bfd
   isis network point-to-point
!
interface Loopback0
   ip address 10.255.255.1/32
   isis enable dc01
!
router isis dc01
   net 49.0001.0102.5525.5001.00
   is-hostname DC01-SSW001
   is-type level-1
   log-adjacency-changes
   authentication mode md5
   authentication key 7 M3PyxLnkzntzlkDlmMXhtQ==
   !
   address-family ipv4 unicast


```
</details>
<details>
<summary><b>SPINE 2:</b></summary>

```
interface Ethernet1
   description # DC01-LSW001 #
   no switchport
   ip address 10.255.253.200/31
   arp aging timeout 300
   bfd interval 2000 min-rx 2000 multiplier 5
   no ip ospf neighbor bfd
   isis enable dc01
   isis bfd
   isis network point-to-point
!
interface Ethernet2
   description # DC01-LSW002 #
   no switchport
   ip address 10.255.253.202/31
   arp aging timeout 300
   bfd interval 2000 min-rx 2000 multiplier 5
   no ip ospf neighbor bfd
   isis enable dc01
   isis bfd
   isis network point-to-point
!
interface Ethernet3
   description # DC01-LSW003 #
   no switchport
   ip address 10.255.253.204/31
   arp aging timeout 300
   bfd interval 2000 min-rx 2000 multiplier 5
   no ip ospf neighbor bfd
   isis enable dc01
   isis bfd
   isis network point-to-point
!
interface Loopback0
   ip address 10.255.255.2/32
   isis enable dc01
!
router isis dc01
   net 49.0001.0102.5525.5002.00
   is-hostname DC01-SSW002
   is-type level-1
   log-adjacency-changes
   authentication mode md5
   authentication key 7 M3PyxLnkzntzlkDlmMXhtQ==
   !
   address-family ipv4 unicast
!
```
</details>



<details>
<summary><b>LEAF 1:</b></summary>

```
interface Ethernet1
   description # DC01-SSW001 #
   no switchport
   ip address 10.255.253.101/31
   arp aging timeout 300
   bfd interval 2000 min-rx 2000 multiplier 5
   no ip ospf neighbor bfd
   isis enable dc01
   isis bfd
   isis network point-to-point
!
interface Ethernet2
   description # DC01-SSW002 #
   no switchport
   ip address 10.255.253.201/31
   arp aging timeout 300
   bfd interval 2000 min-rx 2000 multiplier 5
   no ip ospf neighbor bfd
   isis enable dc01
   isis bfd
   isis network point-to-point
!
interface Loopback0
   ip address 10.255.254.1/32
   isis enable dc01
!
router isis dc01
   net 49.0001.0102.5525.4001.00
   is-hostname DC01-LSW001
   is-type level-1
   log-adjacency-changes
   authentication mode md5
   authentication key 7 M3PyxLnkzntzlkDlmMXhtQ==
   !
   address-family ipv4 unicast
```
</details>




<details>
<summary><b>LEAF 2:</b></summary>

```
!
interface Ethernet1
   description # DC01-SSW001 #
   no switchport
   ip address 10.255.253.103/31
   arp aging timeout 300
   bfd interval 2000 min-rx 2000 multiplier 5
   no ip ospf neighbor bfd
   isis enable dc01
   isis bfd
   isis network point-to-point
!
interface Ethernet2
   description # DC01-SSW002 #
   no switchport
   ip address 10.255.253.203/31
   arp aging timeout 300
   bfd interval 2000 min-rx 2000 multiplier 5
   no ip ospf neighbor bfd
   isis enable dc01
   isis bfd
   isis network point-to-point
!
interface Ethernet3
   description # DC01-LSW003 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.204/31
   arp aging timeout 300
   bfd interval 1000 min-rx 1000 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 CuLN1L9ii+Bw/X3oykPG5w==
   isis enable dc01
   isis network point-to-point
!
interface Loopback0
   ip address 10.255.254.2/32
   isis enable dc01
!
router isis dc01
   net 49.0001.0102.5525.4002.00
   is-hostname DC01-LSW002
   is-type level-1
   log-adjacency-changes
   authentication mode md5
   authentication key 7 M3PyxLnkzntzlkDlmMXhtQ==
   !
   address-family ipv4 unicast
!

```
</details>




<details>
<summary><b>LEAF 3:</b></summary>

```
interface Ethernet1
   description # DC01-SSW001 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.105/31
   arp aging timeout 300
   bfd interval 1000 min-rx 1000 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 ywNY2V3LPbnmR85VqJaKfg==
interface Ethernet2
   description # DC01-SSW002 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.205/31
   arp aging timeout 300
   bfd interval 1000 min-rx 1000 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 ywNY2V3LPbnmR85VqJaKfg==
interface Loopback0
   ip address 10.255.254.3/32


ip routing
router ospf 1
   router-id 10.255.254.3
   auto-cost reference-bandwidth 1000000
   network 10.252.0.0/14 area 0.0.0.0
   max-lsa 12000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
```
</details>


📥 [Скачать](./configs/)  файлы лабы в формате .zip


### Проверка доступности: 
Проверка выполняется на каждом из коммутаторов по следующим критериям:
 - просмотр ISIS-соседей;
 - просмотр таблицы маршрутизации на каждом коммутаторе;
 - проверка связности посредством icmp echo request до каждого LEAF and SPINE.

Под катом находится пример проверки, выполненной на LEAF 1. На остальных устройства проверки выполняются аналогично.

<details>
<summary><b>LEAF 1:</b></summary>

```


DC01-LSW001#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
dc01      default  DC01-SSW001      L1   Ethernet1          P2P               UP    30          0D
dc01      default  DC01-SSW002      L1   Ethernet2          P2P               UP    23          0D
DC01-LSW001#show bfd peers
VRF name: default
-----------------
DstAddr            MyDisc   YourDisc Interface/Transport   Type         LastUp
-------------- ---------- ---------- ------------------- ------ ---------------
10.255.253.100  939777136 2074720834       Ethernet1(13) normal 04/30/26 11:35
10.255.253.200 1143991017 4046718200       Ethernet2(14) normal 04/30/26 11:35

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

DC01-LSW001#show isis database

IS-IS Instance: dc01 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    DC01-LSW001.00-00           325  29615   866    146 L1 <>
    DC01-LSW002.00-00           398  33273   940    159 L1 <>
    DC01-LSW003.00-00           268  11768   462    146 L1 <>
    DC01-SSW001.00-00           924   7217   885    170 L1 <>
    DC01-SSW002.00-00           315  26174   689    170 L1 <>
DC01-LSW001#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 C        10.255.253.100/31 is directly connected, Ethernet1
 I L1     10.255.253.102/31 [115/20] via 10.255.253.100, Ethernet1
 I L1     10.255.253.104/31 [115/20] via 10.255.253.100, Ethernet1
 C        10.255.253.200/31 is directly connected, Ethernet2
 I L1     10.255.253.202/31 [115/20] via 10.255.253.200, Ethernet2
 I L1     10.255.253.204/31 [115/20] via 10.255.253.200, Ethernet2
 C        10.255.254.1/32 is directly connected, Loopback0
 I L1     10.255.254.2/32 [115/30] via 10.255.253.100, Ethernet1
                                   via 10.255.253.200, Ethernet2
 I L1     10.255.254.3/32 [115/30] via 10.255.253.100, Ethernet1
                                   via 10.255.253.200, Ethernet2
 I L1     10.255.255.1/32 [115/20] via 10.255.253.100, Ethernet1
 I L1     10.255.255.2/32 [115/20] via 10.255.253.200, Ethernet2

DC01-LSW001#ping 10.255.255.1
PING 10.255.255.1 (10.255.255.1) 72(100) bytes of data.
80 bytes from 10.255.255.1: icmp_seq=1 ttl=64 time=136 ms
80 bytes from 10.255.255.1: icmp_seq=2 ttl=64 time=128 ms
80 bytes from 10.255.255.1: icmp_seq=3 ttl=64 time=126 ms
80 bytes from 10.255.255.1: icmp_seq=4 ttl=64 time=119 ms
80 bytes from 10.255.255.1: icmp_seq=5 ttl=64 time=92.0 ms

--- 10.255.255.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 83ms
rtt min/avg/max/mdev = 92.023/120.515/136.091/15.189 ms, pipe 5, ipg/ewma 20.853/127.216 ms
^[[ADC01-LSWping 10.255.254.1
PING 10.255.254.1 (10.255.254.1) 72(100) bytes of data.
80 bytes from 10.255.254.1: icmp_seq=1 ttl=64 time=1.92 ms
80 bytes from 10.255.254.1: icmp_seq=2 ttl=64 time=0.303 ms
80 bytes from 10.255.254.1: icmp_seq=3 ttl=64 time=0.304 ms
80 bytes from 10.255.254.1: icmp_seq=4 ttl=64 time=0.298 ms
80 bytes from 10.255.254.1: icmp_seq=5 ttl=64 time=0.296 ms

--- 10.255.254.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 16ms
rtt min/avg/max/mdev = 0.296/0.625/1.927/0.651 ms, ipg/ewma 4.222/1.253 ms
DC01-LSW001#


```
</details>