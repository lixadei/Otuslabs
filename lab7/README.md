# Лабораторная работа №7: VXLAN. Multihoming

## Цели работы
- Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming
- Подключить клиентов двумя линками к различным Leaf-устройствам
- Настроить агрегированный канал со стороны клиента
- Настроить multihoming для работы в Overlay-сети
- Убедиться, что связность не теряется при отключении одного из линков

## Выполнение работы

### 1. Топология сети
![Логическая схема сети с обозначением AS](https://github.com/lixadei/Otuslabs/blob/main/lab7/VXLAN.%20Multihoming-topo.png)

*Рисунок 1. Архитектура Underlay сети (AS 65000-65003):*
- **Spine-устройства** (AS 65000) - обеспечивают транспортную сеть между Leaf-ами
- **Leaf-устройства**:
  - AS 65001 (Leaf1, Leaf2) - выполняют функции VTEP (VXLAN Tunnel End Point)
  - AS 65003 (Leaf3) - выполняет функции VTEP
- **Клиенты PC** - подключены к одному Leaf в разных VLAN
- **Сервер srv-1** - подключен к двум Leaf в разных VLAN

### 2. Конфигурация оборудования
Полные конфигурации оборудования доступны в приложенных файлах (конфигурация spine без изменений)

### 3. Проверка доступности

#### 3.1. Проверка на Leaf1 (AS 65001)

**Проверка состояния MLAG:**

![leaf1#show mlag](https://github.com/user-attachments/assets/c1ab92b4-23aa-4971-af06-b969d13ebba4)

- Состояние MLAG: `Active`
- Peer-линк: `Up`

**Проверка интерфейсов MLAG:**

![leaf1#show mlag interfaces det](https://github.com/user-attachments/assets/c4463a54-2298-4b1f-baa9-0ae0e8c10668)

- Интерфейсы MLAG: `active-full`

**Проверка port-channel:**

![leaf1#show port-channel 1 detailed](https://github.com/user-attachments/assets/cd438ec4-1f3a-4f13-85a7-8432ee2815b2)

- Состояние канала: `Active`
- Участники: `Eth7`, `PeerEthernet7`

**Проверка BGP EVPN:**

![leaf1#show bgp evpn](https://github.com/user-attachments/assets/d77b2687-3357-4b52-8712-c1b480dd69f4)

- Устройство обменивается EVPN маршрутами с VTEP `10.200.3.0`
- Полученные маршруты: `MAC-IP` и `IP-префиксы`
- Локальные маршруты: присутствуют

**Проверка таблицы маршрутизации VRF:**

![leaf1#show ip route vrf OTUS-Vxlan](https://github.com/user-attachments/assets/b286cd1d-9931-4fc4-8a24-9eb039073997)

- Вывод таблицы маршрутизации VRF OTUS-Vxlan
- Наличие маршрутов ко всем клиентским подсетям через VXLAN

**Проверка таблицы VXLAN адресов:**

![leaf1#show vxlan address-table](https://github.com/user-attachments/assets/6e741893-e055-4c2e-a787-32421cc232e7)

- Записи MAC-адресов: присутствуют
- Соответствие VNI: корректное

**Проверка интерфейса VXLAN:**

![show interfaces vxlan 1](https://github.com/user-attachments/assets/ce35c303-beb6-4875-a940-3e3763f48412)

- Состояние VXLAN интерфейса
- Проверка корректной работы VXLAN туннеля

**Проверка VNI:**

![leaf1#show vxlan vni](https://github.com/user-attachments/assets/a9eb5129-01d1-414a-a51b-3d00a0608d17)

- Информация о настроенных VNI
- Проверка, что L3 VNI 112 ассоциирован с VRF OTUS-Vxlan

#### 3.2. Проверка на Leaf2 (AS 65001)

**Проверка таблицы маршрутизации VRF:**

![leaf2#show ip route vrf OTUS-Vxlan](https://github.com/user-attachments/assets/1fc0db1a-37c9-4c7e-bd19-e704ad742ca2)

- Вывод таблицы маршрутизации VRF OTUS-Vxlan
- Наличие маршрутов ко всем клиентским подсетям через VXLAN

#### 3.3. Проверка на Leaf3 (AS 65003)

**Проверка таблицы маршрутизации VRF:**

![leaf3#show ip route vrf OTUS-Vxlan](https://github.com/user-attachments/assets/931d2189-98be-4430-b9d6-e6ff7596cb70)

- Вывод таблицы маршрутизации VRF OTUS-Vxlan
- Наличие маршрутов ко всем клиентским подсетям через VXLAN

### 4. Проверка связности

#### Ping-тесты между устройствами

**Результаты с srv1:**

![Ping результаты с srv1](https://github.com/user-attachments/assets/d1a27b9b-028e-47ea-8f83-7f8a2ea5c10d)

| Источник | Назначение | Результат  |
|----------|------------|------------|
| srv1     | PC3        | Успешно    |
| srv1     | PC4        | Успешно    |

**Результаты с PC3:**

![Ping результаты с pc3](https://github.com/user-attachments/assets/a554aff3-fd42-4ac2-ab3c-425f78f0cf19)

| Источник | Назначение | Результат  |
|----------|------------|------------|
| PC3      | srv1       | Успешно    | 
| PC3      | srv1       | Успешно    |

### 5. Проверка отказоустойчивости

**Тестирование при отключении линка:**

![Отключен линк eth7](https://github.com/user-attachments/assets/86228a2c-c243-4c7c-afd2-41a46a106acd)

**Результаты с PC3:**

![Ping результаты с pc4](https://github.com/user-attachments/assets/656d1051-c453-4674-86db-0ed8f3885e57)

- Отключен линк на leaf2 в сторону srv-1
- Ping с pc4: успешно (без потерь)

## Выводы
- Настроено отказоустойчивое подключение с использованием EVPN Multihoming
- Агрегированные каналы работают корректно
- При отключении одного из линков связность не теряется
- Все тесты проходят без потерь пакетов
