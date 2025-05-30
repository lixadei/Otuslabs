# Лабораторная работа №1: Underlay Network на eBGP

## Цели работы
- Настроить протокол eBGP для Underlay сети
- Проверить связанность между устройствами

## Выполнение работы

### 1. Топология сети
![Схема сети с выделенными AS](https://github.com/lixadei/Otuslabs/blob/main/lab4/ebgp-as-topo.png)
*Рисунок 1. Логическая схема сети с обозначением AS*

### 2. Конфигурация оборудования
Полные конфигурации оборудования доступны в приложенных файлах

### 3. Проверка работоспособности

#### Проверка установления BGP-соседства и таблицы маршрутизации
Проверка со стороны spine1
![image](https://github.com/user-attachments/assets/de7408f9-33a7-45aa-8b40-40fa0d433f4a)
Проверка со стороны spine2
![image](https://github.com/user-attachments/assets/2d6426aa-a3a8-49d0-b904-3d38a8f32fcb)
Проверка со стороны leaf1
![image](https://github.com/user-attachments/assets/7c18b506-0a0f-4e0f-8b75-dc99b302dfef)
Пинги до lo spine1 и spine2 со стороны leaf1
![image](https://github.com/user-attachments/assets/068ad964-d01a-4a36-bf32-a7b3981febf8)
Пинги до lo leaf2 и leaf3 со стороны leaf1
![image](https://github.com/user-attachments/assets/21c1eea8-36f8-49bf-94a3-e536dae5da33)
## Выводы
- Успешно настроен Underlay на базе eBGP
