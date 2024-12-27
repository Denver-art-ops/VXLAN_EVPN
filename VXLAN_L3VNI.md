
# Настройка L3VNI на фабрике и сравнение Assimetric и Symmetric IRB

### Цель:
Посмотреть два режима работы маршрутизации в Фабрике: <br>
Assymetric IRB  (требует наличия на каждом коммутаторе обоих SVI для маршрутизации между ними. <br>
Symmetric IRB (требует наличия L3VNI для связи VLAN между собой).<br>
Настройка L3VNI на фабрике.<br>
Проверка L3 связности.<br>

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

### VLAN 10 и VLAN 20 созданы только на Leaf01 и Leaf03 для проверки Assimetric IRB. Для них VRF и L3VNI не создавались.

dc01-pod01-leaf01#  <br>

interface Vlan10  <br>
   ip address 10.88.88.2/24  <br>
   ip virtual-router address 10.88.88.1/24  <br>
!  <br>
interface Vlan20  <br>
   ip address 10.10.10.2/24  <br>
   ip virtual-router address 10.10.10.1/24  <br>

dc01-pod01-leaf02#  <br>

interface Vlan10 <br>
   ip address 10.88.88.3/24 <br>
   ip virtual-router address 10.88.88.1/24 <br>
! <br>
interface Vlan20 <br>
   ip address 10.10.10.3/24 <br>
   ip virtual-router address 10.10.10.1/24 <br>

### VLAN 30 и VLAN 40   созданы только на Leaf02 и Leaf04 для проверки работы Symmetric IRB.  Для них создан VRF СUSTOMER_L3VNI и собственно L3VNI.

фрагменты дополнительной настройки LEAF02: <br>

interface Vlan30 <br>
   vrf CUSTOMER_L3VNI <br>
   ip address 10.30.30.2/24 <br>
   ip virtual-router address 10.30.30.1/24 <br>
! <br>
interface Vlan40 <br>
   vrf CUSTOMER_L3VNI <br>
   ip address 10.40.40.2/24 <br>
   ip virtual-router address 10.40.40.1/24 <br>
! <br>
interface Vxlan1 <br>
   description =VXLAN= <br>
   vxlan source-interface Loopback1 <br>
   vxlan udp-port 4789 <br>
   vxlan vlan 10 vni 100010 <br>
   vxlan vlan 30 vni 100030 <br>
   vxlan vlan 40 vni 100040 <br>
   vxlan vrf CUSTOMER_L3VNI vni 1000777 <br>
   vxlan learn-restrict any <br>
! <br>
ip virtual-router mac-address 00:00:00:00:00:10 <br>
! <br>
ip routing <br>
ip routing vrf CUSTOMER_L3VNI <br>
<br>
router bgp 4259905001 <br>
   router-id 10.11.1.4 <br>
   ! <br>
   vlan 30 <br>
      rd 65001:100030 <br>
      route-target both 65001:30 <br>
      redistribute learned <br>
      redistribute static <br>
   ! <br>
   vlan 40 <br>
      rd 65001:100040 <br>
      route-target both 65001:40 <br>
      redistribute learned <br>
      redistribute static <br>
   !<br>
    vrf CUSTOMER_L3VNI <br>
      rd 10.11.1.4:1 <br>
      route-target import evpn 65000:1 <br>
      route-target export evpn 65000:1 <br>
      redistribute connected <br>

фрагменты дополнительной настройки LEAF04: <br>
! <br>
interface Vlan30 <br>
   vrf CUSTOMER_L3VNI <br>
   ip address 10.30.30.4/24 <br>
   ip virtual-router address 10.30.30.1/24 <br>
! <br>
interface Vlan40 <br>
   vrf CUSTOMER_L3VNI <br>
   ip address 10.40.40.4/24 <br>
   ip virtual-router address 10.40.40.1/24 <br>
! <br>
interface Vxlan1 <br>
   description =VXLAN= <br>
   vxlan source-interface Loopback1 <br>
   vxlan udp-port 4789 <br>
   vxlan vlan 10 vni 100010 <br>
   vxlan vlan 30 vni 100030 <br>
   vxlan vlan 40 vni 100040 <br>
   vxlan vrf CUSTOMER_L3VNI vni 1000777 <br>
   vxlan learn-restrict any <br>
! <br>
ip virtual-router mac-address 00:00:00:00:00:10 <br>
! <br>
ip routing <br>
ip routing vrf CUSTOMER_L3VNI <br>
! <br>
ip routing <br>
ip routing vrf CUSTOMER_L3VNI <br>
! <br>
router bgp 4259905003 <br>
   router-id 10.11.1.6 <br>
  ! <br>
   vlan 30 <br>
      rd 65001:100030 <br>
      route-target both 65001:30 <br>
      redistribute learned <br>
      redistribute static <br>
   ! <br>
   vlan 40 <br>
      rd 65001:100040 <br>
      route-target both 65001:40 <br>
      redistribute learned <br>
      redistribute static <br>
   ! <br>
   vrf CUSTOMER_L3VNI <br>
      rd 10.11.1.6:1 <br>
      route-target import evpn 65000:1 <br>
      route-target export evpn 65000:1 <br>
      redistribute connected <br>
! <br>
end <br>
dc01-pod01-leaf04# <br>


## Подтверждение работоспособности Assimetric IRB

Пингуем с интерфейса в VLAN20  c ip адресом 10.10.10.2 с LEAF01  адрес в VLAN 10 на LEAF03  10.88.88.3 <br>

dc01-pod01-leaf01#ping 10.88.88.3 source 10.10.10.2 <br>
PING 10.88.88.3 (10.88.88.3) from 10.10.10.2 : 72(100) bytes of data. <br>
80 bytes from 10.88.88.3: icmp_seq=1 ttl=64 time=185 ms <br>
80 bytes from 10.88.88.3: icmp_seq=2 ttl=64 time=446 ms <br>
80 bytes from 10.88.88.3: icmp_seq=3 ttl=64 time=452 ms <br>
80 bytes from 10.88.88.3: icmp_seq=4 ttl=64 time=460 ms <br>
80 bytes from 10.88.88.3: icmp_seq=5 ttl=64 time=462 ms <br>

--- 10.88.88.3 ping statistics --- <br>
5 packets transmitted, 5 received, 0% packet loss, time 53ms <br>
rtt min/avg/max/mdev = 185.248/401.526/462.780/108.302 ms, pipe 5, ipg/ewma 13.250/297.516 ms <br>
dc01-pod01-leaf01#<br>

Проверяем доступность с ПК, включенного в первый LEAF01 в VLAN 10 IP адресов на LEAF03 <br>
<br>
VPCS> ip 10.88.88.88/24 10.88.88.1 <br>
Checking for duplicate address... <br>
VPCS : 10.88.88.88 255.255.255.0 gateway 10.88.88.1 <br>
<br>
VPCS> <br>
VPCS> ping 10.10.10.3<br>
<br>
84 bytes from 10.10.10.3 icmp_seq=1 ttl=64 time=67.369 ms <br>
84 bytes from 10.10.10.3 icmp_seq=2 ttl=64 time=101.972 ms <br>
84 bytes from 10.10.10.3 icmp_seq=3 ttl=64 time=212.186 ms <br> 
84 bytes from 10.10.10.3 icmp_seq=4 ttl=64 time=57.238 ms <br>
84 bytes from 10.10.10.3 icmp_seq=5 ttl=64 time=57.698 ms <br>

Смотрим ARP таблицу на LEAF01: <br>

dc01-pod01-leaf01#show ip arp <br>
Address         Age (sec)  Hardware Addr   Interface <br>
10.11.3.0         0:00:02  5000.00d7.ee0b  Ethernet1 <br>
10.11.3.40        0:00:00  5000.00cb.38c2  Ethernet2 <br>
10.88.88.3        0:17:51  5000.0015.f4e8  Vlan10, not learned <br>
10.88.88.88       0:02:12  0050.7966.6812  Vlan10, Ethernet3 <br>
10.10.10.3        0:18:15  5000.0015.f4e8  Vlan20, not learned <br>


## Подтверждение работоспособности L3VNI:

### show vxlan vtep

dc01-pod01-leaf02#show vxlan vtep <br>
Remote VTEPS for Vxlan1: <br>
<br>
VTEP            Tunnel Type(s) <br>
--------------- -------------- <br>
10.11.1.3       flood <br>
10.11.1.5       flood <br>
10.11.1.6       flood, unicast <br>
10.11.2.3       flood <br>
10.11.2.5       flood <br>
<br>
Total number of remote VTEPS:  5 <br>

dc01-pod01-leaf04#show vxlan vtep  <br>
VTEP            Tunnel Type(s)  <br>
--------------- --------------  <br>
10.11.1.4       unicast, flood  <br>
 <br>
Total number of remote VTEPS:  1  <br>


##### show bgp evpn route-type ip-prefix ipv4

dc01-pod01-leaf02#show bgp evpn route-type ip-prefix ipv4 <br>
BGP routing table information for VRF default <br>
Router identifier 10.11.1.4, local AS number 4259905001 <br>
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP <br>
                    c - Contributing to ECMP, % - Pending BGP convergence <br>
Origin codes: i - IGP, e - EGP, ? - incomplete <br>
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop <br>
 <br>
          Network                Next Hop              Metric  LocPref Weight  Path <br>
 * >      RD: 10.11.1.4:1 ip-prefix 10.30.30.0/24<br>
                                 -                     -       -       0       i <br>
 * >Ec    RD: 10.11.1.6:1 ip-prefix 10.30.30.0/24<br> 
                                 10.11.1.6             -       100     0       4259840001 4259905003 i <br>
 *  ec    RD: 10.11.1.6:1 ip-prefix 10.30.30.0/24<br> 
                                 10.11.1.6             -       100     0       4259840002 4259905003 i <br>
 * >      RD: 10.11.1.4:1 ip-prefix 10.40.40.0/24<br>  
                                 -                     -       -       0       i <br>
 * >Ec    RD: 10.11.1.6:1 ip-prefix 10.40.40.0/24<br> 
                                 10.11.1.6             -       100     0       4259840001 4259905003 i <br>
 *  ec    RD: 10.11.1.6:1 ip-prefix 10.40.40.0/24<br> 
                                 10.11.1.6             -       100     0       4259840002 4259905003 i <br>
<br>
<br>
dc01-pod01-leaf04#show bgp evpn route-type ip-prefix ipv4 <br>
BGP routing table information for VRF default <br>
Router identifier 10.11.1.6, local AS number 4259905003 <br>
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP <br>
                    c - Contributing to ECMP, % - Pending BGP convergence <br>
Origin codes: i - IGP, e - EGP, ? - incomplete <br>
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop <br>
<br>
          Network                Next Hop              Metric  LocPref Weight  Path <br>
 * >Ec    RD: 10.11.1.4:1 ip-prefix 10.30.30.0/24<br>
                                 10.11.1.4             -       100     0       4259840001 4259905001 i <br>
 *  ec    RD: 10.11.1.4:1 ip-prefix 10.30.30.0/24 <br>
                                 10.11.1.4             -       100     0       4259840002 4259905001 i <br>
 * >      RD: 10.11.1.6:1 ip-prefix 10.30.30.0/24 <br>
                                 -                     -       -       0       i <br>
 * >Ec    RD: 10.11.1.4:1 ip-prefix 10.40.40.0/24<br>
                                 10.11.1.4             -       100     0       4259840001 4259905001 i <br>
 *  ec    RD: 10.11.1.4:1 ip-prefix 10.40.40.0/24 <br>
                                 10.11.1.4             -       100     0       4259840002 4259905001 i <br>
 * >      RD: 10.11.1.6:1 ip-prefix 10.40.40.0/24<br>
                                 -                     -       -       0       i <br>

#### Проверяем как были изучены сети: 

dc01-pod01-leaf02#show ip route vrf  CUSTOMER_L3VNI <br>

VRF: CUSTOMER_L3VNI <br>
Codes: C - connected, S - static, K - kernel, <br>
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1, <br>
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1, <br>
       N2 - OSPF NSSA external type2, B - Other BGP Routes, <br>
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1, <br>
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate, <br>
       A O - OSPF Summary, NG - Nexthop Group Static Route, <br>
       V - VXLAN Control Service, M - Martian, <br>
       DH - DHCP client installed default route, <br>
       DP - Dynamic Policy Route, L - VRF Leaked, <br>
       G  - gRIBI, RC - Route Cache Route <br>

Gateway of last resort is not set <br>

 C        10.30.30.0/24 is directly connected, Vlan30 <br>
 C        10.40.40.0/24 is directly connected, Vlan40 <br>

### Убираем IP адреса с IN VLAN 40 чтобы убедиться, что L3VNI работает:

dc01-pod01-leaf02#conf t <br>
dc01-pod01-leaf02(config)#interface vlan 40 <br>
dc01-pod01-leaf02(config-if-Vl40)#no ip address <br>
dc01-pod01-leaf02(config-if-Vl40)#no ip virtual-router address <br>
dc01-pod01-leaf02(config-if-Vl40)# <br>
dc01-pod01-leaf02# <br>
dc01-pod01-leaf02# <br>
dc01-pod01-leaf02#show ip route vrf  CUSTOMER_L3VNI <br>
<br>
VRF: CUSTOMER_L3VNI  <br>
Codes: C - connected, S - static, K - kernel, <br>
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1, <br>
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1, <br>
       N2 - OSPF NSSA external type2, B - Other BGP Routes, <br>
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1, <br>
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate, <br>
       A O - OSPF Summary, NG - Nexthop Group Static Route, <br>
       V - VXLAN Control Service, M - Martian, <br>
       DH - DHCP client installed default route, <br>
       DP - Dynamic Policy Route, L - VRF Leaked, <br>
       G  - gRIBI, RC - Route Cache Route <br>
 <br>
Gateway of last resort is not set <br>
 <br>
 C        10.30.30.0/24 is directly connected, Vlan30 <br>
 B E      10.40.40.0/24 [200/0] via VTEP 10.11.1.6 VNI 1000777 router-mac 50:00:00:72:8b:31 local-interface Vxlan1 <br>

 

#### Пинги с интерфейсов LEAF02 на интерфейсы LEAF04

dc01-pod01-leaf02#ping vrf CUSTOMER_L3VNI 10.40.40.4 source 10.30.30.2 <br>
PING 10.40.40.4 (10.40.40.4) from 10.30.30.2 : 72(100) bytes of data. <br>
80 bytes from 10.40.40.4: icmp_seq=1 ttl=64 time=106 ms <br>
80 bytes from 10.40.40.4: icmp_seq=2 ttl=64 time=96.7 ms <br>
80 bytes from 10.40.40.4: icmp_seq=3 ttl=64 time=115 ms <br>
80 bytes from 10.40.40.4: icmp_seq=4 ttl=64 time=117 ms <br>
80 bytes from 10.40.40.4: icmp_seq=5 ttl=64 time=119 ms <br>

--- 10.40.40.4 ping statistics --- <br>
5 packets transmitted, 5 received, 0% packet loss, time 51ms<br>
rtt min/avg/max/mdev = 96.785/111.205/119.204/8.405 ms, pipe 5, ipg/ewma 12.944/109.525 ms <br>


dc01-pod01-leaf02#ping vrf CUSTOMER_L3VNI 10.30.30.4 source 10.40.40.2 <br>
PING 10.30.30.4 (10.30.30.4) from 10.40.40.2 : 72(100) bytes  of data. <br>
80 bytes from 10.30.30.4: icmp_seq=1 ttl=64 time=103 ms <br>
80 bytes from 10.30.30.4: icmp_seq=2 ttl=64 time=98.2 ms <br>
80 bytes from 10.30.30.4: icmp_seq=3 ttl=64 time=117 ms <br>
80 bytes from 10.30.30.4: icmp_seq=4 ttl=64 time=121 ms <br>
80 bytes from 10.30.30.4: icmp_seq=5 ttl=64 time=129 ms <br>

--- 10.30.30.4 ping statistics --- <br>
5 packets transmitted, 5 received, 0% packet loss, time 46ms <br>
rtt min/avg/max/mdev = 98.279/113.970/129.298/11.413 ms, pipe 5, ipg/ewma 11.606/109.692 ms <br>

 
#### Пинги в сторону LEAF02 с ПК, включенного в LEAF04 <br>
<br>
VPCS> ip 10.30.30.30/24 10.30.30.1<br>
Checking for duplicate address...<br>
VPCS : 10.30.30.30 255.255.255.0 gateway 10.30.30.1 <br>
<br>


VPCS> ping 10.30.30.2 <br>
<br>
84 bytes from 10.30.30.2 icmp_seq=1 ttl=63 time=41.829 ms <br>
84 bytes from 10.30.30.2 icmp_seq=2 ttl=63 time=52.854 ms <br>
84 bytes from 10.30.30.2 icmp_seq=3 ttl=63 time=70.110 ms <br>
84 bytes from 10.30.30.2 icmp_seq=4 ttl=63 time=36.718 ms <br>
84 bytes from 10.30.30.2 icmp_seq=5 ttl=63 time=46.140 ms <br>
 <br>
VPCS> ping 10.40.40.2 <br>
<br>
84 bytes from 10.40.40.2 icmp_seq=1 ttl=63 time=737.575 ms <br>
84 bytes from 10.40.40.2 icmp_seq=2 ttl=63 time=80.171 ms <br>
84 bytes from 10.40.40.2 icmp_seq=3 ttl=63 time=71.114 ms <br>
84 bytes from 10.40.40.2 icmp_seq=4 ttl=63 time=69.663 ms <br>
84 bytes from 10.40.40.2 icmp_seq=5 ttl=63 time=42.761 ms <br>




##### dc01-pod01-leaf01
dc01-pod01-leaf01#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.11.1.3, local AS number 4259905000
Neighbor Status Codes: m - Under maintenance

|      |Description|Neighbor |V  |AS        |MsgRcvd|MsgSent|InQ|OutQ|Up/Down |State|PfxRcd|PfxAcc|
|------|-----------|---------|---|----------|-------|-------|---|----|--------|-----|------|------|
|      |SPINE01_Lo1|10.11.1.1|4  |4259840001|137    |109    |0  |0   |00:30:55|Estab|1     |1     |
|      |SPINE02_Lo1|10.11.1.2|4  |4259840002|109    |110    |0  |0   |00:38:18|Estab|1     |1     |

##### dc01-pod01-leaf02
dc01-pod01-leaf02#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.11.1.4, local AS number 4259905001
Neighbor Status Codes: m - Under maintenance

|      |Description|Neighbor |V  |AS        |MsgRcvd|MsgSent|InQ|OutQ|Up/Down |State|PfxRcd|PfxAcc|
|------|-----------|---------|---|----------|-------|-------|---|----|--------|-----|------|------|
|      |SPINE01_Lo1|10.11.1.1|4  |4259840001|227    |231    |0  |0   |00:40:57|Estab|2     |2     |
|      |SPINE02_Lo1|10.11.1.2|4  |4259840002|196    |194    |0  |0   |00:40:48|Estab|2     |2     |

##### dc01-pod01-leaf03
dc01-pod01-leaf03#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.11.1.5, local AS number 4259905002
Neighbor Status Codes: m - Under maintenance

|      |Description|Neighbor |V  |AS        |MsgRcvd|MsgSent|InQ|OutQ|Up/Down |State|PfxRcd|PfxAcc|
|------|-----------|---------|---|----------|-------|-------|---|----|--------|-----|------|------|
|      |SPINE01_Lo1|10.11.1.1|4  |4259840001|279    |246    |0  |0   |00:41:51|Estab|1     |1     |
|      |SPINE02_Lo1|10.11.1.2|4  |4259840002|266    |220    |0  |0   |00:41:58|Estab|1     |1     |

##### dc01-pod01-leaf04
dc01-pod01-leaf04#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.11.1.6, local AS number 4259905003
Neighbor Status Codes: m - Under maintenance

|      |Description|Neighbor |V  |AS        |MsgRcvd|MsgSent|InQ|OutQ|Up/Down |State|PfxRcd|PfxAcc|
|------|-----------|---------|---|----------|-------|-------|---|----|--------|-----|------|------|
|      |SPINE01_Lo1|10.11.1.1|4  |4259840001|269    |247    |0  |0   |00:42:47|Estab|2     |2     |
|      |SPINE02_Lo1|10.11.1.2|4  |4259840002|207    |197    |0  |0   |00:43:10|Estab|2     |2     |

##### dc01-pod01-Sleaf01
dc01-pod01-Sleaf01#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.11.1.7, local AS number 4259905101
Neighbor Status Codes: m - Under maintenance

|      |Description|Neighbor |V  |AS        |MsgRcvd|MsgSent|InQ|OutQ|Up/Down |State|PfxRcd|PfxAcc|
|------|-----------|---------|---|----------|-------|-------|---|----|--------|-----|------|------|
|      |SPINE01_Lo1|10.11.1.1|4  |4259840001|169    |192    |0  |0   |00:43:15|Estab|2     |2     |
|      |SPINE02_Lo1|10.11.1.2|4  |4259840002|151    |189    |0  |0   |00:43:05|Estab|2     |2     |

##### dc01-pod01-Sleaf02
dc01-pod01-Sleaf02#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.11.1.8, local AS number 4259905102
Neighbor Status Codes: m - Under maintenance

|      |Description|Neighbor |V  |AS        |MsgRcvd|MsgSent|InQ|OutQ|Up/Down |State|PfxRcd|PfxAcc|
|------|-----------|---------|---|----------|-------|-------|---|----|--------|-----|------|------|
|      |SPINE01_Lo1|10.11.1.1|4  |4259840001|214    |194    |0  |0   |00:44:17|Estab|2     |2     |
|      |SPINE02_Lo1|10.11.1.2|4  |4259840002|252    |181    |0  |0   |00:43:53|Estab|2     |2     |
  

##### dc01-pod01-leaf01#show interfaces vxlan 1

|Vxlan1 is up                                                    |line protocol is up (connected)|
|----------------------------------------------------------------|-------------------------------|
|  Hardware is Vxlan                                             |                               |
|  Description: =VXLAN=                                          |                               |
|  Source interface is Loopback2 and is active with 10.11.2.3    |                               |
|  Listening on UDP port 4789                                    |                               |
|  Replication/Flood Mode is headend with Flood List Source: EVPN|                               |
|  Remote MAC learning via EVPN                                  |                               |
|  VNI mapping to VLANs                                          |                               |
|  Static VLAN to VNI mapping is                                 |                               |
|    [10                                                         | 100010]                       |
|  Note: All Dynamic VLANs used by VCS are internal VLANs.       |                               |
|        Use 'show vxlan vni' for details.                       |                               |
|  Static VRF to VNI mapping is not configured                   |                               |
|  Headend replication flood vtep list is:                       |                               |
|    10 10.11.2.5                                                |                               |
|  Shared Router MAC is 0000.0000.0000                           |                               |

##### dc01-pod01-leaf03#show interfaces vxlan 1

|Vxlan1 is up                                                    |line protocol is up (connected)|
|----------------------------------------------------------------|-------------------------------|
|  Hardware is Vxlan                                             |                               |
|  Description: =VXLAN=                                          |                               |
|  Source interface is Loopback2 and is active with 10.11.2.5    |                               |
|  Listening on UDP port 4789                                    |                               |
|  Replication/Flood Mode is headend with Flood List Source: EVPN|                               |
|  Remote MAC learning via EVPN                                  |                               |
|  VNI mapping to VLANs                                          |                               |
|  Static VLAN to VNI mapping is                                 |                               |
|    [10                                                         | 100010]                       |
|  Note: All Dynamic VLANs used by VCS are internal VLANs.       |                               |
|        Use 'show vxlan vni' for details.                       |                               |
|  Static VRF to VNI mapping is not configured                   |                               |
|  Headend replication flood vtep list is:                       |                               |
|    10 10.11.2.3                                                |                               |
|  Shared Router MAC is 0000.0000.0000                           |                               |

##### dc01-pod01-leaf01#show vxlan vtep

|Remote VTEPS for Vxlan1|              |
|-----------------------|--------------|
|VTEP            Tunnel Type(s)|              |
|--------------- --------------|              |
|10.11.2.5       flood  |              |
|Total number of remote VTEPS|  1           |


##### dc01-pod01-leaf03#show vxlan vtep

|Remote VTEPS for Vxlan1       |      |
|------------------------------|------|
|VTEP            Tunnel Type(s)|      |
|--------------- --------------|      |
|10.11.2.5       flood         |      |
|Total number of remote VTEPS  |  1   |


#### dc01-pod01-leaf03#
show mac address-table

|------------------------------------------------------------------  |
|--------------------------------------------------------------------|
|Vlan    Mac Address       Type        Ports      Moves   Last Move  |
|----    -----------       ----        -----      -----   ---------  |
|   1    0000.0000.0010    STATIC      Cpu                           |
|  10    0000.0000.0010    STATIC      Cpu                           |
|  10    0050.7966.6812    DYNAMIC     Vx1        1       0:00:22 ago|
|  10    0050.7966.6815    DYNAMIC     Et3        1       0:00:23 ago|
|Total Mac Addresses for this criterion: 4                           |
|          Multicast Mac Address Table                               |
|------------------------------------------------------------------  |
|Vlan    Mac Address       Type        Ports                         |
|----    -----------       ----        -----                         |
|Total Mac Addresses for this criterion: 0                           |


#### dc01-pod01-leaf01
#show vxlan address-table

|----------------------------------------------------------------------   |
|-------------------------------------------------------------------------|
|VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move  |
|----  -----------     ----      ---  ----             -----   ---------  |
|  10  0050.7966.6815  EVPN      Vx1  10.11.2.5        1       0:01:40 ago|
|Total Remote Mac Addresses for this criterion: 1                         |


#### dc01-pod01-leaf01# show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.11.1.3, local AS number 4259905000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

|      |Network  |Next|Hop|Metric      |LocPref   |Weight        |Path|
|------|---------|----|---|------------|----------|--------------|----|
|      |*        |>   |RD:|10.11.1.3:10|mac-ip    |0050.7966.6812|    |
|      |-        |-   |-  |0           |i         |              |    |
|      |*        |>Ec |RD:|10.11.1.5:10|mac-ip    |0050.7966.6815|    |
|      |10.11.2.5|-   |100|0           |4259840002|4259905002    |i   |
|      |*        |ec  |RD:|10.11.1.5:10|mac-ip    |0050.7966.6815|    |
|      |10.11.2.5|-   |100|0           |4259840001|4259905002    |i   |

#### dc01-pod01-leaf03#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.11.1.5, local AS number 4259905002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

|      |Network  |Next|Hop|Metric      |LocPref   |Weight        |Path|
|------|---------|----|---|------------|----------|--------------|----|
|      |*        |>Ec |RD:|10.11.1.3:10|mac-ip    |0050.7966.6812|    |
|      |10.11.2.3|-   |100|0           |4259840001|4259905000    |i   |
|      |*        |ec  |RD:|10.11.1.3:10|mac-ip    |0050.7966.6812|    |
|      |10.11.2.3|-   |100|0           |4259840002|4259905000    |i   |
|      |*        |>   |RD:|10.11.1.5:10|mac-ip    |0050.7966.6815|    |
|      |-        |-   |-  |0           |i         |              |    |


### Главный критерий - IP связность между хостами 10.88.88.3 и 10.88.88.5

|VPCS>|ping |10.88.88.5|FIELD4    |FIELD5    |FIELD6|FIELD7      |FIELD8|
|-----|-----|----------|----------|----------|------|------------|------|
|84   |bytes|from      |10.88.88.5|icmp_seq=1|ttl=64|time=121.441|ms    |
|84   |bytes|from      |10.88.88.5|icmp_seq=2|ttl=64|time=74.594 |ms    |
|84   |bytes|from      |10.88.88.5|icmp_seq=3|ttl=64|time=46.078 |ms    |
|84   |bytes|from      |10.88.88.5|icmp_seq=4|ttl=64|time=60.178 |ms    |
|84   |bytes|from      |10.88.88.5|icmp_seq=5|ttl=64|time=186.543|ms    |


### Как и ожидалось отключение IP адреса на VLAN интерфейсе на связность не влияет.  Его можно вообще не настраивать

###  Далее для VLAN 10 (VNI 10010) был настроен AnyCast GW. (виртуальный IP и виртуальный MAC)
