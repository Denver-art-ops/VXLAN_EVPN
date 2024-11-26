# Настройка OSPF в Underlay

### Цель:
Настройка OSPF в Underlay на фабрике.<br>
Настроить IP на всех активных портах (PtP линки и loopback1) для дальнейшей работы над проектом.<br>
Адресное пространство должно быть задокументировано.<br>

### Принципы назначения IP адресов, адресное пространство
Описаны в документе: [README.md](README.md)

### Итоговая схема
![Topology_OSPF.png](Topology_OSPF.png)

## Конфигурации устройств:

|                             |                               |                            |                        |
|-----------------------------|-------------------------------|----------------------------|------------------------|
| [SPINE01.cfg](SPINE01.cfg)  |   [BLEAF01.cfg](BLEAF01.cfg)  | [SLEAF01.cfg](SLEAF01.cfg) | [BGW01.cfg](BGW01.cfg) |
| [SPINE02.cfg](SPINE02.cfg)  |   [BLEAF02.cfg](BLEAF02.cfg)  | [SLEAF02.cfg](SLEAF02.cfg) | [BGW02.cfg](BGW02.cfg) |
| [LEAF01.cfg](LEAF01.cfg)    |-------------------------------|----------------------------|------------------------|
| [LEAF02.cfg](LEAF02.cfg)    |-------------------------------|----------------------------|------------------------|
| [LEAF03.cfg](LEAF03.cfg)    |-------------------------------|----------------------------|------------------------|
| [LEAF04.cfg](LEAF04.cfg)    |-------------------------------|----------------------------|------------------------|


## Подтверждение работоспособности Топологии:

### show ip ospf neighbor

#### dc01-pod01-spine01#
dc01-pod01-spine01#show ip ospf neighbor

|Neighbor  |ID |Instance|VRF|Pri |State   |Dead      |Time      |Address|Interface|
|----------|---|--------|---|----|--------|----------|----------|-------|---------|
|10.11.1.3 |1  |default |0  |FULL|00:00:38|10.11.3.1 |Ethernet1 |       |         |
|10.11.1.4 |1  |default |0  |FULL|00:00:33|10.11.3.3 |Ethernet2 |       |         |
|10.11.1.5 |1  |default |0  |FULL|00:00:34|10.11.3.5 |Ethernet3 |       |         |
|10.11.1.6 |1  |default |0  |FULL|00:00:30|10.11.3.7 |Ethernet4 |       |         |
|10.11.1.7 |1  |default |0  |FULL|00:00:34|10.11.3.9 |Ethernet5 |       |         |
|10.11.1.8 |1  |default |0  |FULL|00:00:34|10.11.3.11|Ethernet6 |       |         |
|10.11.1.9 |1  |default |0  |FULL|00:00:31|10.11.3.13|Ethernet7 |       |         |
|10.11.1.10|1  |default |0  |FULL|00:00:32|10.11.3.15|Ethernet8 |       |         |
|10.11.1.12|1  |default |0  |FULL|00:00:35|10.11.3.19|Ethernet10|       |         |
|10.11.1.11|1  |default |0  |FULL|00:00:31|10.11.3.17|Ethernet9 |       |         |


#### dc01-pod01-spine02#

dc01-pod01-spine02#show ip ospf neighbor

|Neighbor  |ID |Instance|VRF|Pri |State   |Dead      |Time      |Address|Interface|
|----------|---|--------|---|----|--------|----------|----------|-------|---------|
|10.11.1.3 |1  |default |0  |FULL|00:00:35|10.11.3.41|Ethernet1 |       |         |
|10.11.1.4 |1  |default |0  |FULL|00:00:31|10.11.3.43|Ethernet2 |       |         |
|10.11.1.7 |1  |default |0  |FULL|00:00:33|10.11.3.49|Ethernet5 |       |         |
|10.11.1.8 |1  |default |0  |FULL|00:00:32|10.11.3.51|Ethernet6 |       |         |
|10.11.1.9 |1  |default |0  |FULL|00:00:33|10.11.3.53|Ethernet7 |       |         |
|10.11.1.10|1  |default |0  |FULL|00:00:32|10.11.3.55|Ethernet8 |       |         |
|10.11.1.11|1  |default |0  |FULL|00:00:29|10.11.3.57|Ethernet9 |       |         |
|10.11.1.12|1  |default |0  |FULL|00:00:30|10.11.3.59|Ethernet10|       |         |
|10.11.1.5 |1  |default |0  |FULL|00:00:33|10.11.3.45|Ethernet3 |       |         |
|10.11.1.6 |1  |default |0  |FULL|00:00:29|10.11.3.47|Ethernet4 |       |         |




### show ip ospf 


### show bfd peers
