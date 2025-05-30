# Лабораторная работа №1: Underlay Network на eBGP

## Цели работы
- Настроить протокол eBGP для Underlay сети
- Проверить связанность между устройствами
- Обеспечить маршрутизацию между всеми узлами

## Выполнение работы

### 1. Топология сети
![Логическая схема сети с обозначением AS](https://github.com/lixadei/Otuslabs/blob/main/lab4/ebgp-as-topo.png)  
*Рисунок 1. Архитектура Underlay сети с выделенными Autonomous Systems*

### 2. Конфигурация оборудования
Полные конфигурации оборудования доступны в приложенных файлах

### 3. Проверка работоспособности

#### Проверка BGP-соседства

**Spine1**  
*Проверка состояния BGP сессий на Spine1 (AS 65000):*
![Статус BGP сессий на Spine1](https://github.com/user-attachments/assets/de7408f9-33a7-45aa-8b40-40fa0d433f4a)

**Spine2**  
*Проверка состояния BGP сессий на Spine2 (AS 65000):*
![Статус BGP сессий на Spine2](https://github.com/user-attachments/assets/2d6426aa-a3a8-49d0-b904-3d38a8f32fcb)

**Leaf1**  
*Анализ таблицы маршрутизации на Leaf1 (AS 65001):*
![Таблица маршрутизации Leaf1](https://github.com/user-attachments/assets/7c18b506-0a0f-4e0f-8b75-dc99b302dfef)

#### Проверка связности

**Тестирование до Spine-узлов**  
*Результаты ping-тестов с Leaf1 на Spine-устройства:*
![Пинги до Spine1 и Spine2](https://github.com/user-attachments/assets/068ad964-d01a-4a36-bf32-a7b3981febf8)

**Тестирование до других Leaf-узлов**  
*Результаты ping-тестов между Leaf-устройствами:*
![Пинги до Leaf2 и Leaf3](https://github.com/user-attachments/assets/21c1eea8-36f8-49bf-94a3-e536dae5da33)

### Выводы
   - Все BGP сессии установлены (State = Established)
   - Достигнута 100% связность между узлами
   - Маршруты Loopback-интерфейсов корректно анонсируются
   - Сеть готова к развертыванию overlay-решений (VXLAN/EVPN)
