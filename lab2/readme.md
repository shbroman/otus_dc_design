


# Задание:

1. Настроите OSPF в Underlay сети, для IP связанности между всеми сетевыми устройствами.
2. Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
3. Убедитесь в наличии IP связанности между устройствами в OSFP домене




# Решение:

1. [Выделение адресов](#выделение-блоков-адресов)
2. [Назначение IP-адресов](#схема)


    
## Разворачивание топологии:
В качестве оборудования было выбрано оборудование Arista ver. 4.29.2F, в качестве среды разработки - Pnetlab



### Схема:
![Схема](./images/scheme.png "Визуализация")

### Описание:
- 10.255.255.X/24 - SPINE loopback IP address, где X - номер SPINE
- 10.255.254.X/24 - LEAF loopback IP address, где X - номер LEAF
- 10.255.253.0/24 - линковая подсеть для связи SPINE-LEAF. Используются /31 подсети. Четный номер - SPINE, нечетный LEAF
- 10.0.0.0/24 - сервисы

### Конфигурация:

- SPINE1:
```
interface Ethernet1
   description # DC01-LSW001 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.100/31
   arp aging timeout 300
   ip ospf network point-to-point
interface Ethernet2
   description # DC01-LSW002 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.102/31
   arp aging timeout 300
   ip ospf network point-to-point
interface Ethernet3
   description # DC01-LSW003 #
   speed forced 40gfull
   no switchport
   ip address 10.255.253.104/31
   arp aging timeout 300
   ip ospf network point-to-point
interface Loopback0
   ip address 10.255.255.1/32

ip routing
router ospf 1
   router-id 10.255.255.1
   auto-cost reference-bandwidth 1000000
   network 10.252.0.0/14 area 0.0.0.0
   max-lsa 12000
```
