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
[SPINE01.cfg](SPINE01.cfg)      [BLEAF01.cfg](BLEAF01.cfg)   [SLEAF01.cfg](SLEAF01.cfg)  [BGW01.cfg](BGW01.cfg)
[SPINE02.cfg](SPINE02.cfg)      [BLEAF02.cfg](BLEAF02.cfg)   [SLEAF02.cfg](SLEAF02.cfg)  [BGW02.cfg](BGW02.cfg)
[LEAF01.cfg](LEAF01.cfg)
[LEAF02.cfg](LEAF02.cfg)
[LEAF03.cfg](LEAF03.cfg)
[LEAF04.cfg](LEAF04.cfg)


## Подтверждение работоспособности Топологии:

### show ip ospf neighbors


### show ip ospf 


### show bfd peers
