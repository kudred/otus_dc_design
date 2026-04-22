# Lab02. Построение Underlay сети (OSPF).

## Задание:
1. Собрать сеть с топологией CLOS;
2. Распределить адресное пространство для Underlay сети;
3. Настроить протокол OSPF в Underlay сети;
4. Зафиксировать в документации план работ, адресное пространство, схему сети, настройки;
5. Убедится в наличии IP связанности между устройствами в OSPF домене.

### Соберем схему в PNETLab:
![](lab02-01.png)

## Выполнение
Выделим адресное пространство.

Для IPv4 будем использовать адреса из сети 10.0.0.0/13 (RFC 1918).

Для IPv6 будем использовать случайно сгенерированный Unique Local префикс fdcd:c467:a7d3::0/48 (RFC 4193) и Link-local адреса fe80::/10.

Для сети управления (out-of-band management) будем использовать сеть 172.16.0.0/24.

### Таблица сетей:
|Сеть IPv4|Сеть IPv6|Назначение|
|--|--|--|
|10.0.0.0/13    |fdcd:c467:a7d3:1::/64      |Весь диапазон|
|10.0.0.0/24    |fdcd:c467:a7d3:1:0::/80    |Loopback 1|
|10.1.0.0/24    |fdcd:c467:a7d3:1:1::/80    |Loopback 2|
|10.2.0.0/24    |fe80::/10                  |point to point линки|
|10.3.0.0/24    |-                          |Резерв|
|10.4.0.0/14    |-                          |Сервисы|
|172.16.0.0/24  |-                          |OOB Management|

Назначим адреса на устройства, согласно таблице.

### Таблица адресов:
|Устройство|Интерфейс|Адрес IPv4|Адрес IPv6|Назначение|
|--|--|--|--|--|
|Spine01    |Ma1   |172.16.0.1/24  |-                          |OOB|
|           |Lo0   |10.0.0.1/32    |fdcd:c467:a7d3:1:0::1/128  |Loopback 1|
|           |Lo1   |10.1.0.1/32    |fdcd:c467:a7d3:1:1::1/128  |Loopback 2|
|			|Et1   |10.2.1.0/31    |fe80::1/64                 |p2p to Leaf1|
|			|Et2   |10.2.1.2/31    |fe80::1/64                 |p2p to Leaf2|
|			|Et3   |10.2.1.4/31    |fe80::1/64                 |p2p to Leaf3|
|Spine02    |Ma1   |172.16.0.2/24  |-                          |OOB|
|           |Lo0   |10.0.0.2/32    |fdcd:c467:a7d3:1:0::2/128  |Loopback 1|
|			|Lo1   |10.1.0.2/32    |fdcd:c467:a7d3:1:1::2/128  |Loopback 2|
|			|Et1   |10.2.2.0/31    |fe80::2/64                 |p2p to Leaf1|
|			|Et2   |10.2.2.2/31    |fe80::2/64                 |p2p to Leaf2|
|			|Et3   |10.2.2.4/31    |fe80::2/64                 |p2p to Leaf3|
|Leaf01     |Ma1   |172.16.0.3/24  |-                          |OOB|
|           |Lo0   |10.0.0.3/32    |fdcd:c467:a7d3:1:0::3/128  |Loopback 1|
|			|Lo1   |10.1.0.3/32    |fdcd:c467:a7d3:1:1::3/128  |Loopback 2|
|			|Et1   |10.2.1.1/31    |fe80::3/64                 |p2p to Spine01|
|			|Et2   |10.2.2.1/31    |fe80::3/64                 |p2p to Spine02|
|Leaf02     |Ma1   |172.16.0.4/24  |-                          |OOB|
|           |Lo0   |10.0.0.4/32    |fdcd:c467:a7d3:1:0::4/128  |Loopback 1|
|			|Lo1   |10.1.0.4/32    |fdcd:c467:a7d3:1:1::4/128  |Loopback 2|
|			|Et1   |10.2.1.3/31    |fe80::4/64                 |p2p to Spine01|
|			|Et2   |10.2.2.3/31    |fe80::4/64                 |p2p to Spine02|
|Leaf03     |Ma1   |172.16.0.5/24  |-                          |OOB|
|           |Lo0   |10.0.0.5/32    |fdcd:c467:a7d3:1:0::5/128  |Loopback 1|
|			|Lo1   |10.1.0.5/32    |fdcd:c467:a7d3:1:1::5/128  |Loopback 2|
|			|Et1   |10.2.1.5/31    |fe80::5/64                 |p2p to Spine01|
|			|Et2   |10.2.2.5/31    |fe80::5/64                 |p2p to Spine02|


Проверим IP связанность между Spine01 и Leaf01:
```
Spine01#ping 10.2.1.1
PING 10.2.1.1 (10.2.1.1) 72(100) bytes of data.
80 bytes from 10.2.1.1: icmp_seq=1 ttl=64 time=5.33 ms
80 bytes from 10.2.1.1: icmp_seq=2 ttl=64 time=2.41 ms
80 bytes from 10.2.1.1: icmp_seq=3 ttl=64 time=3.20 ms
80 bytes from 10.2.1.1: icmp_seq=4 ttl=64 time=2.19 ms
80 bytes from 10.2.1.1: icmp_seq=5 ttl=64 time=1.44 ms
--- 10.2.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 20ms
rtt min/avg/max/mdev = 1.442/2.912/5.326/1.329 ms, ipg/ewma 4.937/4.049 ms

Spine01#ping fe80::3 interface ethernet 1
PING fe80::3%et1(fe80::3%et1) 52 data bytes
60 bytes from fe80::3%et1: icmp_seq=1 ttl=64 time=3.41 ms
60 bytes from fe80::3%et1: icmp_seq=2 ttl=64 time=3.65 ms
60 bytes from fe80::3%et1: icmp_seq=3 ttl=64 time=2.71 ms
60 bytes from fe80::3%et1: icmp_seq=4 ttl=64 time=3.11 ms
60 bytes from fe80::3%et1: icmp_seq=5 ttl=64 time=3.59 ms
--- fe80::3%et1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 16ms
rtt min/avg/max/mdev = 2.706/3.295/3.653/0.350 ms, ipg/ewma 3.929/3.355 ms
```

Проверим IP связанность между Spine01 и Leaf02:
```
Spine01#ping 10.2.1.3
PING 10.2.1.3 (10.2.1.3) 72(100) bytes of data.
80 bytes from 10.2.1.3: icmp_seq=1 ttl=64 time=5.10 ms
80 bytes from 10.2.1.3: icmp_seq=2 ttl=64 time=2.88 ms
80 bytes from 10.2.1.3: icmp_seq=3 ttl=64 time=1.89 ms
80 bytes from 10.2.1.3: icmp_seq=4 ttl=64 time=1.59 ms
80 bytes from 10.2.1.3: icmp_seq=5 ttl=64 time=2.18 ms
--- 10.2.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 19ms
rtt min/avg/max/mdev = 1.590/2.728/5.101/1.261 ms, ipg/ewma 4.674/3.858 ms

Spine01#ping fe80::4 interface ethernet 2
PING fe80::4%et2(fe80::4%et2) 52 data bytes
60 bytes from fe80::4%et2: icmp_seq=1 ttl=64 time=2.96 ms
60 bytes from fe80::4%et2: icmp_seq=2 ttl=64 time=3.12 ms
60 bytes from fe80::4%et2: icmp_seq=3 ttl=64 time=1.64 ms
60 bytes from fe80::4%et2: icmp_seq=4 ttl=64 time=2.14 ms
60 bytes from fe80::4%et2: icmp_seq=5 ttl=64 time=2.26 ms
--- fe80::4%et2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 13ms
rtt min/avg/max/mdev = 1.639/2.423/3.118/0.545 ms, ipg/ewma 3.354/2.668 ms
```

Проверим IP связанность между Spine01 и Leaf03:
```
Spine01#ping 10.2.1.5
PING 10.2.1.5 (10.2.1.5) 72(100) bytes of data.
80 bytes from 10.2.1.5: icmp_seq=1 ttl=64 time=3.93 ms
80 bytes from 10.2.1.5: icmp_seq=2 ttl=64 time=3.53 ms
80 bytes from 10.2.1.5: icmp_seq=3 ttl=64 time=3.19 ms
80 bytes from 10.2.1.5: icmp_seq=4 ttl=64 time=1.98 ms
80 bytes from 10.2.1.5: icmp_seq=5 ttl=64 time=1.85 ms
--- 10.2.1.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 19ms
rtt min/avg/max/mdev = 1.849/2.896/3.934/0.835 ms, ipg/ewma 4.854/3.354 ms

Spine01#ping fe80::5 interface ethernet 3
PING fe80::5%et3(fe80::5%et3) 52 data bytes
60 bytes from fe80::5%et3: icmp_seq=1 ttl=64 time=5.43 ms
60 bytes from fe80::5%et3: icmp_seq=2 ttl=64 time=3.38 ms
60 bytes from fe80::5%et3: icmp_seq=3 ttl=64 time=3.25 ms
60 bytes from fe80::5%et3: icmp_seq=4 ttl=64 time=4.23 ms
60 bytes from fe80::5%et3: icmp_seq=5 ttl=64 time=3.20 ms
--- fe80::5%et3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 21ms
rtt min/avg/max/mdev = 3.199/3.896/5.432/0.853 ms, ipg/ewma 5.218/4.640 ms
```

Аналогично проверим IP связанность между Spine02 и Leafs (вывод сокращён):
```
Spine02#ping 10.2.2.1
Spine02#ping 10.2.2.3
Spine02#ping 10.2.2.5
Spine02#ping fe80::3 interface ethernet 1
Spine02#ping fe80::4 interface ethernet 2
Spine02#ping fe80::5 interface ethernet 3
```

Настроим OSPFv2 и OSPFv3.

Все устройства будут в одной зоне 0.0.0.0 (backbone).

В качестве RouterID будем использовать адрес интерфейса Loopback0.

Все интерфейсы в OSPF по-умолчанию сделаем пассивными.

Для автоматического расчета стоимости линков в OSPF будем использовать reference-bandwidth = 1 Терабит/c.

На p2p интерфейсах увеличим MTU, включим BFD, настроим аутентификацию для OSPF и изменим тип линков на point-to-point.




Для примера приведем конфигурации Spine01 и Leaf01.

### Конфигурация Spine01:
```
Spine01(config)#
hostname Spine01

ip routing
ipv6 unicast-routing
no logging console

router ospf 1
	router-id 10.0.0.1
	passive-interface default
	no passive-interface ethernet 1-3
	auto-cost reference-bandwidth 1000000

ipv6 router ospf 1
	router-id 10.0.0.1
	passive-interface default
	no passive-interface ethernet 1-3
	auto-cost reference-bandwidth 1000000

interface Management1
    mac-address 00:00:00:01:01:01
	ip address 172.16.0.1/24

interface loopback 0
    ip address 10.0.0.1/32
    ipv6 enable
    ipv6 address fdcd:c467:a7d3:1:0::1/128
	ip ospf area 0.0.0.0
	ipv6 ospf 1 area 0.0.0.0

interface loopback 1
    ip address 10.1.0.1/32
    ipv6 enable
    ipv6 address fdcd:c467:a7d3:1:1::1/128

interface ethernet 1-3
    ipv6 address fe80::1 link-local
	mtu 9214
	ip ospf network point-to-point
	ip ospf area 0.0.0.0
	ip ospf authentication message-digest
	ip ospf message-digest-key 1 md5 0 P@$$w0rd
	bfd interval 100 min-rx 100 multiplier 3
	ip ospf neighbor bfd
	ipv6 ospf network point-to-point
	ipv6 ospf 1 area 0.0.0.0
	ipv6 ospf authentication ipsec spi 1000 md5 passphrase 0 P@$$w0rd6
	ipv6 ospf bfd

interface ethernet 1
    description p2p to Leaf01
    mac-address 00:00:00:01:00:01
    no switchport
    ip address 10.2.1.0/31
    ipv6 enable

interface ethernet 2
    description p2p to Leaf02
    mac-address 00:00:00:01:00:02
    no switchport
    ip address 10.2.1.2/31
    ipv6 enable

interface ethernet 3
    description p2p to Leaf03
    mac-address 00:00:00:01:00:03
    no switchport
    ip address 10.2.1.4/31
    ipv6 enable

interface ethernet 4
    mac-address 00:00:00:01:00:04
interface ethernet 5
    mac-address 00:00:00:01:00:05
interface ethernet 6
    mac-address 00:00:00:01:00:06
interface ethernet 7
    mac-address 00:00:00:01:00:07
interface ethernet 8
    mac-address 00:00:00:01:00:08

```


### Конфигурация Leaf01:
```
Leaf01(config)#
hostname Leaf01

ip routing
ipv6 unicast-routing
no logging console

router ospf 1
	router-id 10.0.0.3
	max-lsa 12000
	passive-interface default
	no passive-interface ethernet 1-2
	auto-cost reference-bandwidth 1000000

ipv6 router ospf 1
	router-id 10.0.0.3
	passive-interface default
	no passive-interface ethernet 1-2
	auto-cost reference-bandwidth 1000000
	
interface Management1
    mac-address 00:00:00:03:01:01
	ip address 172.16.0.3/24

interface loopback 0
    ip address 10.0.0.3/32
    ipv6 enable
    ipv6 address fdcd:c467:a7d3:1:0::3/128
	ip ospf area 0.0.0.0
	ipv6 ospf 1 area 0.0.0.0

interface loopback 1
    ip address 10.1.0.3/32
    ipv6 enable
    ipv6 address fdcd:c467:a7d3:1:1::3/128

interface ethernet 1-2
    ipv6 address fe80::3 link-local
	mtu 9214
	ip ospf network point-to-point
	ip ospf area 0.0.0.0
	ip ospf authentication message-digest
	ip ospf message-digest-key 1 md5 0 P@$$w0rd
	bfd interval 100 min-rx 100 multiplier 3
	ip ospf neighbor bfd
	ipv6 ospf network point-to-point
	ipv6 ospf 1 area 0.0.0.0
	ipv6 ospf authentication ipsec spi 1000 md5 passphrase 0 P@$$w0rd6
	ipv6 ospf bfd

interface ethernet 1
    description p2p to Spine01
    mac-address 00:00:00:03:00:01
    no switchport
    ip address 10.2.1.1/31
    ipv6 enable

interface ethernet 2
    description p2p to Spine02
    mac-address 00:00:00:03:00:02
    no switchport
    ip address 10.2.2.1/31
    ipv6 enable

interface ethernet 3
    mac-address 00:00:00:03:00:03
interface ethernet 4
    mac-address 00:00:00:03:00:04
interface ethernet 5
    mac-address 00:00:00:03:00:05
interface ethernet 6
    mac-address 00:00:00:03:00:06
interface ethernet 7
    mac-address 00:00:00:03:00:07
interface ethernet 8
    mac-address 00:00:00:03:00:08
```


На Spine01 проверим установление соседских отношений по IPv4:
```
Spine01#show ip ospf neighbor
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.0.3        1        default  0   FULL                   00:00:36    10.2.1.1        Ethernet1
10.0.0.4        1        default  0   FULL                   00:00:31    10.2.1.3        Ethernet2
10.0.0.5        1        default  0   FULL                   00:00:32    10.2.1.5        Ethernet3
```
и по IPv6:
```
Spine01#show ipv6 ospf neighbor
Routing Process "ospf 1":
Neighbor 10.0.0.4 VRF default priority is 0, state is Full
  In area 0.0.0.0 interface Ethernet2
  Adjacency was established 00:00:43 ago
  Current state was established 00:00:43 ago
  DR is None BDR is None
  Options is E R V6
  Dead timer is due in 35 seconds
  Bfd request is sent and the state is Up
  Graceful-restart-helper mode is Inactive
  Graceful-restart attempts: 0
Neighbor 10.0.0.5 VRF default priority is 0, state is Full
  In area 0.0.0.0 interface Ethernet3
  Adjacency was established 00:00:14 ago
  Current state was established 00:00:14 ago
  DR is None BDR is None
  Options is E R V6
  Dead timer is due in 34 seconds
  Bfd request is sent and the state is Up
  Graceful-restart-helper mode is Inactive
  Graceful-restart attempts: 0
Neighbor 10.0.0.3 VRF default priority is 0, state is Full
  In area 0.0.0.0 interface Ethernet1
  Adjacency was established 15:35:24 ago
  Current state was established 15:35:24 ago
  DR is None BDR is None
  Options is E R V6
  Dead timer is due in 35 seconds
  Bfd request is sent and the state is Up
  Graceful-restart-helper mode is Inactive
  Graceful-restart attempts: 0
  ```


На Spine01 проверим таблицу маршрутизации IPv4:
```
Spine01#show ip route

VRF: default

Gateway of last resort is not set

 C        10.0.0.1/32
           directly connected, Loopback0
 O        10.0.0.2/32 [110/2010]
           via 10.2.1.1, Ethernet1
           via 10.2.1.3, Ethernet2
           via 10.2.1.5, Ethernet3
 O        10.0.0.3/32 [110/1010]
           via 10.2.1.1, Ethernet1
 O        10.0.0.4/32 [110/1010]
           via 10.2.1.3, Ethernet2
 O        10.0.0.5/32 [110/1010]
           via 10.2.1.5, Ethernet3
 C        10.1.0.1/32
           directly connected, Loopback1
 C        10.2.1.0/31
           directly connected, Ethernet1
 C        10.2.1.2/31
           directly connected, Ethernet2
 C        10.2.1.4/31
           directly connected, Ethernet3
 O        10.2.2.0/31 [110/2000]
           via 10.2.1.1, Ethernet1
 O        10.2.2.2/31 [110/2000]
           via 10.2.1.3, Ethernet2
 O        10.2.2.4/31 [110/2000]
           via 10.2.1.5, Ethernet3
```
Видим маршруты до Loopback интерфейсов всех устройств.

На Spine01 проверим таблицу маршрутизации IPv6:
```
Spine01#show ipv6 route
VRF: default
Displaying 6 of 9 IPv6 routing table entries

 C        fdcd:c467:a7d3:1::1/128 [0/0]
           via Loopback0, directly connected
 O3       fdcd:c467:a7d3:1::2/128 [110/1020]
           via fe80::3, Ethernet1
 O3       fdcd:c467:a7d3:1::3/128 [110/1010]
           via fe80::3, Ethernet1
 O3       fdcd:c467:a7d3:1::4/128 [110/1010]
           via fe80::4, Ethernet2
 O3       fdcd:c467:a7d3:1::5/128 [110/1010]
           via fe80::5, Ethernet3
 C        fdcd:c467:a7d3:1:1::1/128 [0/0]
           via Loopback1, directly connected

```
Так же видим маршруты до Loopback интерфейсов всех устройств.


Проверим IP связанность между Spine01 и Leaf01 по IPv4 и IPv6:
```
Spine01#ping 10.0.0.3
PING 10.0.0.3 (10.0.0.3) 72(100) bytes of data.
80 bytes from 10.0.0.3: icmp_seq=1 ttl=64 time=2.69 ms
80 bytes from 10.0.0.3: icmp_seq=2 ttl=64 time=2.54 ms
80 bytes from 10.0.0.3: icmp_seq=3 ttl=64 time=2.03 ms
80 bytes from 10.0.0.3: icmp_seq=4 ttl=64 time=3.18 ms
80 bytes from 10.0.0.3: icmp_seq=5 ttl=64 time=4.56 ms
--- 10.0.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 14ms
rtt min/avg/max/mdev = 2.028/3.001/4.563/0.863 ms, ipg/ewma 3.521/2.904 ms

Spine01#ping fdcd:c467:a7d3:1:0::3
PING fdcd:c467:a7d3:1:0::3(fdcd:c467:a7d3:1::3) 52 data bytes
60 bytes from fdcd:c467:a7d3:1::3: icmp_seq=1 ttl=64 time=3.48 ms
60 bytes from fdcd:c467:a7d3:1::3: icmp_seq=2 ttl=64 time=3.94 ms
60 bytes from fdcd:c467:a7d3:1::3: icmp_seq=3 ttl=64 time=3.41 ms
60 bytes from fdcd:c467:a7d3:1::3: icmp_seq=4 ttl=64 time=3.09 ms
60 bytes from fdcd:c467:a7d3:1::3: icmp_seq=5 ttl=64 time=4.10 ms
--- fdcd:c467:a7d3:1:0::3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 15ms
rtt min/avg/max/mdev = 3.089/3.604/4.104/0.368 ms, ipg/ewma 3.813/3.545 ms

```

Проверим IP связанность между Spine01 и Leaf02 по IPv4 и IPv6:
```
Spine01#ping 10.0.0.4
PING 10.0.0.4 (10.0.0.4) 72(100) bytes of data.
80 bytes from 10.0.0.4: icmp_seq=1 ttl=64 time=3.39 ms
80 bytes from 10.0.0.4: icmp_seq=2 ttl=64 time=3.38 ms
80 bytes from 10.0.0.4: icmp_seq=3 ttl=64 time=3.15 ms
80 bytes from 10.0.0.4: icmp_seq=4 ttl=64 time=2.80 ms
80 bytes from 10.0.0.4: icmp_seq=5 ttl=64 time=2.63 ms
--- 10.0.0.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 14ms
rtt min/avg/max/mdev = 2.633/3.072/3.394/0.307 ms, ipg/ewma 3.550/3.209 ms

Spine01#ping fdcd:c467:a7d3:1:0::4
PING fdcd:c467:a7d3:1:0::4(fdcd:c467:a7d3:1::4) 52 data bytes
60 bytes from fdcd:c467:a7d3:1::4: icmp_seq=1 ttl=64 time=3.84 ms
60 bytes from fdcd:c467:a7d3:1::4: icmp_seq=2 ttl=64 time=4.89 ms
60 bytes from fdcd:c467:a7d3:1::4: icmp_seq=3 ttl=64 time=3.35 ms
60 bytes from fdcd:c467:a7d3:1::4: icmp_seq=4 ttl=64 time=2.67 ms
60 bytes from fdcd:c467:a7d3:1::4: icmp_seq=5 ttl=64 time=2.65 ms
--- fdcd:c467:a7d3:1:0::4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 18ms
rtt min/avg/max/mdev = 2.645/3.479/4.893/0.837 ms, ipg/ewma 4.426/3.605 ms

```

Проверим IP связанность между Spine01 и Leaf03 по IPv4 и IPv6:
```
Spine01#ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5) 72(100) bytes of data.
80 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=4.64 ms
80 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=5.75 ms
80 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=2.58 ms
80 bytes from 10.0.0.5: icmp_seq=4 ttl=64 time=2.94 ms
80 bytes from 10.0.0.5: icmp_seq=5 ttl=64 time=4.49 ms
--- 10.0.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 24ms
rtt min/avg/max/mdev = 2.579/4.078/5.745/1.166 ms, ipg/ewma 6.120/4.329 ms

Spine01#ping fdcd:c467:a7d3:1:0::5
PING fdcd:c467:a7d3:1:0::5(fdcd:c467:a7d3:1::5) 52 data bytes
60 bytes from fdcd:c467:a7d3:1::5: icmp_seq=1 ttl=64 time=3.11 ms
60 bytes from fdcd:c467:a7d3:1::5: icmp_seq=2 ttl=64 time=2.64 ms
60 bytes from fdcd:c467:a7d3:1::5: icmp_seq=3 ttl=64 time=4.16 ms
60 bytes from fdcd:c467:a7d3:1::5: icmp_seq=4 ttl=64 time=2.50 ms
60 bytes from fdcd:c467:a7d3:1::5: icmp_seq=5 ttl=64 time=2.07 ms
--- fdcd:c467:a7d3:1:0::5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 16ms
rtt min/avg/max/mdev = 2.070/2.893/4.160/0.714 ms, ipg/ewma 4.073/2.971 ms
```

Проверим IP связанность между Spine01 и Spine02 по IPv4 и IPv6:
```
Spine01#ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 72(100) bytes of data.
80 bytes from 10.0.0.2: icmp_seq=1 ttl=63 time=11.0 ms
80 bytes from 10.0.0.2: icmp_seq=2 ttl=63 time=5.81 ms
80 bytes from 10.0.0.2: icmp_seq=3 ttl=63 time=2.62 ms
80 bytes from 10.0.0.2: icmp_seq=4 ttl=63 time=2.68 ms
80 bytes from 10.0.0.2: icmp_seq=5 ttl=63 time=3.46 ms
--- 10.0.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 39ms
rtt min/avg/max/mdev = 2.620/5.105/10.952/3.144 ms, ipg/ewma 9.781/7.883 ms

Spine01#ping fdcd:c467:a7d3:1:0::2
PING fdcd:c467:a7d3:1:0::2(fdcd:c467:a7d3:1::2) 52 data bytes
60 bytes from fdcd:c467:a7d3:1::2: icmp_seq=1 ttl=63 time=7.83 ms
60 bytes from fdcd:c467:a7d3:1::2: icmp_seq=2 ttl=63 time=6.42 ms
60 bytes from fdcd:c467:a7d3:1::2: icmp_seq=3 ttl=63 time=4.39 ms
60 bytes from fdcd:c467:a7d3:1::2: icmp_seq=4 ttl=63 time=2.94 ms
60 bytes from fdcd:c467:a7d3:1::2: icmp_seq=5 ttl=63 time=2.64 ms
--- fdcd:c467:a7d3:1:0::2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 31ms
rtt min/avg/max/mdev = 2.644/4.844/7.830/2.003 ms, ipg/ewma 7.859/6.199 ms
```


Проверим работоспособность BFD:
```
Spine01#show bfd peers
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type           LastUp
--------- ----------- ----------- -------------------- ------- ----------------
10.2.1.1  1865089838  1286053506        Ethernet1(22)  normal   04/21/26 19:45
10.2.1.3   458622646   430066288        Ethernet2(23)  normal   04/21/26 19:45
10.2.1.5  3727709014  2057504065        Ethernet3(20)  normal   04/21/26 19:45

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

DstAddr     MyDisc   YourDisc Interface/Transport   Type         LastUp LastDown
------- ---------- ---------- ------------------- ------ -------------- --------
fe80::3 2395735173  862626535       Ethernet1(22) normal 04/21/26 20:09       NA
fe80::4   97786484 3960497956       Ethernet2(23) normal 04/22/26 11:44       NA
fe80::5 2680906623 2996531171       Ethernet3(20) normal 04/22/26 11:45       NA

        LastDiag    State
------------------- -----
   No Diagnostic       Up
   No Diagnostic       Up
   No Diagnostic       Up
```


Для примера выведем из работы Spine01, например для обслуживания.
Проверим таблицу маршрутизации на Spine01:
```
Spine01#show ip route
VRF: default
Gateway of last resort is not set

 C        10.0.0.1/32
           directly connected, Loopback0
 O        10.0.0.2/32 [110/2010]
           via 10.2.1.1, Ethernet1
           via 10.2.1.3, Ethernet2
           via 10.2.1.5, Ethernet3
 O        10.0.0.3/32 [110/1010]
           via 10.2.1.1, Ethernet1
 O        10.0.0.4/32 [110/1010]
           via 10.2.1.3, Ethernet2
 O        10.0.0.5/32 [110/1010]
           via 10.2.1.5, Ethernet3
 C        10.1.0.1/32
           directly connected, Loopback1
 C        10.2.1.0/31
           directly connected, Ethernet1
 C        10.2.1.2/31
           directly connected, Ethernet2
 C        10.2.1.4/31
           directly connected, Ethernet3
 O        10.2.2.0/31 [110/2000]
           via 10.2.1.1, Ethernet1
 O        10.2.2.2/31 [110/2000]
           via 10.2.1.3, Ethernet2
 O        10.2.2.4/31 [110/2000]
           via 10.2.1.5, Ethernet3
```

Проверим доступные пути с Leaf03 до Leaf01:
```
Leaf03#show ip route 10.0.0.3
VRF: default
 O        10.0.0.3/32 [110/2010]
           via 10.2.1.4, Ethernet1
           via 10.2.2.4, Ethernet2
```
Видим ECMP маршрут через оба спайна.

На Spine01 увеличиваем метрики до максимума:
```
Spine01(config)#
router ospf 1
    max-metric router-lsa
```

Проверяем таблицу маршрутизации на Spine01:
```
Spine01#show ip route
VRF: default
Gateway of last resort is not set

 C        10.0.0.1/32
           directly connected, Loopback0
 O        10.0.0.2/32 [110/66545]
           via 10.2.1.1, Ethernet1
           via 10.2.1.3, Ethernet2
           via 10.2.1.5, Ethernet3
 O        10.0.0.3/32 [110/65545]
           via 10.2.1.1, Ethernet1
 O        10.0.0.4/32 [110/65545]
           via 10.2.1.3, Ethernet2
 O        10.0.0.5/32 [110/65545]
           via 10.2.1.5, Ethernet3
 C        10.1.0.1/32
           directly connected, Loopback1
 C        10.2.1.0/31
           directly connected, Ethernet1
 C        10.2.1.2/31
           directly connected, Ethernet2
 C        10.2.1.4/31
           directly connected, Ethernet3
 O        10.2.2.0/31 [110/66535]
           via 10.2.1.1, Ethernet1
 O        10.2.2.2/31 [110/66535]
           via 10.2.1.3, Ethernet2
 O        10.2.2.4/31 [110/66535]
           via 10.2.1.5, Ethernet3
```

Видим увеличенные значения метрик.

Проверим доступные пути с Leaf03 до Leaf01:
```
Leaf03#show ip route 10.0.0.3
VRF: default
 O        10.0.0.3/32 [110/2010]
           via 10.2.2.4, Ethernet2
```
Видим только один маршрут через Spine02.

Выполним трассировку:
```
Leaf03#traceroute 10.0.0.3
traceroute to 10.0.0.3 (10.0.0.3), 30 hops max, 60 byte packets
 1  10.2.2.4 (10.2.2.4)  3.465 ms  4.381 ms  4.757 ms
 2  10.0.0.3 (10.0.0.3)  11.585 ms  12.637 ms  12.711 ms
```
Видим, что трафик идет через Spine02.


### На этом настройка OSPF закончена.

