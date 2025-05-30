# Лабораторная работа №2: Underlay Network на OSPF

## Цели работы
- Настроить Underlay-сеть с использованием протокола OSPF
- Обеспечить базовую IP-связность между всеми узлами сети
- Проверить корректность работы протокола маршрутизации

## Выполнение работы

### 1. Топология сети
![Схема сети с выделенными OSPF Area](https://github.com/lixadei/Otuslabs/blob/main/lab2/area.png)
*Рисунок 1. Логическая схема сети с обозначением OSPF Area*

### 2. Конфигурация оборудования
Полные конфигурации оборудования доступны в приложенных файлах

### 3. Проверка работоспособности

#### Проверка установления OSPF-соседства и таблицы маршрутизации
![Состояние OSPF-соседей](https://github.com/user-attachments/assets/0bdb3af2-1b36-4cdd-aa2a-8555123b4cb2)

![Вывод таблицы маршрутизации OSPF](https://github.com/user-attachments/assets/de1aca8f-d42d-4a4a-ab38-b826947bb95d)

## Выводы
- Успешно настроен Underlay на базе OSPF
- Подтверждена корректная установка соседских отношений
- Проверена полная связность между всеми узлами сети
