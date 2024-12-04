# Настройка eBGPv4 в Underlay

### Цель:
Настройка eBGP в Underlay на фабрике.<br>
Настроить IP на всех активных портах (PtP линки и loopback1) для дальнейшей работы над проектом.<br>
Адресное пространство должно быть задокументировано.<br>

### Принципы назначения IP адресов, адресное пространство
Описаны в документе: [README.md](README.md)

### Итоговая схема
![ISIS_Topology_L1_L2.png](ISIS_Topology_L1_L2.png)

## Конфигурации устройств:

|                             |                               |                            |                        |
|-----------------------------|-------------------------------|----------------------------|------------------------|
| [SPINE01.cfg](SPINE01.txt.txt)  |   [BLEAF01.cfg](BLEAF01.txt.txt)  | [SLEAF01.cfg](SLEAF01.txt.txt) | [BGW01.cfg](BGW01.txt.txt) |
| [SPINE02.cfg](SPINE02.txt.txt)  |   [BLEAF02.cfg](BLEAF02.txt.txt)  | [SLEAF02.cfg](SLEAF02.txt.txt) | [BGW02.cfg](BGW02.txt.txt) |
| [LEAF01.cfg](LEAF01.txt.txt)    |-------------------------------|----------------------------|------------------------|
| [LEAF02.cfg](LEAF02.txt.txt)    |-------------------------------|----------------------------|------------------------|
| [LEAF03.cfg](LEAF03.txt.txt)    |-------------------------------|----------------------------|------------------------|
| [LEAF04.cfg](LEAF04.txt.txt)    |-------------------------------|----------------------------|------------------------|



## Мощности стенда не хватало для работы BFD пришлось отключить часть устройств.
## Выбран eBGP и выбрана не единая, а разные AS для SPINE  намеренно, чтобы взять наиболее сложный кейс. В случае единой AS на SPINE настроек бы потребовалось меньше.

## Подтверждение работоспособности Топологии:

### show bfd peers

#### dc01-pod01-spine01#
dc01-pod01-spine01#show bfd peers

VRF name: default

|DstAddr   |MyDisc     |YourDisc   |Interface/Transport |Type   |LastUp         |      |
|----------|-----------|-----------|--------------------|-------|---------------|------|
|10.11.3.1 |781865319  |905301080  |Ethernet1(17)       |normal |12/03/24       |09:09 |
|10.11.3.3 |702803546  |1777991436 |Ethernet2(18)       |normal |12/03/24       |09:10 |
|10.11.3.5 |2852931168 |4184444231 |Ethernet3(19)       |normal |12/03/24       |09:09 |
|10.11.3.7 |1000268329 |1038889858 |Ethernet4(20)       |normal |12/03/24       |09:09 |
|10.11.3.9 |289015528  |2075822374 |Ethernet5(21)       |normal |12/03/24       |10:01 |
|10.11.3.11|1896418641 |1954307089 |Ethernet6(22)       |normal |12/03/24       |10:01 |

|              |LastDown           |LastDiag          |State |
|--------------|-------------------|------------------|------|
|              |NA                 |No      Diagnostic|Up    |
|              |NA                 |No      Diagnostic|Up    |
|              |NA                 |No      Diagnostic|Up    |
|              |NA                 |No      Diagnostic|Up    |
|              |NA                 |No      Diagnostic|Up    |
|              |NA                 |No      Diagnostic|Up    |


#### dc01-pod01-spine02#

dc01-pod01-spine02#show bfd peers

VRF name: default

|DstAddr   |MyDisc    |YourDisc   |Interface/Transport |Type   |LastUp         |      |
|----------|----------|-----------|--------------------|-------|---------------|------|
|10.11.3.41|2699976320|1472873612 |Ethernet1(16)       |normal |12/03/24       |09:09 |
|10.11.3.43|3395767375|716321651  |Ethernet2(17)       |normal |12/03/24       |09:09 |
|10.11.3.45|2063858295|3974180735 |Ethernet3(18)       |normal |12/03/24       |09:09 |
|10.11.3.47|1302925306|534778853  |Ethernet4(19)       |normal |12/03/24       |09:09 |
|10.11.3.49|1467659710|3491795341 |Ethernet5(20)       |normal |12/03/24       |10:01 |
|10.11.3.51|4145999257|1541684558 |Ethernet6(21)       |normal |12/03/24       |10:01 |

|              |LastDown           |LastDiag|State     |      |
|--------------|-------------------|--------|----------|------|
|              |NA                 |No      |Diagnostic|Up    |
|              |NA                 |No      |Diagnostic|Up    |
|              |NA                 |No      |Diagnostic|Up    |
|              |NA                 |No      |Diagnostic|Up    |
|              |NA                 |No      |Diagnostic|Up    |
|              |NA                 |No      |Diagnostic|Up    |

### show isis neighbors

#### dc01-pod01-spine01#

show isis neighbors

|Instance|VRF    |System |Id  |Type     |Interface SNPA|State|Hold time|Circuit Id |
|--------|-------|-------|----|---------|--------------|-----|---------|----|
|ZEUS    |default|LEAF01 |L1L2|Ethernet1|P2P           |UP   |25       |0D  |   
|ZEUS    |default|LEAF02 |L1L2|Ethernet2|P2P           |UP   |22       |0D  |    
|ZEUS    |default|LEAF03 |L1L2|Ethernet3|P2P           |UP   |28       |0D  |    
|ZEUS    |default|LEAF04 |L1L2|Ethernet4|P2P           |UP   |21       |0D  |    
|ZEUS    |default|SLEAF01|L1L2|Ethernet5|P2P           |UP   |26       |0E  |    
|ZEUS    |default|SLEAF02|L1L2|Ethernet6|P2P           |UP   |23       |0E  |    

#### dc01-pod01-spine02#

show isis neighbors

|Instance|VRF    |System Id  |Type|Interface|SNPA|State|Hold time|Circuit Id |
|--------|-------|-----------|----|---------|----|----|----------|----|
|ZEUS    |default|LEAF01     |L1L2|Ethernet1|P2P |UP  |26        |0E  |    
|ZEUS    |default|LEAF02     |L1L2|Ethernet2|P2P |UP  |30        |0E  |    
|ZEUS    |default|LEAF03     |L1L2|Ethernet3|P2P |UP  |24        |0E  |    
|ZEUS    |default|LEAF04     |L1L2|Ethernet4|P2P |UP  |24        |0E  |    
|ZEUS    |default|SLEAF01    |L1L2|Ethernet5|P2P |UP  |23        |0F  |    
|ZEUS    |default|SLEAF02    |L1L2|Ethernet6|P2P |UP  |25        |0F  |   


### show isis database

#### dc01-pod01-spine01#
show isis database
IS-IS Instance: ZEUS VRF: default
IS-IS Level 1 Link State Database

|      |LSPID        |Seq   Num  |Cksum|Life |Length|IS |Flags|
|------|-------------|-----------|-----|-----|------|------|---|
|      |SPINE01.00-00|149431     |23698|511  |298   |L2    |<> |     
|      |SPINE02.00-00|150115     |57990|1164 |298   |L2    |<> |     
|      |LEAF01.00-00 |33906      |30291|545  |149   |L2    |<> |     
|      |LEAF02.00-00 |33920      |19945|862  |149   |L2    |<> |     
|      |LEAF03.00-00 |34036      |65151|1141 |149   |L2    |<> |     
|      |LEAF04.00-00 |34124      |30479|526  |136   |L2    |<> |     
|      |SLEAF01.00-00|6          |55103|794  |150   |L2    |<> |     
|      |SLEAF02.00-00|6          |38350|832  |150   |L2    |<> |     

IS-IS Level 2 Link State Database
|      |LSPID        |Seq   Num  |Cksum|Life |Length|IS    |Flags|
|------|-------------|-----------|-----|-----|------|------|---|
|      |SPINE01.00-00|117275     |16480|753  |444   |L2    |<> |     
|      |SPINE02.00-00|106968     |54635|1006 |444   |L2    |<> |     
|      |LEAF01.00-00 |107770     |13586|507  |367   |L2    |<> |     
|      |LEAF02.00-00 |107683     |19137|677  |367   |L2    |<> |     
|      |LEAF03.00-00 |107670     |3378 |1001 |367   |L2    |<> |     
|      |LEAF04.00-00 |108259     |52293|691  |363   |L2    |<> |    
|      |SLEAF01.00-00|8          |23827|765  |368   |L2    |<> |     
|      |SLEAF02.00-00|9          |2855 |937  |368   |L2    |<> |     


#### dc01-pod01-spine02#

show isis database
IS-IS Instance: ZEUS VRF: default
IS-IS Level 1 Link State Database
|LSPID|Seq          |Num   |Cksum|Life|Length|IS |Flags|
|-----|-------------|------|-----|----|------|---|-----|
|     |SPINE01.00-00|149432|57635|959 |298   |L2 |<>   |
|     |SPINE02.00-00|150115|57990|837 |298   |L2 |<>   |
|     |LEAF01.00-00 |33907 |15461|1107|149   |L2 |<>   |
|     |LEAF02.00-00 |33920 |19945|535 |149   |L2 |<>   |
|     |LEAF03.00-00 |34036 |65151|813 |149   |L2 |<>   |
|     |LEAF04.00-00 |34125 |58904|933 |136   |L2 |<>   |
|     |SLEAF01.00-00|6     |55103|467 |150   |L2 |<>   |
|     |SLEAF02.00-00|6     |38350|505 |150   |L2 |<>   |

IS-IS Level 2 Link State Database

|LSPID|Seq          |Num   |Cksum|Life|Length|IS |Flags|
|-----|-------------|------|-----|----|------|---|-----|
|     |SPINE01.00-00|117276|20235|1143|444   |L2 |<>   |
|     |SPINE02.00-00|106968|54635|678 |444   |L2 |<>   |
|     |LEAF01.00-00 |107771|3134 |908 |367   |L2 |<>   |
|     |LEAF02.00-00 |107683|19137|349 |367   |L2 |<>   |
|     |LEAF03.00-00 |107670|3378 |673 |367   |L2 |<>   |
|     |LEAF04.00-00 |108260|17919|1109|363   |L2 |<>   |
|     |SLEAF01.00-00|8     |23827|437 |368   |L2 |<>   |
|     |SLEAF02.00-00|9     |2855 |610 |368   |L2 |<>   |



## Проверяем доступность loopback1 SPINE02 c SPINE01:

dc01-pod01-spine01#ping 10.11.1.2
PING 10.11.1.2 (10.11.1.2) 72(100) bytes of data.
80 bytes from 10.11.1.2: icmp_seq=1 ttl=63 time=86.1 ms
80 bytes from 10.11.1.2: icmp_seq=2 ttl=63 time=81.1 ms
80 bytes from 10.11.1.2: icmp_seq=3 ttl=63 time=80.2 ms
80 bytes from 10.11.1.2: icmp_seq=4 ttl=63 time=76.2 ms
80 bytes from 10.11.1.2: icmp_seq=5 ttl=63 time=78.3 ms

--- 10.11.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 48ms
rtt min/avg/max/mdev = 76.225/80.420/86.186/3.356 ms, pipe 5, ipg/ewma 12.092/83.120 ms

## Проверяем доступность loopback1 LEAF04 c SPINE01:

dc01-pod01-spine01#ping 10.11.1.6
PING 10.11.1.6 (10.11.1.6) 72(100) bytes of data.
80 bytes from 10.11.1.6: icmp_seq=1 ttl=64 time=11.5 ms
80 bytes from 10.11.1.6: icmp_seq=2 ttl=64 time=13.1 ms
80 bytes from 10.11.1.6: icmp_seq=3 ttl=64 time=10.3 ms
80 bytes from 10.11.1.6: icmp_seq=4 ttl=64 time=7.96 ms
80 bytes from 10.11.1.6: icmp_seq=5 ttl=64 time=13.8 ms

--- 10.11.1.6 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 59ms
rtt min/avg/max/mdev = 7.964/11.370/13.850/2.096 ms, ipg/ewma 14.913/11.457 ms
