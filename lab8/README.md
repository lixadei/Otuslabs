# Лабораторная работа №8: VXLAN. Multihoming.

## Цели работы
- Реализовать передачу суммарных префиксов через EVPN route-type 5
- Анонсировать суммарные префиксы клиентов в Overlay сети
- Настроить маршрутизацию между клиентами через суммарный префикс
- Проверить связанность между клиентами

## Выполнение работы

### 1. Топология сети
![Логическая схема сети с обозначением AS](https://github.com/lixadei/Otuslabs/blob/main/lab6/VxLAN%20EVPN%20L3-topo.png)

*Рисунок 1. Архитектура Underlay сети (AS 65000-65003) с выделенными:*
- **Spine-устройствами** (AS 65000) - обеспечивают транспортную сеть между Leaf-ами
- **Leaf-устройствами** (AS 65001-65003) - выполняют функции VTEP (VXLAN Tunnel End Point)
- **Клиенты PC** - подключены к различным Leaf в разных VLAN

### 2. Конфигурация оборудования
Полные конфигурации оборудования доступны в приложенных файлах (конфигурация spine без изменений)

### 3. Анализ работоспособности BGP EVPN VXLAN инфраструктуры

#### 3.1. Проверка на Leaf1 (AS 65001)

##### Проверка таблицы маршрутизации VRF
![leaf1#show ip route vrf OTUS-Vxlan](https://github.com/user-attachments/assets/9023c543-9571-4624-8a98-7285cb49bcc8)
- Вывод таблицы маршрутизации VRF OTUS-Vxlan
- Наличие маршрутов ко всем клиентским подсетям через VXLAN

##### Проверка MAC/IP маршрутов EVPN
![leaf1#show bgp evpn route-type mac-ip](https://github.com/user-attachments/assets/5a0123df-b764-4bd8-b7a9-9f89c993a096)
- Вывод MAC/IP маршрутов EVPN
- Убеждаемся в получении маршрутов от других Leaf-ов

##### Проверка ARP-таблицы
![leaf1#show arp vrf OTUS-Vxlan](https://github.com/user-attachments/assets/c046f03c-5b5b-4161-bcfe-b341768df645)
- Просмотр ARP-таблицы в VRF

##### Проверка состояния VXLAN интерфейса
![leaf1#show interfaces vxlan 1](https://github.com/user-attachments/assets/4e4b52a8-4cad-474e-885c-4a721f8e782a)
- Состояние VXLAN интерфейса
- Проверка корректной работы VXLAN туннеля

##### Проверка настроенных VNI
![leaf1#show vxlan vni](https://github.com/user-attachments/assets/44cf3f26-7d07-4211-a6ac-463301ca98f9)
- Информация о настроенных VNI
- Проверка, что L3 VNI 112 ассоциирован с VRF OTUS-Vxlan

#### 3.2. Проверка на Leaf2 (AS 65002)

##### Проверка таблицы маршрутизации VRF
![leaf2#show ip route vrf OTUS-Vxlan](https://github.com/user-attachments/assets/31d3dfd6-4ede-4006-a234-5d594088035a)
- Вывод таблицы маршрутизации VRF OTUS-Vxlan
- Наличие маршрутов ко всем клиентским подсетям через VXLAN

#### 3.3. Проверка на Leaf3 (AS 65003)

##### Проверка таблицы маршрутизации VRF
![leaf3#show ip route vrf OTUS-Vxlan](https://github.com/user-attachments/assets/57b0d1c5-6ec4-4cf7-8d3b-07fc3312a8e6)
- Вывод таблицы маршрутизации VRF OTUS-Vxlan
- Наличие маршрутов ко всем клиентским подсетям через VXLAN

### 4. Проверка связности

#### Распределение клиентов по VLAN:
- VLAN10 - leaf1  
- VLAN11 - leaf2  
- VLAN12 - leaf3  

#### Проверка связности между клиентами

##### Ping-тесты с PC1
![Ping результаты с pc1](https://github.com/user-attachments/assets/31907c8d-4abd-4409-a1ef-493323d9734c)

| Источник | Назначение | Результат  |
|----------|------------|------------|
| PC1      | PC2        | Успешно    |
| PC1      | PC3        | Успешно    |
| PC1      | PC4        | Успешно    |

##### Ping-тесты с PC3
![Ping результаты с pc3](https://github.com/user-attachments/assets/f4405754-3695-431d-9081-5d7502d1235b)

| Источник | Назначение | Результат  |
|----------|------------|------------|
| PC3      | PC2        | Успешно    | 
| PC3      | PC4        | Успешно    |

**Вывод:**  
- L3 связанность через VXLAN работает корректно на всей тестовой инфраструктуре
- Все тесты проходят без потерь пакетов
- Маршрутизация между клиентами в разных VLAN и на разных Leaf-ах осуществляется через VXLAN Overlay сеть
