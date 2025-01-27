
# VxLAN Routing.

### Цель:
Реализовать маршрутизацию между "клиентами" через EVPN route-type 5 <br>
Подключить внешний маршрутизатор <br>
Организовать на нем передачу маршрутов между клиентами в VRF GREEN и  RED <br>

### Принципы назначения IP адресов, адресное пространство
Описаны в документе: [README.md](README.md)

### Итоговая схема
![EXT_FW.png](EXT_FW.png)

## Конфигурации устройств:

|                             |
|-----------------------------|
| [SPINE01.cfg](SPINE01.txt)  | 
| [SPINE02.cfg](SPINE02.txt)  | 
| [LEAF01.cfg](LEAF01.txt)    |
| [LEAF02.cfg](LEAF02.txt)    |
| [LEAF03.cfg](LEAF03.txt)    |
| [LEAF04.cfg](LEAF04.txt)    |
| [CUSTOMER.cfg](CUSTOMER.txt)|

### В качестве внешнего маршрутизатора воспользуемся уже существующим подключением маршрутизатора CUSTOMER.  Но в существующем Port-channel в транках добавим новые VLAN 301-302 в них на SVI поднимем BGP-пиринг между LEAF03-04 и CUSTOMER.

### Вместо того, чтобы автоматически "смешать" два VRF в Global таблице внешнего маршрутизатора CUSTOMER, "вытянем" по Option-A VRF GREEN и RED на стороннее устройство, чтобы иметь возможность манипулировать ликингом маршрутов между ними.
### Эмулируем своего рода ядро сети.

####  Этап 1 Создаем на LEAF03   VRF GREEN и RED, создаем SVI интерфейсы в этих VRF и добавляем ассоциированные VLAN в L2 VNI a VRF в L3 VNI.  Настраиваем пиринг с IP интерфейсами внешнего маршрутизатора CUSTOMER (192.168.1.200 в VLAN 301 и 192.168.2.200 в VLAN 302)

<details>
  <summary>Настройка LEAF03 </summary>
   
```
vlan 301
   name GREEN
!
vlan 302
   name RED
!
vrf instance GREEN
!
vrf instance RED
!
interface Port-Channel1
   switchport trunk allowed vlan add 301-302
!
interface Vlan301
   vrf GREEN
   ip address 192.168.1.39/24
   ip virtual-router address 192.168.1.1/24
!
interface Vlan302
   vrf RED
   ip address 192.168.2.39/24
   ip virtual-router address 192.168.2.1/24
!
interface Vxlan1
   description =VXLAN=
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 100010
   vxlan vlan 20 vni 100020
   vxlan vlan 201 vni 100201
   vxlan vlan 202 vni 100202
   vxlan vlan 301 vni 1000301
   vxlan vlan 302 vni 1000302
   vxlan vrf CUSTOMER_L3VNI vni 1000777
   vxlan vrf GREEN vni 1000888
   vxlan vrf RED vni 1000999
   vxlan learn-restrict any
!
ip routing vrf GREEN
ip routing vrf RED
!
router bgp 4259905002
!
   vlan 301
      rd 65001:1000301
      route-target both 301:301
   !
   vlan 302
      rd 65001:1000302
      route-target both 302:302
   !
   vrf GREEN
      rd 1000888:888
      route-target import evpn 301:1000888
      route-target export evpn 301:1000888
      neighbor 192.168.1.200 remote-as 65500
      neighbor 192.168.1.200 description GREEN_VRF-GW
      !
      address-family ipv4
         neighbor 192.168.1.200 activate
   !
   vrf RED
      rd 1000999:999
      route-target import evpn 302:1000999
      route-target export evpn 302:1000999
      neighbor 192.168.2.200 remote-as 65500
      neighbor 192.168.2.200 description RED_VRF-GW
      !
      address-family ipv4
         neighbor 192.168.2.200 activate
!
```
</details>

####  Этап 2 Создаем на LEAF04   VRF GREEN и RED, создаем SVI интерфейсы в этих VRF и добавляем ассоциированные VLAN в L2 VNI a VRF в L3 VNI.  Настраиваем пиринг с IP интерфейсами внешнего маршрутизатора CUSTOMER (192.168.1.200 в VLAN 301 и 192.168.2.200 в VLAN 302), также создаем два SVI в 501-502 VLAN и добавляем их в VRF GREEN и VRF RED (для проверки).

<details>
  <summary>Настройка LEAF04 </summary>
   
```
vlan 301
   name GREEN
!
vlan 302
   name RED
!
vlan 501-502

vrf instance GREEN
!
vrf instance RED
!
interface Port-Channel1
   switchport trunk allowed vlan add 301-302
!  
interface Vlan301
   vrf GREEN
   ip address 192.168.1.40/24
   ip virtual-router address 192.168.1.1/24
!
interface Vlan302
   vrf RED
   ip address 192.168.2.40/24
   ip virtual-router address 192.168.2.1/24
!
interface Vlan501
   vrf GREEN
   ip address 192.168.22.40/24
!
interface Vlan502
   vrf RED
   ip address 192.168.23.40/24
!
interface Vxlan1
   description =VXLAN=
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 100010
   vxlan vlan 30 vni 100030
   vxlan vlan 40 vni 100040
   vxlan vlan 201 vni 100201
   vxlan vlan 202 vni 100202
   vxlan vlan 301 vni 1000301
   vxlan vlan 302 vni 1000302
   vxlan vrf CUSTOMER_L3VNI vni 1000777
   vxlan vrf GREEN vni 1000888
   vxlan vrf RED vni 1000999
   vxlan learn-restrict any
!
ip routing vrf GREEN
ip routing vrf RED
!
router bgp 4259905003
  !
   vlan 301
      rd 65001:1000301
      route-target both 301:301
   !
   vlan 302
      rd 65001:1000302
      route-target both 302:302
  !
   vrf CUSTOMER_L3VNI
      rd 10.11.1.6:1
      route-target import evpn 65000:1
      route-target export evpn 65000:1
      redistribute connected
   !
   vrf GREEN
      rd 1000888:888
      route-target import evpn 301:1000888
      route-target export evpn 301:1000888
      neighbor 192.168.1.200 remote-as 65500
      neighbor 192.168.1.200 description GREEN_VRF-GW
      !
      address-family ipv4
         neighbor 192.168.1.200 activate
   !
   vrf RED
      rd 1000999:999
      route-target import evpn 302:1000999
      route-target export evpn 302:1000999
      neighbor 192.168.2.200 remote-as 65500
      neighbor 192.168.2.200 description RED_VRF-GW
      !
      address-family ipv4
         neighbor 192.168.2.200 activate
!
```
</details>

####  Этап 3 Создаем VLAN 301-302 в port-channel на стороне внешнего маршрутизатора CUSTOMER. Ассоциируем с этими VLAN локальные VRF RED и VRF GREEN. Создаем также два локальных VLAN 401-402 в VRF GREEN и RED (для проверки).
### Настраиваем BGP-пиринг с LEAF03 и LEAF04 на SVI-интерфейсах VLANов 301-302.
### Настраиваем route-leaking между VRF GREEN и VRF RED в AF EVPN локально на марашрутизаторе CUSTOMER

<details>
  <summary>Настройка CUSTOMER </summary>
   
```
!
vlan 301
   name GREEN
   trunk group ESI_LAG
!
vlan 302
   name RED
   trunk group ESI_LAG
!
vlan 401-402
!
vrf instance GREEN
!
vrf instance RED
!
interface Vlan301
   vrf GREEN
   ip address 192.168.1.200/24
!
interface Vlan302
   vrf RED
   ip address 192.168.2.200/24
!
interface Vlan401
   vrf GREEN
   ip address 192.168.3.200/24
!
interface Vlan402
   vrf RED
   ip address 192.168.4.200/24
!
interface Vxlan1
   vxlan source-interface Loopback0
   vxlan udp-port 4789
   vxlan vrf GREEN vni 20001
   vxlan vrf RED vni 10001
!
ip routing vrf GREEN
ip routing vrf RED
!
ip route 0.0.0.0/0 1.1.1.254
!
router bgp 65500
   router-id 8.8.8.8
   !
   vrf GREEN
      rd 2:2
      route-target import evpn 10:1
      route-target export evpn 20:1
      neighbor 192.168.1.39 remote-as 4259905002
      neighbor 192.168.1.40 remote-as 4259905003
      redistribute dynamic
      redistribute bgp leaked
      !
      address-family ipv4
         neighbor 192.168.1.40 activate
         neighbor 192.168.3.39 activate
         network 8.8.8.8/32
         network 192.168.1.0/24
         redistribute connected
   !
   vrf RED
      rd 8.8.8.8:3
      route-target import evpn 20:1
      route-target export evpn 10:1
      neighbor 192.168.2.39 remote-as 4259905002
      neighbor 192.168.2.40 remote-as 4259905003
      redistribute dynamic
      redistribute bgp leaked
      !
      address-family ipv4
         neighbor 192.168.2.39 activate
         neighbor 192.168.2.40 activate
         network 8.8.8.8/32
         network 192.168.2.0/24
         redistribute connected
!
```
</details>

### Проверяем состояние BGP между CUSTOMER и LEAF03-04 в VRF GREEN и  RED
<details>
  <summary>CUSTOMER#show ip bgp summary vrf GREEN </summary>
```
CUSTOMER#show ip bgp summary vrf GREEN
BGP summary information for VRF GREEN
Router identifier 192.168.3.200, local AS number 65500
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  192.168.1.39 4 4259905002       127       125    0    0 01:42:44 Estab   0      0
  192.168.1.40 4 4259905003       123       127    0    0 01:42:49 Estab   0      0
```
</details>

<details>
  <summary>CUSTOMER#show ip bgp summary vrf RED </summary>
```
CUSTOMER#show ip bgp summary vrf RED
BGP summary information for VRF RED
Router identifier 192.168.4.200, local AS number 65500
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  192.168.2.39 4 4259905002       124       127    0    0 01:43:40 Estab   0      0
  192.168.2.40 4 4259905003       128       128    0    0 01:43:38 Estab   0      0
```
</details>


### Проверяем, какие маршруты анонсируются CUSTOMER в сторону LEAF03-04 в VRF GREEN и RED

<details>
  <summary>CUSTOMER#show ip bgp neighbors 192.168.1.39 advertised-routes vrf GREEN </summary>
```
CUSTOMER#show ip bgp neighbors 192.168.1.39 advertised-routes vrf GREEN
BGP routing table information for VRF GREEN
Router identifier 192.168.3.200, local AS number 65500
Route status codes: s - suppressed contributor, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast, q - Queued for advertisement
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI Origin Validation codes: V - valid, I - invalid, U - unknown
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >      192.168.1.0/24         192.168.1.200         -       -          -       -       65500 i
 * >      192.168.2.0/24         192.168.1.200         -       -          -       -       65500 i
 * >      192.168.3.0/24         192.168.1.200         -       -          -       -       65500 i
 * >      192.168.4.0/24         192.168.1.200         -       -          -       -       65500 i
```
</details>

<details>
  <summary>CUSTOMER#show ip bgp neighbors 192.168.2.39 advertised-routes vrf RED </summary>
```
CUSTOMER#show ip bgp neighbors 192.168.2.39 advertised-routes vrf RED
BGP routing table information for VRF RED
Router identifier 192.168.4.200, local AS number 65500
Route status codes: s - suppressed contributor, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast, q - Queued for advertisement
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI Origin Validation codes: V - valid, I - invalid, U - unknown
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >      192.168.1.0/24         192.168.2.200         -       -          -       -       65500 i
 * >      192.168.2.0/24         192.168.2.200         -       -          -       -       65500 i
 * >      192.168.3.0/24         192.168.2.200         -       -          -       -       65500 i
 * >      192.168.4.0/24         192.168.2.200         -       -          -       -       65500 i
```
</details>


<details>
  <summary>CUSTOMER#show ip bgp neighbors 192.168.1.40 advertised-routes vrf GREEN </summary>
```
CUSTOMER#show ip bgp neighbors 192.168.1.40 advertised-routes vrf GREEN
BGP routing table information for VRF GREEN
Router identifier 192.168.3.200, local AS number 65500
Route status codes: s - suppressed contributor, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast, q - Queued for advertisement
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI Origin Validation codes: V - valid, I - invalid, U - unknown
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >      192.168.1.0/24         192.168.1.200         -       -          -       -       65500 i
 * >      192.168.2.0/24         192.168.1.200         -       -          -       -       65500 i
 * >      192.168.3.0/24         192.168.1.200         -       -          -       -       65500 i
 * >      192.168.4.0/24         192.168.1.200         -       -          -       -       65500 i
```
</details>

<details>
  <summary>CUSTOMER#show ip bgp neighbors 192.168.2.40 advertised-routes vrf RED </summary>
```
CUSTOMER#show ip bgp neighbors 192.168.2.40 advertised-routes vrf RED
BGP routing table information for VRF RED
Router identifier 192.168.4.200, local AS number 65500
Route status codes: s - suppressed contributor, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast, q - Queued for advertisement
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI Origin Validation codes: V - valid, I - invalid, U - unknown
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >      192.168.1.0/24         192.168.2.200         -       -          -       -       65500 i
 * >      192.168.2.0/24         192.168.2.200         -       -          -       -       65500 i
 * >      192.168.3.0/24         192.168.2.200         -       -          -       -       65500 i
 * >      192.168.4.0/24         192.168.2.200         -       -          -       -       65500 i
```
</details>


### Проверяем таблицу маршрутизации CUSTOMER в VRF GREEN и RED 

<details>
  <summary>CUSTOMER#show ip route vrf GREEN </summary>
```
CUSTOMER#show ip route vrf GREEN

VRF: GREEN
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

 C        192.168.1.0/24 is directly connected, Vlan301
 B L      192.168.2.0/24 is directly connected (source VRF RED), Vlan302 (egress VRF RED)
 C        192.168.3.0/24 is directly connected, Vlan401
 B L      192.168.4.0/24 is directly connected (source VRF RED), Vlan402 (egress VRF RED)
```
</details>

<details>
  <summary>CUSTOMER#show ip route vrf RED </summary>
```
CUSTOMER#show ip route vrf RED

VRF: RED
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

 B L      192.168.1.0/24 is directly connected (source VRF GREEN), Vlan301 (egress VRF GREEN)
 C        192.168.2.0/24 is directly connected, Vlan302
 B L      192.168.3.0/24 is directly connected (source VRF GREEN), Vlan401 (egress VRF GREEN)
 C        192.168.4.0/24 is directly connected, Vlan402
```
</details>


### Проверяем таблицу маршрутизации LEAF04  в VRF GREEN и RED

<details>
  <summary>dc01-pod01-leaf04#show ip route vrf GREEN </summary>
```
dc01-pod01-leaf04#show ip route vrf GREEN

VRF: GREEN
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

 C        192.168.1.0/24 is directly connected, Vlan301
 B E      192.168.2.0/24 [200/0] via 192.168.1.200, Vlan301
 B E      192.168.3.0/24 [200/0] via 192.168.1.200, Vlan301
 B E      192.168.4.0/24 [200/0] via 192.168.1.200, Vlan301
```
</details>

<details>
  <summary>dc01-pod01-leaf04#show ip route vrf RED </summary>
```
dc01-pod01-leaf04#show ip route vrf RED

VRF: RED
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

 B E      192.168.1.0/24 [200/0] via 192.168.2.200, Vlan302
 C        192.168.2.0/24 is directly connected, Vlan302
 B E      192.168.3.0/24 [200/0] via 192.168.2.200, Vlan302
 B E      192.168.4.0/24 [200/0] via 192.168.2.200, Vlan302
```
</details>


### Смотрим как выглядит Type-5 маршрут на LEAF04 в сторону сети 192.168.4.0 в VRF RED на CUSTOMER.  Видим, что маршруты на эту сеть мы видим в обоих VRF (GREEN и RED)

<details>
  <summary>dc01-pod01-leaf04#show bgp evpn route-type ip-prefix 192.168.4.0 </summary>
``` 
dc01-pod01-leaf04#show bgp evpn route-type ip-prefix 192.168.4.0
BGP routing table information for VRF default
Router identifier 10.11.1.6, local AS number 4259905003
BGP routing table entry for ip-prefix 192.168.4.0/24, Route Distinguisher: 1000888:888
 Paths: 2 available
  65500
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:301:1000888 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:72:8b:31
      VNI: 1000888
  4259840002 4259905002 65500
    10.11.1.5 from 10.11.1.2 (10.11.1.2)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external
      Extended Community: Route-Target-AS:301:1000888 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:15:f4:e8
      VNI: 1000888
BGP routing table entry for ip-prefix 192.168.4.0/24, Route Distinguisher: 1000999:999
 Paths: 2 available
  65500
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, best
      Extended Community: Route-Target-AS:302:1000999 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:72:8b:31
      VNI: 1000999
  4259840002 4259905002 65500
    10.11.1.5 from 10.11.1.2 (10.11.1.2)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external
      Extended Community: Route-Target-AS:302:1000999 TunnelEncap:tunnelTypeVxlan EvpnRouterMac:50:00:00:15:f4:e8
      VNI: 1000999
dc01-pod01-leaf04#
```
</details>


### Проверяем Доступность интерфейсов, находящихся  в разных VRF на LEAF04

<details>
  <summary>dc01-pod01-leaf04#ping vrf GREEN 192.168.2.200 source 192.168.1.40 </summary>
``` 
dc01-pod01-leaf04#ping vrf GREEN 192.168.2.200 source 192.168.1.40

PING 192.168.2.200 (192.168.2.200) from 192.168.1.40 : 72(100) bytes of data.
80 bytes from 192.168.2.200: icmp_seq=1 ttl=64 time=185 ms
80 bytes from 192.168.2.200: icmp_seq=2 ttl=64 time=214 ms
80 bytes from 192.168.2.200: icmp_seq=3 ttl=64 time=303 ms
80 bytes from 192.168.2.200: icmp_seq=4 ttl=64 time=352 ms
80 bytes from 192.168.2.200: icmp_seq=5 ttl=64 time=513 ms

--- 192.168.2.200 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 50ms
rtt min/avg/max/mdev = 185.253/313.652/513.107/116.399 ms, pipe 5, ipg/ewma 12.511/258.237 ms
```
</details>

<details>
  <summary>dc01-pod01-leaf04#ping vrf GREEN 192.168.4.200 source 192.168.1.40 </summary>
```
dc01-pod01-leaf04#ping vrf GREEN 192.168.4.200 source 192.168.1.40

PING 192.168.4.200 (192.168.4.200) from 192.168.1.40 : 72(100) bytes of data.
80 bytes from 192.168.4.200: icmp_seq=1 ttl=64 time=172 ms
80 bytes from 192.168.4.200: icmp_seq=2 ttl=64 time=182 ms
80 bytes from 192.168.4.200: icmp_seq=3 ttl=64 time=204 ms
80 bytes from 192.168.4.200: icmp_seq=4 ttl=64 time=220 ms
80 bytes from 192.168.4.200: icmp_seq=5 ttl=64 time=245 ms

--- 192.168.4.200 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 172.666/205.146/245.556/26.222 ms, pipe 5, ipg/ewma 11.600/190.881 ms
```
</details>


### ВЫВОДЫ: внешние type-5 маршруты анонсируются, роут-ликинг работает.

