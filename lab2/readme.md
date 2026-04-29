


# Задание:

1. Настроите OSPF в Underlay сети, для IP связанности между всеми сетевыми устройствами.
2. Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
3. Убедитесь в наличии IP связанности между устройствами в OSFP домене




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
   speed forced 40gfull
   no switchport
   ip address 10.255.253.100/31
   arp aging timeout 300
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 ywNY2V3LPbnmR85VqJaKfg==
interface Ethernet2
   description # DC01-LSW002 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.102/31
   arp aging timeout 300
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 ywNY2V3LPbnmR85VqJaKfg==
interface Ethernet3
   description # DC01-LSW003 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.104/31
   arp aging timeout 300
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 ywNY2V3LPbnmR85VqJaKfg==
interface Loopback0
   ip address 10.255.255.1/32

ip routing
router ospf 1
   router-id 10.255.255.1
   auto-cost reference-bandwidth 1000000
   network 10.252.0.0/14 area 0.0.0.0
   max-lsa 12000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
```
</details>
<details>
<summary><b>SPINE 2:</b></summary>

```

interface Ethernet1
   description # DC01-LSW001 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.200/31
   arp aging timeout 300
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 ywNY2V3LPbnmR85VqJaKfg==
interface Ethernet2
   description # DC01-LSW002 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.202/31
   arp aging timeout 300
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 ywNY2V3LPbnmR85VqJaKfg==
interface Ethernet3
   description # DC01-LSW003 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.204/31
   arp aging timeout 300
   bfd interval 100 min-rx 100 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 ywNY2V3LPbnmR85VqJaKfg==
interface Loopback0
   ip address 10.255.255.2/32

ip routing
router ospf 1
   router-id 10.255.255.2
   auto-cost reference-bandwidth 1000000
   network 10.252.0.0/14 area 0.0.0.0
   max-lsa 12000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
```
</details>



<details>
<summary><b>LEAF 1:</b></summary>

```

interface Ethernet1
   description # DC01-SSW001 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.101/31
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
   ip address 10.255.253.201/31
   arp aging timeout 300
   bfd interval 1000 min-rx 1000 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 ywNY2V3LPbnmR85VqJaKfg==
interface Loopback0
   ip address 10.255.254.1/32

ip routing
router ospf 1
   router-id 10.255.254.1
   auto-cost reference-bandwidth 1000000
   network 10.252.0.0/14 area 0.0.0.0
   max-lsa 12000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
```
</details>




<details>
<summary><b>LEAF 2:</b></summary>

```
interface Ethernet1
   description # DC01-SSW001 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.103/31
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
   ip address 10.255.253.203/31
   arp aging timeout 300
   bfd interval 1000 min-rx 1000 multiplier 3
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf authentication-key 7 ywNY2V3LPbnmR85VqJaKfg==
interface Loopback0
   ip address 10.255.254.2/32


ip routing
router ospf 1
   router-id 10.255.254.2
   auto-cost reference-bandwidth 1000000
   network 10.252.0.0/14 area 0.0.0.0
   max-lsa 12000
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
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
 - просмотр OSPF-соседей;
 - просмотр таблицы маршрутизации на каждом коммутаторе;
 - проверка связности посредством icmp echo request до каждого LEAF and SPINE.

Под катом находится пример проверки, выполненной на LEAF 1. На остальных устройства проверки выполняются аналогично

<details>
<summary><b>LEAF 1:</b></summary>

```

show ip ospf database

            OSPF Router with ID(10.255.254.1) (Instance ID 1) (VRF default)


                 Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age         Seq#         Checksum Link count
10.255.255.2    10.255.255.2    529         0x8000001a   0x2fbc   7
10.255.254.2    10.255.254.2    537         0x8000000c   0xe5dc   5
10.255.254.3    10.255.254.3    531         0x8000000c   0x3186   5
10.255.254.1    10.255.254.1    529         0x8000000c   0xaf1e   5
10.255.255.1    10.255.255.1    918         0x80000008   0xb3a7   7


DC01-LSW001# show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.255.255.2    1        default  0   FULL                   00:00:36    10.255.253.200  Ethernet2
10.255.255.1    1        default  0   FULL                   00:00:37    10.255.253.100  Ethernet1

DC01-LSW001#show bfd peers
VRF name: default
-----------------
DstAddr            MyDisc   YourDisc Interface/Transport   Type         LastUp
-------------- ---------- ---------- ------------------- ------ ---------------
10.255.253.100 2209485441 4272456046       Ethernet1(14) normal 04/29/26 10:48
10.255.253.200 2905977715 4206177367       Ethernet2(15) normal 04/29/26 11:01

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up



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
 O        10.255.253.102/31 [110/50] via 10.255.253.100, Ethernet1
 O        10.255.253.104/31 [110/50] via 10.255.253.100, Ethernet1
 C        10.255.253.200/31 is directly connected, Ethernet2
 O        10.255.253.202/31 [110/50] via 10.255.253.200, Ethernet2
 O        10.255.253.204/31 [110/50] via 10.255.253.200, Ethernet2
 C        10.255.254.1/32 is directly connected, Loopback0
 O        10.255.254.2/32 [110/60] via 10.255.253.100, Ethernet1
                                   via 10.255.253.200, Ethernet2
 O        10.255.254.3/32 [110/60] via 10.255.253.100, Ethernet1
                                   via 10.255.253.200, Ethernet2
 O        10.255.255.1/32 [110/35] via 10.255.253.100, Ethernet1
 O        10.255.255.2/32 [110/35] via 10.255.253.200, Ethernet2

DC01-LSW001#ping 10.255.254.2
PING 10.255.254.2 (10.255.254.2) 72(100) bytes of data.
80 bytes from 10.255.254.2: icmp_seq=1 ttl=63 time=128 ms
80 bytes from 10.255.254.2: icmp_seq=2 ttl=63 time=123 ms
80 bytes from 10.255.254.2: icmp_seq=3 ttl=63 time=116 ms
80 bytes from 10.255.254.2: icmp_seq=4 ttl=63 time=123 ms
80 bytes from 10.255.254.2: icmp_seq=5 ttl=63 time=148 ms

--- 10.255.254.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 49ms
rtt min/avg/max/mdev = 116.429/127.942/148.644/11.032 ms, pipe 5, ipg/ewma 12.4s
DC01-LSW001#ping 10.255.254.3
PING 10.255.254.3 (10.255.254.3) 72(100) bytes of data.
80 bytes from 10.255.254.3: icmp_seq=1 ttl=63 time=80.2 ms
80 bytes from 10.255.254.3: icmp_seq=2 ttl=63 time=85.3 ms
80 bytes from 10.255.254.3: icmp_seq=3 ttl=63 time=86.0 ms
80 bytes from 10.255.254.3: icmp_seq=4 ttl=63 time=84.2 ms
80 bytes from 10.255.254.3: icmp_seq=5 ttl=63 time=82.5 ms

--- 10.255.254.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 51ms
rtt min/avg/max/mdev = 80.264/83.703/86.072/2.117 ms, pipe 5, ipg/ewma 12.962/8s
DC01-LSW001#ping 10.255.255.1
PING 10.255.255.1 (10.255.255.1) 72(100) bytes of data.
80 bytes from 10.255.255.1: icmp_seq=1 ttl=64 time=46.3 ms
80 bytes from 10.255.255.1: icmp_seq=2 ttl=64 time=47.5 ms
80 bytes from 10.255.255.1: icmp_seq=3 ttl=64 time=49.9 ms
80 bytes from 10.255.255.1: icmp_seq=4 ttl=64 time=49.3 ms
80 bytes from 10.255.255.1: icmp_seq=5 ttl=64 time=47.8 ms

--- 10.255.255.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 42ms
rtt min/avg/max/mdev = 46.310/48.189/49.949/1.326 ms, pipe 5, ipg/ewma 10.693/4s
DC01-LSW001#ping 10.255.255.2
PING 10.255.255.2 (10.255.255.2) 72(100) bytes of data.
80 bytes from 10.255.255.2: icmp_seq=1 ttl=64 time=38.1 ms
80 bytes from 10.255.255.2: icmp_seq=2 ttl=64 time=32.4 ms
80 bytes from 10.255.255.2: icmp_seq=3 ttl=64 time=24.3 ms
80 bytes from 10.255.255.2: icmp_seq=4 ttl=64 time=9.10 ms
80 bytes from 10.255.255.2: icmp_seq=5 ttl=64 time=12.1 ms

--- 10.255.255.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 111ms
rtt min/avg/max/mdev = 9.102/23.236/38.156/11.252 ms, pipe 3, ipg/ewma 27.847/2s
DC01-LSW001#
```
</details>