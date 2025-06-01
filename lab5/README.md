# Лабораторная работа №5: VxLAN. L2 VNI

## Цели работы
- Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами
- Настроить связанность между клиентами.
- Проверить корректность распространения MAC-адресов через BGP EVPN

## Выполнение работы

### 1. Топология сети
![Логическая схема сети с обозначением AS](https://github.com/lixadei/Otuslabs/blob/main/lab5/VxLAN.%20L2-VNI-topo.png) 
*Рисунок 1. Архитектура Underlay сети (AS 65000-65003) с выделенными:*
- **Spine-устройствами** (AS 65000)
- **Leaf-устройствами** (AS 65001-65003)
- **Клиентами pc**

### 2. Конфигурация оборудования
Полные конфигурации оборудования доступны в приложенных файлах

### 3. Анализ работоспособности BGP EVPN VXLAN инфраструктуры

#### 3.1. Анализ BGP EVPN соседств

**Spine1 (AS 65000)**  
![Статус BGP EVPN сессий на Spine1](https://github.com/user-attachments/assets/5642a6ba-b609-4794-b50d-2adff0a624ce)  
- **Сессии:** Установлены с Leaf1, Leaf2, Leaf3  
- **Статус:** Все `Established` (Up)  

**Spine2 (AS 65000)**  
![Статус BGP EVPN сессий на Spine2](https://github.com/user-attachments/assets/64525118-84fa-4eb4-8759-5059d3c63a01)  
- **Сессии:** Установлены с Leaf1, Leaf2, Leaf3  
- **Статус:** Все `Established` (Up)  

**Leaf1 (AS 65001)**  
![Статус BGP сессий Leaf1](https://github.com/user-attachments/assets/7c41c8e5-83d6-4937-bfc2-6551df31f244)  
- Активные BGP сессии с обоими Spine
- Активные BGP EVPN сессии с обоими Leaf

#### 3.2. Проверка VXLAN компонентов

**Таблица MAC-адресов (Leaf1):**  
![VXLAN address-table](https://github.com/user-attachments/assets/b3e64743-7371-46ac-915d-e01be3504bde)  
  - MAC клиента PC2: 0050.7966.6807 → VTEP 10.200.2.0 
  - MAC клиента PC3: 0050.7966.6808 → VTEP 10.200.2.0   
  - MAC клиента PC4: 0050.7966.6809 → VTEP 10.200.3.0  

**Список VTEP (Leaf1):**  
![VXLAN VTEP](https://github.com/user-attachments/assets/ee41da39-5111-4b06-9cdf-a2df070cb414)  
- **Удаленные VTEP:**  
  - 10.200.2.0 (Leaf2)  
  - 10.200.3.0 (Leaf3)

**Таблица EVPN маршрутов (Leaf1):**  
![BGP EVPN MAC-IP](https://github.com/user-attachments/assets/06f3c4b0-c5ad-4df2-b7ec-49cb2bf47649)  
- **Маршруты типа MAC/IP:**  
  - Все клиентские MAC-адреса присутствуют 

#### 3.3. Проверка связности

**Результаты ping-тестов:**  
![Ping результаты](https://github.com/user-attachments/assets/573a8e9f-cd4b-488d-b74e-7a18090fb13f)  
| Тест       | Источник | Назначение | Результат  |
|------------|----------|------------|------------| 
| VLAN10-L2  | PC1      | PC2        | Успешно    | 
| VLAN10-L2  | PC1      | PC3        | Успешно    |
| VLAN10-L2  | PC1      | PC4        | Успешно    |

**Вывод:**  
- L2 связанность через VXLAN работает корректно  
- Все тесты проходят без потерь пакетов  
