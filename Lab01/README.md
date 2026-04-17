### Lab01.  Основы проектирования сети.


|Устройство|Интерфейс|Адрес|Назначение|
|--|--|--|--|
|Spine101|Lo0|10.0.0.101/32|Loopback 1|
|				 |Lo1|10.1.0.101/32|Loopback 2|
|				 |Et1|10.2.101.0/31|p2p to Leaf1|
|				 |Et2|10.2.101.2/31|p2p to Leaf2|
|				 |Et3|10.2.101.4/31|p2p to Leaf3|
|Spine102|Lo0|10.0.0.102/32|Loopback 1|
|				 |Lo1|10.1.0.102/32|Loopback 2|
|				 |Et1|10.2.102.0/31|p2p to Leaf1|
|				 |Et2|10.2.102.2/31|p2p to Leaf2|
|				 |Et3|10.2.102.4/31|p2p to Leaf3|
|Leaf1   |Lo0|10.0.0.1/32|Loopback 1|
|				 |Lo1|10.1.0.1/32|Loopback 2|
|				 |Et1|10.2.101.1/31|p2p to Spine101|
|				 |Et2|10.2.102.1/31|p2p to Spine102|
|Leaf2   |Lo0|10.0.0.2/32|Loopback 1|
|				 |Lo1|10.1.0.2/32|Loopback 2|
|				 |Et1|10.2.101.3/31|p2p to Spine101|
|				 |Et2|10.2.102.3/31|p2p to Spine102|
|Leaf3   |Lo0|10.0.0.3/32|Loopback 1|
|				 |Lo1|10.1.0.3/32|Loopback 2|
|				 |Et1|10.2.101.5/31|p2p to Spine101|
|				 |Et2|10.2.102.5/31|p2p to Spine102|

