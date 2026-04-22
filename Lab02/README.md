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

На p2p интерфейсах увеличим MTU, включим BFD и настроим аутентификацию для OSPF.




Для примера приведем конфигурации Spine01 и Leaf01.

### конфигурация Spine01:
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


### конфигурация Leaf01:
```
Leaf01(config)

hostname Leaf01

interface Management1
    mac-address 00:00:00:03:01:01
	ip address 172.16.0.3/24

interface loopback 0
    ip address 10.0.0.3/32
    ipv6 enable
    ipv6 address fdcd:c467:a7d3:1:0::3/128

interface loopback 1
    ip address 10.1.0.3/32
    ipv6 enable
    ipv6 address fdcd:c467:a7d3:1:1::3/128

interface ethernet 1 - 2
    ipv6 address fe80::3 link-local

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


