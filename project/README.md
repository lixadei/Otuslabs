# Проектирование сети ЦОД на основе EVPN-VXLAN с использованием архитектуры Spine-Leaf

## Цели работы
- Создание масштабируемой, отказоустойчивой и гибкой сети в рамках ЦОД
- Поддержка единых L2/L3 доменов
- Обеспечение высокой пропускной способности для трафика "восток-запад"

## 1. Введение
Современные центры обработки данных (ЦОД) требуют высокопроизводительных, масштабируемых и отказоустойчивых сетевых решений. Традиционные подходы (например, трёхуровневая архитектура Core-Distribution-Access) не справляются с ростом трафика "восток-запад" (East-West), характерного для облачных и виртуализированных сред.

В данном проекте рассматривается проектирование сети ЦОД на основе:
- Архитектуры Spine-Leaf (CLOS) для обеспечения масштабируемости и отсутствия переподписки
- Технологии EVPN-VXLAN для гибкой поддержки L2/L3-доменов
- Протокола BGP (eBGP) в underlay и overlay сетях

## 2. Архитектура сети

### 2.1. Spine-Leaf топология
![Логическая схема сети с обозначением AS](https://github.com/lixadei/Otuslabs/blob/main/project/project-topo.ebgp-evpn-vxlan-clos.png)

*Рисунок 1. Архитектура Underlay сети (AS 65000-65003) с выделенными:*
- **Spine-устройствами** (AS 65000)
- **Leaf-устройствами** (AS 65001-65003)
- **Клиенты PC и SRV**
- **Граничный роутер (borderrouter)** 

Сеть строится по принципу CLOS, где:
- Leaf-коммутаторы – обеспечивают подключение серверов, виртуальных машин и сетевых сервисов
- Spine-коммутаторы – формируют высокоскоростное ядро, обеспечивая связность между Leaf

**Преимущества архитектуры:**
- Отсутствие переподписки (non-blocking fabric)
- Горизонтальная масштабируемость (добавление Spine/Leaf без изменения топологии)
- Высокая отказоустойчивость (ECMP, MLAG)

### 2.2. Underlay-сеть (физическая инфраструктура)
Для маршрутизации в underlay используется eBGP (между Spine и Leaf), так как он:
- Обеспечивает простоту настройки и масштабируемость
- Поддерживает ECMP для балансировки нагрузки
- Позволяет гибко управлять политиками маршрутизации

**Адресное пространство underlay:**
- Loopback-интерфейсы (для BGP) – /32
- P2P-линки между Spine и Leaf – /31

![ipplan](https://github.com/lixadei/Otuslabs/blob/main/project/ipplan.PNG)

*Таблица 1. План IP-адресации Underlay сети*

### 2.3. Overlay-сеть (логическая инфраструктура)
Для overlay применяется EVPN-VXLAN, который решает ключевые задачи:
- Гибкость: поддержка L2 (VLAN) и L3 (VRF) доменов
- Масштабируемость: использование VNI (VXLAN Network Identifier) для изоляции трафика

**Компоненты EVPN:**
- Type-2 (MAC/IP маршруты) – для L2-связности
- Type-5 (IP Prefix маршруты) – для L3-маршрутизации между VRF
- Type-3 (Inclusive Multicast) – для установки VXLAN туннелей

### 2.4. Подключение серверов
- **ESI-LAG (Ethernet Segment Identifier)** – обеспечивает отказоустойчивость при подключении сервера к двум Leaf
- **VLAN ↔ VNI маппинг** – для поддержки L2-доменов поверх VXLAN

## Реализация

### 3. Конфигурация оборудования
Полные конфигурации оборудования доступны в приложенных файлах.

### 4. Вывод диагностической информации

#### 4.1. Проверка BGP-сессий на Spine1|Spine2 (AS 65000)
```
spine1#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.1.1.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.0 4 65001           4227      4082    0    0 02:48:01 Estab   2      2
  10.2.2.0 4 65002           4208      4107    0    0 02:48:04 Estab   2      2
  10.2.3.0 4 65003           4183      4242    0    0 02:48:06 Estab   2      2
  10.4.1.1 4 65001           1944      1951    0    0 01:23:00 Estab   2      2
  10.4.1.3 4 65002           1961      1959    0    0 01:23:06 Estab   2      2
  10.4.1.5 4 65003           3934      3939    0    0 02:47:55 Estab   2      2

spine2#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.1.2.0, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.0 4 65001           4299      4180    0    0 02:52:00 Estab   2      2
  10.2.2.0 4 65002           4297      4211    0    0 02:52:02 Estab   2      2
  10.2.3.0 4 65003           4274      4343    0    0 02:52:06 Estab   2      2
  10.4.2.1 4 65001           2043      2053    0    0 01:26:59 Estab   2      2
  10.4.2.3 4 65002           2044      2047    0    0 01:27:05 Estab   2      2
  10.4.2.5 4 65003           4034      4043    0    0 02:51:55 Estab   2      2
```
- Все BGP-сессии между spine-устройствами и leaf-устройствами находятся в состоянии Established
- Время работы сессий (Up/Down) показывает стабильную работу без сбоев

#### 4.2. Проверка BGP-сессий на Leaf1 (AS 65001)
```
leaf1#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.2.1.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0 4 65000           4213      4360    0    0 02:53:40 Estab   5      5
  10.1.2.0 4 65000           4219      4338    0    0 02:53:41 Estab   5      5
  10.4.1.0 4 65000           4095      4073    0    0 01:28:39 Estab   5      5
  10.4.2.0 4 65000           4090      4090    0    0 01:28:39 Estab   5      5
leaf1#show ip bgp summary vrf OTUS-Vxlan
BGP summary information for VRF OTUS-Vxlan
Router identifier 172.16.11.252, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor    V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  11.11.11.11 4 65100           3215      3852    0    0 01:28:42 Estab   4      4
leaf1#show ip bgp summary vrf OTUS2VRF
BGP summary information for VRF OTUS2VRF
Router identifier 77.77.77.100, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor    V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  77.77.77.77 4 65100           3604      4090    0    0 01:28:54 Estab   4      4
```
- Успешные BGP-сессии со всеми spine-устройствами
- Сессии в сторону Пограничного маршрутизатора в VRF OTUS-Vxlan и OTUS2VRF также работают стабильно


#### 4.3. Проверка BGP-сессий на Leaf2 (AS 65002)
```
leaf2#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.2.2.0, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0 4 65000           4298      4398    0    0 02:56:13 Estab   5      5
  10.1.2.0 4 65000           4309      4396    0    0 02:56:13 Estab   5      5
  10.4.1.2 4 65000           4148      4142    0    0 01:31:16 Estab   5      5
  10.4.2.2 4 65000           4142      4144    0    0 01:31:15 Estab   5      5
```
- Успешные BGP-сессии со всеми spine-устройствами

#### 4.4. Проверка BGP-сессий на Leaf3 (AS 65003)
```
leaf3#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.2.3.0, local AS number 65003
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.0 4 65000           4492      4431    0    0 02:58:46 Estab   5      5
  10.1.2.0 4 65000           4501      4431    0    0 02:58:47 Estab   5      5
  10.4.1.4 4 65000           4195      4194    0    0 02:58:35 Estab   5      5
  10.4.2.4 4 65000           4206      4198    0    0 02:58:35 Estab   5      5
```
- Успешные BGP-сессии со всеми spine-устройствами

#### 4.5. Проверка BGP-сессий на borderrouter (AS 65100)
```
borderrouter#show ip bgp summary
BGP summary information for VRF default
Router identifier 8.8.8.8, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  11.11.11.200     4  65001           3988      3339    0    0 01:34:41 Estab   3      3
  77.77.77.100     4  65001           4225      3722    0    0 01:34:42 Estab   1      1
```
- Две активные BGP-сессии с Leaf1 (AS 65001)

#### 4.6. Проверка leaf1#show bgp evpn (AS 65001)
```leaf1#show bgp evpn
leaf1#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.2.1.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65003:10010 mac-ip 0050.7966.6808
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 mac-ip 0050.7966.6808
                                 10.200.3.0            -       100     0       65000 65003 i
 * >Ec    RD: 65003:10010 mac-ip 0050.7966.6808 172.16.10.3
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 mac-ip 0050.7966.6808 172.16.10.3
                                 10.200.3.0            -       100     0       65000 65003 i
 * >Ec    RD: 65003:10011 mac-ip 0050.7966.6809
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 mac-ip 0050.7966.6809
                                 10.200.3.0            -       100     0       65000 65003 i
 * >Ec    RD: 65003:10011 mac-ip 0050.7966.6809 172.16.11.3
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 mac-ip 0050.7966.6809 172.16.11.3
                                 10.200.3.0            -       100     0       65000 65003 i
 * >      RD: 65001:10011 mac-ip 0050.7966.680c
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10011 mac-ip 0050.7966.680c
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 0050.7966.680c
                                 10.200.2.0            -       100     0       65000 65002 i
 * >      RD: 65001:100777 mac-ip 0050.7966.680d
                                 -                     -       -       0       i
 * >Ec    RD: 65002:100777 mac-ip 0050.7966.680d
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:100777 mac-ip 0050.7966.680d
                                 10.200.2.0            -       100     0       65000 65002 i
 * >      RD: 65001:10010 mac-ip 5000.0072.8b31
                                 -                     -       -       0       i
 * >      RD: 65001:10011 mac-ip 5000.0072.8b31
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10010 mac-ip 5000.0072.8b31
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 mac-ip 5000.0072.8b31
                                 10.200.2.0            -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 mac-ip 5000.0072.8b31
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 5000.0072.8b31
                                 10.200.2.0            -       100     0       65000 65002 i
 * >Ec    RD: 65002:10010 mac-ip 5000.0072.8b31 172.16.10.1
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 mac-ip 5000.0072.8b31 172.16.10.1
                                 10.200.2.0            -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 mac-ip 5000.0072.8b31 172.16.11.2
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 5000.0072.8b31 172.16.11.2
                                 10.200.2.0            -       100     0       65000 65002 i
 * >      RD: 65001:100777 mac-ip 5000.0088.fe27
                                 -                     -       -       0       i
 * >Ec    RD: 65002:100777 mac-ip 5000.0088.fe27
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:100777 mac-ip 5000.0088.fe27
                                 10.200.2.0            -       100     0       65000 65002 i
 * >      RD: 65001:100777 mac-ip 5000.0088.fe27 77.77.77.77
                                 -                     -       -       0       i
 * >      RD: 65001:10010 imet 10.200.1.0
                                 -                     -       -       0       i
 * >      RD: 65001:10011 imet 10.200.1.0
                                 -                     -       -       0       i
 * >      RD: 65001:100777 imet 10.200.1.0
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10010 imet 10.200.2.0
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 imet 10.200.2.0
                                 10.200.2.0            -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 imet 10.200.2.0
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 imet 10.200.2.0
                                 10.200.2.0            -       100     0       65000 65002 i
 * >Ec    RD: 65002:100777 imet 10.200.2.0
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:100777 imet 10.200.2.0
                                 10.200.2.0            -       100     0       65000 65002 i
 * >Ec    RD: 65003:10010 imet 10.200.3.0
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 imet 10.200.3.0
                                 10.200.3.0            -       100     0       65000 65003 i
 * >Ec    RD: 65003:10011 imet 10.200.3.0
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 imet 10.200.3.0
                                 10.200.3.0            -       100     0       65000 65003 i
 * >      RD: 65001:112 ip-prefix 0.0.0.0/0
                                 -                     -       100     0       65100 ?
 * >      RD: 65001:777 ip-prefix 0.0.0.0/0
                                 -                     -       100     0       65100 ?
 * >      RD: 65001:112 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       65100 i
 * >      RD: 65001:777 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       65100 i
 * >      RD: 65001:112 ip-prefix 11.11.11.0/24
                                 -                     -       -       0       i
 *        RD: 65001:112 ip-prefix 11.11.11.0/24
                                 -                     -       100     0       65100 i
 * >      RD: 65001:777 ip-prefix 11.11.11.0/24
                                 -                     -       100     0       65100 i
 * >      RD: 65001:112 ip-prefix 77.77.77.0/24
                                 -                     -       100     0       65100 i
 * >      RD: 65001:777 ip-prefix 77.77.77.0/24
                                 -                     -       -       0       i
 *        RD: 65001:777 ip-prefix 77.77.77.0/24
                                 -                     -       100     0       65100 i
 * >      RD: 65001:112 ip-prefix 172.16.10.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65002:112 ip-prefix 172.16.10.0/24
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:112 ip-prefix 172.16.10.0/24
                                 10.200.2.0            -       100     0       65000 65002 i
 * >Ec    RD: 65003:112 ip-prefix 172.16.10.0/24
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:112 ip-prefix 172.16.10.0/24
                                 10.200.3.0            -       100     0       65000 65003 i
 * >      RD: 65001:112 ip-prefix 172.16.11.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65002:112 ip-prefix 172.16.11.0/24
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:112 ip-prefix 172.16.11.0/24
                                 10.200.2.0            -       100     0       65000 65002 i
 * >Ec    RD: 65003:112 ip-prefix 172.16.11.0/24
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:112 ip-prefix 172.16.11.0/24
                                 10.200.3.0            -       100     0       65000 65003 i

```
- В таблице EVPN присутствуют маршруты от всех соседей (Leaf2, Leaf3)
- Маршруты MAC/IP (0050.7966.6808 и другие) успешно получены и активны
- Маршруты IMET (Inclusive Multicast Ethernet Tag) для всех VNI присутствуют
- Маршруты IP Prefix (например, 172.16.10.0/24) корректно анонсируются и принимаются

#### 4.7. Проверка leaf1#show bgp evpn vni 112 (AS 65001)
```leaf1#show bgp evpn vni 112
leaf1#show bgp evpn vni 112
BGP routing table information for VRF default
Router identifier 10.2.1.0, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65003:10010 mac-ip 0050.7966.6808 172.16.10.3
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 mac-ip 0050.7966.6808 172.16.10.3
                                 10.200.3.0            -       100     0       65000 65003 i
 * >Ec    RD: 65003:10011 mac-ip 0050.7966.6809 172.16.11.3
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 mac-ip 0050.7966.6809 172.16.11.3
                                 10.200.3.0            -       100     0       65000 65003 i
 * >      RD: 65001:112 ip-prefix 0.0.0.0/0
                                 -                     -       100     0       65100 ?
 * >      RD: 65001:112 ip-prefix 8.8.8.8/32
                                 -                     -       100     0       65100 i
 * >      RD: 65001:112 ip-prefix 11.11.11.0/24
                                 -                     -       -       0       i
 *        RD: 65001:112 ip-prefix 11.11.11.0/24
                                 -                     -       100     0       65100 i
 * >      RD: 65001:112 ip-prefix 77.77.77.0/24
                                 -                     -       100     0       65100 i
 * >      RD: 65001:112 ip-prefix 172.16.10.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65002:112 ip-prefix 172.16.10.0/24
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:112 ip-prefix 172.16.10.0/24
                                 10.200.2.0            -       100     0       65000 65002 i
 * >Ec    RD: 65003:112 ip-prefix 172.16.10.0/24
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:112 ip-prefix 172.16.10.0/24
                                 10.200.3.0            -       100     0       65000 65003 i
 * >      RD: 65001:112 ip-prefix 172.16.11.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65002:112 ip-prefix 172.16.11.0/24
                                 10.200.2.0            -       100     0       65000 65002 i
 *  ec    RD: 65002:112 ip-prefix 172.16.11.0/24
                                 10.200.2.0            -       100     0       65000 65002 i
 * >Ec    RD: 65003:112 ip-prefix 172.16.11.0/24
                                 10.200.3.0            -       100     0       65000 65003 i
 *  ec    RD: 65003:112 ip-prefix 172.16.11.0/24
                                 10.200.3.0            -       100     0       65000 65003 i
```
Для VNI 112 присутствуют все ожидаемые типы маршрутов:
- MAC/IP маршруты (например, 0050.7966.6808)
- IMET маршруты
- IP Prefix маршруты (включая маршрут по умолчанию 0.0.0.0/0)
- Маршруты от Leaf2 и Leaf3 отмечены как ECMP (Equal-Cost Multi-Path)

#### 3.3. Проверка на Leaf3 (AS 65003)
**Проверка состояния MLAG:**
```
leaf1#show mlag
MLAG Configuration:
domain-id                          :             mlag100
local-interface                    :             Vlan100
peer-address                       :        10.100.100.1
peer-link                          :     Port-Channel100
peer-config                        :        inconsistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:03:37:66
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   1
```
- Состояние MLAG: Active (активный узел в паре MLAG)
- Peer-линк: Up (соединение между MLAG-узлами работает)
- Статус переговоров: Connected (успешное установление соединения)

**Проверка интерфейсов MLAG:**

leaf1#show mlag interfaces detail
```                                        local/remote
 mlag         state   local   remote    oper    config    last change   changes
------ ------------- ------- -------- ------- ---------- -------------- -------
    1   active-full     Po1      Po1   up/up   ena/ena    2:18:19 ago         5
```
- Интерфейсы MLAG: active-full (полная активность, оба узла работают)

**Проверка port-channel:**
```
leaf1#show port-channel 1
Port Channel Port-Channel1:
  Active Ports: Ethernet7 PeerEthernet7
leaf1#show port-channel 1 detailed
Port Channel Port-Channel1 (Fallback State: Unconfigured):
Minimum links: unconfigured
Minimum speed: unconfigured
Current weight/Max weight: 1/16
  Active Ports:
      Port               Time Became Active      Protocol      Mode      Weight
    ------------------ ----------------------- ------------- ----------- ------
      Ethernet7          9:18:57                 LACP          Active      1
      PeerEthernet7      9:18:57                 LACP          Active      0
```
- Состояние канала: Active (рабочее состояние)
- Участники: Eth7, PeerEthernet7 (оба порта активны)
- Используется протокол LACP в режиме Active

### 5. Проверка связности

#### Проверка связности между клиентами

##### Ping-тесты с PC4
```      
VPCS> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=63 time=482.800 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=62 time=68.323 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=62 time=45.794 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=62 time=115.250 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=62 time=45.154 ms

VPCS> ping 172.16.11.1

84 bytes from 172.16.11.1 icmp_seq=1 ttl=64 time=125.900 ms
84 bytes from 172.16.11.1 icmp_seq=2 ttl=64 time=38.501 ms
84 bytes from 172.16.11.1 icmp_seq=3 ttl=64 time=36.786 ms
84 bytes from 172.16.11.1 icmp_seq=4 ttl=64 time=99.239 ms
84 bytes from 172.16.11.1 icmp_seq=5 ttl=64 time=61.364 ms

VPCS> ping 172.16.10.1

84 bytes from 172.16.10.1 icmp_seq=1 ttl=64 time=438.972 ms
84 bytes from 172.16.10.1 icmp_seq=2 ttl=64 time=46.192 ms
84 bytes from 172.16.10.1 icmp_seq=3 ttl=64 time=40.884 ms
84 bytes from 172.16.10.1 icmp_seq=4 ttl=64 time=50.395 ms
84 bytes from 172.16.10.1 icmp_seq=5 ttl=64 time=45.836 ms

VPCS> ping 172.16.11.2

84 bytes from 172.16.11.2 icmp_seq=1 ttl=64 time=47.995 ms
84 bytes from 172.16.11.2 icmp_seq=2 ttl=64 time=41.749 ms
84 bytes from 172.16.11.2 icmp_seq=3 ttl=64 time=56.660 ms
84 bytes from 172.16.11.2 icmp_seq=4 ttl=64 time=115.920 ms
84 bytes from 172.16.11.2 icmp_seq=5 ttl=64 time=114.755 ms

VPCS> ping 77.77.77.150

84 bytes from 77.77.77.150 icmp_seq=1 ttl=59 time=487.272 ms
84 bytes from 77.77.77.150 icmp_seq=2 ttl=59 time=81.635 ms
84 bytes from 77.77.77.150 icmp_seq=3 ttl=59 time=87.911 ms
84 bytes from 77.77.77.150 icmp_seq=4 ttl=59 time=82.305 ms
84 bytes from 77.77.77.150 icmp_seq=5 ttl=59 time=84.500 ms

VPCS> ping 172.16.10.3

84 bytes from 172.16.10.3 icmp_seq=1 ttl=63 time=112.668 ms
84 bytes from 172.16.10.3 icmp_seq=2 ttl=63 time=23.201 ms
84 bytes from 172.16.10.3 icmp_seq=3 ttl=63 time=18.382 ms
84 bytes from 172.16.10.3 icmp_seq=4 ttl=63 time=23.373 ms
84 bytes from 172.16.10.3 icmp_seq=5 ttl=63 time=14.731 ms
```

| Источник | Назначение | Результат  |
|----------|------------|------------|
| PC4      | br lo1     | Успешно    |
| PC4      | PC1        | Успешно    |
| PC4      | SRV-1      | Успешно    |
| PC4      | SRV-1      | Успешно    |
| PC4      | PC2        | Успешно    |
| PC4      | PC3        | Успешно    |

## 5. Заключение

В ходе проекта успешно реализована сеть ЦОД на базе EVPN-VXLAN с архитектурой Spine-Leaf, которая обеспечивает:

✅ **Масштабируемость** – за счёт архитектуры CLOS и горизонтального масштабирования  
✅ **Гибкость** – поддержка L2/L3 сервисов через EVPN и VXLAN  
✅ **Отказоустойчивость** – реализована через:
- ESI-LAG для серверов
- MLAG для пары Leaf-коммутаторов
- ECMP в underlay сети
- Избыточные BGP сессии

Все компоненты сети продемонстрировали стабильную работу в ходе тестирования. Архитектура позволяет легко добавлять новые Spine и Leaf устройства для масштабирования сети без изменения существующей топологии.
