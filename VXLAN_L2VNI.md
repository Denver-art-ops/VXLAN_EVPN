# Настройка MP BGP (eBGP) в Overlay и настройка L2VNI  (VXLAN)

### Цель:
Настройка MP-BGP в Overlay на фабрике.<br>
Настройка L2VNI на фабрике.<br>
Проверка L2 связности.<br>

### Принципы назначения IP адресов, адресное пространство
Описаны в документе: [README.md](README.md)

### Итоговая схема
![Topology_eBGP.png](Topology_eBGP.png)

## Конфигурации устройств:

|                             |                               |                            |                        |
|-----------------------------|-------------------------------|----------------------------|------------------------|
| [SPINE01.cfg](SPINE01.txt)  |   [BLEAF01.cfg](BLEAF01.txt)  | [SLEAF01.cfg](SLEAF01.txt) | [BGW01.cfg](BGW01.txt) |
| [SPINE02.cfg](SPINE02.txt)  |   [BLEAF02.cfg](BLEAF02.txt)  | [SLEAF02.cfg](SLEAF02.txt) | [BGW02.cfg](BGW02.txt) |
| [LEAF01.cfg](LEAF01.txt)    |-------------------------------|----------------------------|------------------------|
| [LEAF02.cfg](LEAF02.txt)    |-------------------------------|----------------------------|------------------------|
| [LEAF03.cfg](LEAF03.txt)    |-------------------------------|----------------------------|------------------------|
| [LEAF04.cfg](LEAF04.txt)    |-------------------------------|----------------------------|------------------------|




### Выбран eBGP и выбрана не единая, а разные AS для SPINE  намеренно, чтобы взять наиболее сложный кейс. В случае единой AS на SPINE настроек бы потребовалось меньше.

### Bleaf и BGW пришлось отключить, так как стенд не справляется с нагрузкой

### as-path multypath relax включен по умолчанию

###  maximum path 2 настроен, так как SPINE в схеме всего 2

### На SPINE настроен next hop unchanged

### На разных устройства намеренно BGPv4 включен по разному (через network и через neightbor activate)

### В  Leaf1 включен хост с адресом 10.88.88.3/24 шлюз по умолчанию для хоста намеренно не был указан.

### В  Leaf1 включен хост с адресом 10.88.88.5/24 шлюз по умолчанию для хоста намеренно не был указан.

##



## Подтверждение работоспособности L2VNI:

### show bgp evpn summary

##### dc01-pod01-spine01
dc01-pod01-spine01#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.11.1.1, local AS number 4259840001
Neighbor Status Codes: m - Under maintenance

|      |Description|Neighbor  |V  |AS        |MsgRcvd|MsgSent|InQ|OutQ|Up/Down |State |PfxRcd|PfxAcc|
|------|-----------|----------|---|----------|-------|-------|---|----|--------|------|------|------|
|      |LEAF01_Lo1 |10.11.1.3 |4  |4259905000|89     |120    |0  |0   |00:27:41|Estab |2     |2     |
|      |LEAF02_Lo1 |10.11.1.4 |4  |4259905001|177    |307    |0  |0   |00:36:26|Estab |1     |1     |
|      |LEAF03_Lo1 |10.11.1.5 |4  |4259905002|164    |287    |0  |0   |00:36:17|Estab |2     |2     |
|      |LEAF04_Lo1 |10.11.1.6 |4  |4259905003|192    |312    |0  |0   |00:36:15|Estab |1     |1     |
|      |SLEAF01_Lo1|10.11.1.7 |4  |4259905101|146    |254    |0  |0   |00:35:43|Estab |1     |1     |
|      |SLEAF02_Lo1|10.11.1.8 |4  |4259905102|145    |284    |0  |0   |00:36:07|Estab |1     |1     |
|      |BLEAF01_Lo1|10.11.1.9 |4  |4259905201|0      |0      |0  |0   |00:57:45|Active|      |      |
|      |BLEAF02_Lo1|10.11.1.10|4  |4259905202|0      |0      |0  |0   |00:57:45|Active|      |      |
|      |BGW01_Lo1  |10.11.1.11|4  |4259905301|0      |0      |0  |0   |00:57:45|Active|      |      |
|      |BGW02_Lo1  |10.11.1.12|4  |4259905302|0      |0      |0  |0   |00:57:44|Active|      |      |


##### dc01-pod01-spine02
dc01-pod01-spine01#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.11.1.2, local AS number 4259840002
Neighbor Status Codes: m - Under maintenance

|Description|Neighbor   |V         |AS |MsgRcvd   |MsgSent|InQ|OutQ|Up/Down|State   |PfxRcd|PfxAcc|FIELD13|
|-----------|-----------|----------|---|----------|-------|---|----|-------|--------|------|------|-------|
|           |LEAF01_Lo1 |10.11.1.3 |4  |4259905000|109    |110|0   |0      |00:36:48|Estab |1     |1      |
|           |LEAF02_Lo1 |10.11.1.4 |4  |4259905001|178    |209|0   |0      |00:38:02|Estab |1     |1      |
|           |LEAF03_Lo1 |10.11.1.5 |4  |4259905002|192    |284|0   |0      |00:38:08|Estab |1     |1      |
|           |LEAF04_Lo1 |10.11.1.6 |4  |4259905003|174    |220|0   |0      |00:38:21|Estab |1     |1      |
|           |SLEAF01_Lo1|10.11.1.7 |4  |4259905101|165    |173|0   |0      |00:37:19|Estab |1     |1      |
|           |SLEAF02_Lo1|10.11.1.8 |4  |4259905102|152    |259|0   |0      |00:37:18|Estab |1     |1      |
|           |BLEAF01_Lo1|10.11.1.9 |4  |4259905201|0      |0  |0   |0      |01:05:39|Active|      |       |
|           |BLEAF02_Lo1|10.11.1.10|4  |4259905202|0      |0  |0   |0      |01:05:39|Active|      |       |
|           |BGW01_Lo1  |10.11.1.11|4  |4259905301|0      |0  |0   |0      |01:05:41|Active|      |       |
|           |BGW02_Lo1  |10.11.1.12|4  |4259905302|0      |0  |0   |0      |01:05:38|Active|      |       |


## Смотрим таблицы маршрутизации:

#### dc01-pod01-spine01#

dc01-pod01-spine01#show ip route

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

|C        10.11.1.1/32 is directly connected   |Loopback1  |
|----------------------------------------------|-----------|
| B E      10.11.1.2/32 [200/0] via 10.11.3.3  | Ethernet2 |
|                               via 10.11.3.13 | Ethernet7 |
| B E      10.11.1.3/32 [200/0] via 10.11.3.1  | Ethernet1 |
| B E      10.11.1.4/32 [200/0] via 10.11.3.3  | Ethernet2 |
| B E      10.11.1.5/32 [200/0] via 10.11.3.5  | Ethernet3 |
| B E      10.11.1.6/32 [200/0] via 10.11.3.7  | Ethernet4 |
| B E      10.11.1.7/32 [200/0] via 10.11.3.9  | Ethernet5 |
| B E      10.11.1.8/32 [200/0] via 10.11.3.11 | Ethernet6 |
| B E      10.11.1.9/32 [200/0] via 10.11.3.13 | Ethernet7 |
| B E      10.11.1.10/32 [200/0] via 10.11.3.15| Ethernet8 |
| B E      10.11.1.11/32 [200/0] via 10.11.3.17| Ethernet9 |
| B E      10.11.1.12/32 [200/0] via 10.11.3.19| Ethernet10|
| C        10.11.3.0/31 is directly connected  | Ethernet1 |
| C        10.11.3.2/31 is directly connected  | Ethernet2 |
| C        10.11.3.4/31 is directly connected  | Ethernet3 |
| C        10.11.3.6/31 is directly connected  | Ethernet4 |
| C        10.11.3.8/31 is directly connected  | Ethernet5 |
| C        10.11.3.10/31 is directly connected | Ethernet6 |
| C        10.11.3.12/31 is directly connected | Ethernet7 |
| C        10.11.3.14/31 is directly connected | Ethernet8 |
| C        10.11.3.16/31 is directly connected | Ethernet9 |
| C        10.11.3.18/31 is directly connected | Ethernet10|
| B E      10.11.3.40/31 [200/0] via 10.11.3.1 | Ethernet1 |
| B E      10.11.3.42/31 [200/0] via 10.11.3.3 | Ethernet2 |
| B E      10.11.3.44/31 [200/0] via 10.11.3.5 | Ethernet3 |
| B E      10.11.3.46/31 [200/0] via 10.11.3.7 | Ethernet4 |
| B E      10.11.3.48/31 [200/0] via 10.11.3.9 | Ethernet5 |
| B E      10.11.3.50/31 [200/0] via 10.11.3.11| Ethernet6 |
| B E      10.11.3.52/31 [200/0] via 10.11.3.13| Ethernet7 |
| B E      10.11.3.54/31 [200/0] via 10.11.3.15| Ethernet8 |
| B E      10.11.3.56/31 [200/0] via 10.11.3.17| Ethernet9 |
| B E      10.11.3.58/31 [200/0] via 10.11.3.19| Ethernet10|


#### dc01-pod01-spine02#

dc01-pod01-spine02#show ip route

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

|B E      10.11.1.1/32 [200/0] via 10.11.3.43  |Ethernet2  |
|----------------------------------------------|-----------|
|                               via 10.11.3.53 | Ethernet7 |
| C        10.11.1.2/32 is directly connected  | Loopback1 |
| B E      10.11.1.3/32 [200/0] via 10.11.3.41 | Ethernet1 |
| B E      10.11.1.4/32 [200/0] via 10.11.3.43 | Ethernet2 |
| B E      10.11.1.5/32 [200/0] via 10.11.3.45 | Ethernet3 |
| B E      10.11.1.6/32 [200/0] via 10.11.3.47 | Ethernet4 |
| B E      10.11.1.7/32 [200/0] via 10.11.3.49 | Ethernet5 |
| B E      10.11.1.8/32 [200/0] via 10.11.3.51 | Ethernet6 |
| B E      10.11.1.9/32 [200/0] via 10.11.3.53 | Ethernet7 |
| B E      10.11.1.10/32 [200/0] via 10.11.3.55| Ethernet8 |
| B E      10.11.1.11/32 [200/0] via 10.11.3.57| Ethernet9 |
| B E      10.11.1.12/32 [200/0] via 10.11.3.59| Ethernet10|
| B E      10.11.3.0/31 [200/0] via 10.11.3.41 | Ethernet1 |
| B E      10.11.3.2/31 [200/0] via 10.11.3.43 | Ethernet2 |
| B E      10.11.3.4/31 [200/0] via 10.11.3.45 | Ethernet3 |
| B E      10.11.3.6/31 [200/0] via 10.11.3.47 | Ethernet4 |
| B E      10.11.3.8/31 [200/0] via 10.11.3.49 | Ethernet5 |
| B E      10.11.3.10/31 [200/0] via 10.11.3.51| Ethernet6 |
| B E      10.11.3.12/31 [200/0] via 10.11.3.53| Ethernet7 |
| B E      10.11.3.14/31 [200/0] via 10.11.3.55| Ethernet8 |
| B E      10.11.3.16/31 [200/0] via 10.11.3.57| Ethernet9 |
| B E      10.11.3.18/31 [200/0] via 10.11.3.59| Ethernet10|
| C        10.11.3.40/31 is directly connected | Ethernet1 |
| C        10.11.3.42/31 is directly connected | Ethernet2 |
| C        10.11.3.44/31 is directly connected | Ethernet3 |
| C        10.11.3.46/31 is directly connected | Ethernet4 |
| C        10.11.3.48/31 is directly connected | Ethernet5 |
| C        10.11.3.50/31 is directly connected | Ethernet6 |
| C        10.11.3.52/31 is directly connected | Ethernet7 |
| C        10.11.3.54/31 is directly connected | Ethernet8 |
| C        10.11.3.56/31 is directly connected | Ethernet9 |
| C        10.11.3.58/31 is directly connected | Ethernet10|


## Проверяем доступность loopback1 SPINE02 c SPINE01:

dc01-pod01-spine01#ping 10.11.1.2
PING 10.11.1.2 (10.11.1.2) 72(100) bytes of data.
80 bytes from 10.11.1.2: icmp_seq=1 ttl=63 time=102 ms
80 bytes from 10.11.1.2: icmp_seq=2 ttl=63 time=115 ms
80 bytes from 10.11.1.2: icmp_seq=3 ttl=63 time=110 ms
80 bytes from 10.11.1.2: icmp_seq=4 ttl=63 time=102 ms
80 bytes from 10.11.1.2: icmp_seq=5 ttl=63 time=62.5 ms

--- 10.11.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 89ms
rtt min/avg/max/mdev = 62.520/98.690/115.708/18.801 ms, pipe 5, ipg/ewma 22.346/99.234 ms

## Проверяем доступность loopback1 LEAF04 c SPINE01:

dc01-pod01-spine01#ping 10.11.1.6
PING 10.11.1.6 (10.11.1.6) 72(100) bytes of data.
80 bytes from 10.11.1.6: icmp_seq=1 ttl=64 time=17.1 ms
80 bytes from 10.11.1.6: icmp_seq=2 ttl=64 time=21.7 ms
80 bytes from 10.11.1.6: icmp_seq=3 ttl=64 time=16.8 ms
80 bytes from 10.11.1.6: icmp_seq=4 ttl=64 time=10.4 ms
80 bytes from 10.11.1.6: icmp_seq=5 ttl=64 time=11.3 ms

--- 10.11.1.6 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 76ms
rtt min/avg/max/mdev = 10.409/15.500/21.762/4.175 ms, pipe 2, ipg/ewma 19.143/16.042 ms
