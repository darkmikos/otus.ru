## **OSPF. Фильтрация**

### Цель:

Цель данной работы: настроить OSPF офисе Москва, разделить сеть на зоны, настроить фильтрацию между зонами.

### Условия:

1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по-умолчанию
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101
5. Настройка для IPv6 повторяет логику IPv4
6. План работы и изменения зафиксированы в документации

### Ход выполнения:
Документация оформлена на github, использован markdown. Для выполнения лабораторной работы использовался эмулятор EVE-NG, терминал Gnome 3.38.

#### 1. Документирование адресного пространства для лабораторного стенда.

#### 1.1 Общий план адресации:

Таблица 1

| Используемые сети: | Тип  | Описание                                                     | Сети           |
| ------------------ | ---- | ------------------------------------------------------------ | -------------- |
|                    | ipv4 | используется для Point-to-Point соединений                   | 10.64.0.0/10   |
|                    | ipv4 | используется для интерфейсов Loopback's.                     | 192.168.0.0/16 |
|                    | ipv4 | используется для управления устройствами                     | 172.22.0.0/16  |
|                    | ipv4 | Пользовательские подсети                                     | 10.0.0.0/8     |
|                    | ipv6 | сеть выделенная провайдером. В подразделениях используются сети с префиксом /64. | 2001:AAAA::/48 |
|                    | ipv6 | сеть для адресов link-local                                  | FE80::/10      |

Описание сетей проекта по всем территориальным подразделениям находится в [Таблице (subnet)](https://docs.google.com/spreadsheets/d/1sV0p-q8V7yGR3Pjk1xMUrwc2siTIBcI2daprLvBshPI/edit?usp=sharing) размещенной в docs.google.com

#### 1.2 Таблица выделенных подсетей.

| Расположение | AS     | IPv4 сеть        | Родительская сеть | IPv6 сеть                  | Родительская сеть     | Описание               |
| ------------ | ------ | ---------------- | ----------------- | -------------------------- | --------------------- | ---------------------- |
| `Киторн`     | `101`  | `100.64.0.0/31`  | `100.64.0.0/23`   | `2001:AAAA:BB00:100::/64`  | `2001:AAAA:BB00::/48` | `R22e0/0 - R14e0/2`    |
|              | `101`  | `100.64.0.2/31`  | `100.64.0.0/23`   | `2001:AAAA:BB00:102::/64`  | `2001:AAAA:BB00::/48` | `R22e0/1 - R21e0/1`    |
|              | `101`  | `100.64.0.4/31`  | `100.64.0.0/23`   | `2001:AAAA:BB00:104::/64`  | `2001:AAAA:BB00::/48` | `R22e0/2 - R23e0/0`    |
|              | `101`  | `192.168.0.0/24` |                   | `2001:AAAA:BB00:192::/64`  | `2001:AAAA:BB00::/48` | `Loopback's`           |
| `Москва`     | `1001` | `100.65.0.0/31`  | `100.65.0.0/23`   | `2001:AAAA:BB01:100::/64`  | `2001:AAAA:BB01::/48` | `R14e0/3 - R19e0/0`    |
|              | `1001` | `100.65.0.2/31`  | `100.65.0.0/23`   | `2001:AAAA:BB01:102::/64`  | `2001:AAAA:BB01::/48` | `R14e0/0 - R12e0/2`    |
|              | `1001` | `100.65.0.4/31`  | `100.65.0.0/23`   | `2001:AAAA:BB01:104::/64`  | `2001:AAAA:BB01::/48` | `R14e0/1 - R13e0/3`    |
|              | `1001` | `100.65.0.6/31`  | `100.65.0.0/23`   | `2001:AAAA:BB01:106::/64`  | `2001:AAAA:BB01::/48` | `R15e0/1 - R12e0/3`    |
|              | `1001` | `100.65.0.8/31`  | `100.65.0.0/23`   | `2001:AAAA:BB01:108::/64`  | `2001:AAAA:BB01::/48` | `R15e0/0 - R13e0/2`    |
|              | `1001` | `100.65.0.10/31` | `100.65.0.0/23`   | `2001:AAAA:BB01:110::/64`  | `2001:AAAA:BB01::/48` | `R15e0/3 - R20e0/0`    |
|              | `1001` | `100.65.0.12/31` | `100.65.0.0/23`   | `2001:AAAA:BB01:112::/64`  | `2001:AAAA:BB01::/48` | `R12e0/1 - R13e0/1`    |
|              | `1001` | `192.168.1.0/24` |                   | `2001:AAAA:BB01:192::/64`  | `2001:AAAA:BB01::/48` | `Loopback's`           |
|              | `1001` | `172.22.1.0/24`  |                   | `2001:AAAA:BB01:172::/64`  | `2001:AAAA:BB01::/48` | `Управление Vlan 10`   |
|              | `1001` | `10.1.0.0/24`    | `10.1.0.0/21`     | `2001:AAAA:BB01:1001::/64` | `2001:AAAA:BB01::/48` | `Пользователи Vlan 11` |
|              | `1001` | `10.1.1.0/24`    | `10.1.0.0/21`     | `2001:AAAA:BB01:1002::/64` | `2001:AAAA:BB01::/48` | `Пользователи Vlan 12` |
| `Ламас`      | `302`  | `100.68.0.0/31`  | `100.68.0.0/23`   | `2001:AAAA:BB06:100::/64`  | `2001:AAAA:BB06::/48` | `R21e0/0 - R15e0/2`    |
|              | `302`  | `100.68.0.2/31`  | `100.68.0.0/23`   | `2001:AAAA:BB06:102::/64`  | `2001:AAAA:BB06::/48` | `R21e0/2 - R24e0/0`    |
|              | `302`  | `192.168.6.0/24` |                   | `2001:AAAA:BB06:192::/64`  | `2001:AAAA:BB06::/48` | `Loopback's`           |

#### 1.3. Таблица IP адресов.

| Расположение | Устройство | Интерфейс    | IPv4 адрес     | Родит. сеть       | IPv6 адрес                     | Родительская сеть          | Описание                   |
| ------------ | ---------- | ------------ | -------------- | ----------------- | ------------------------------ | -------------------------- | -------------------------- |
| `Киторн`     |            |              |                |                   |                                |                            |                            |
|              | `R22`      | `Lo0`        | `192.168.0.22` | `192.168.0.22/32` | `2001:AAAA:BB00:192::22/128`   | `2001:AAAA:BB00:192::/64`  | `Loopback R22`             |
|              |            | `e0/0`       | `100.64.0.0`   | `100.64.0.0/31`   | `2001:AAAA:BB00:100::E0/64`    | `2001:AAAA:BB00:100::/64`  |                            |
|              |            |              |                |                   | `FE80::22:E0`                  |                            |                            |
|              |            | `e0/1`       | `100.64.0.2`   | `100.64.0.2/31`   | `2001:AAAA:BB00:102::2:E1/64`  | `2001:AAAA:BB00:102::/64`  |                            |
|              |            |              |                |                   | `FE80::22:E1`                  | `FE80::/10`                |                            |
|              |            | `e0/2`       | `100.64.0.4`   | `100.64.0.2/31`   | `2001:AAAA:BB00:104::4:E2/64`  | `2001:AAAA:BB00:104::/64`  |                            |
|              |            |              |                |                   | `FE80::22:E2`                  |                            |                            |
| `Москва`     |            |              |                |                   |                                |                            |                            |
|              | `R14`      | `Lo0`        | `192.168.1.14` | `192.168.1.14/32` | `2001:AAAA:BB01:192::14/128`   | `2001:AAAA:BB01:192::/64`  | `Loopback R14`             |
|              |            | `e0/2`       | `100.64.0.1`   | `100.64.0.0/31`   | `2001:AAAA:BB00:100::1:E2/64`  | `2001:AAAA:BB00:100::/64`  |                            |
|              |            |              |                |                   | `FE80::14:E2`                  | `FE80::/10`                |                            |
|              |            | `e0/3`       | `100.65.0.0`   | `100.65.0.0/31`   | `2001:AAAA:BB01:100::E3/64`    | `2001:AAAA:BB01:100::/64`  |                            |
|              |            |              |                |                   | `FE80::14:E3`                  | `FE80::/10`                |                            |
|              |            | `e0/0`       | `100.65.0.2`   | `100.65.0.2/31`   | `2001:AAAA:BB01:102::2:E0/64`  | `2001:AAAA:BB01:102::/64`  |                            |
|              |            |              |                |                   | `FE80::14:E0`                  | `FE80::/10`                |                            |
|              |            | `e0/1`       | `100.65.0.4`   | `100.65.0.4/31`   | `2001:AAAA:BB01:104::4:E1/64`  | `2001:AAAA:BB01:104::/64`  |                            |
|              |            |              |                |                   | `FE80::14:E1`                  | `FE80::/10`                |                            |
|              | `R15`      | `Lo0`        | `192.168.1.15` | `192.168.1.15/32` | `2001:AAAA:BB01:192::15/128`   | `2001:AAAA:BB01:192::/64`  | `Loopback R15`             |
|              |            | `e0/2`       | `100.68.0.1`   | `100.68.0.1/31`   | `2001:AAAA:BB06:100::1:E2/64`  | `2001:AAAA:BB06:100::/64`  |                            |
|              |            |              |                |                   | `FE80::15:E2`                  | `FE80::/10`                |                            |
|              |            | `e0/1`       | `100.65.0.6`   | `100.65.0.6/31`   | `001:AAAA:BB01:106::6:E1/64`   | `2001:AAAA:BB01:106::/64`  |                            |
|              |            |              |                |                   | `FE80::15:E1`                  | `FE80::/10`                |                            |
|              |            | `e0/0`       | `100.65.0.8`   | `100.65.0.8/31`   | `2001:AAAA:BB01:108::8:E0/64`  | `2001:AAAA:BB01:108::/64`  |                            |
|              |            |              |                |                   | `FE80::15:E0`                  | `FE80::/10`                |                            |
|              |            | `e0/3`       | `100.65.0.10`  | `100.65.0.10/31`  | `2001:AAAA:BB01:110::10:E3/64` | `2001:AAAA:BB01:110::/64`  |                            |
|              |            |              |                |                   | `FE80::15:E3`                  | `FE80::/10`                |                            |
|              | `R19`      | `Lo0`        | `192.168.1.19` | `192.168.1.19/32` | `2001:AAAA:BB01:192::19/128`   | `2001:AAAA:BB01:192::/64`  | `Loopback R19`             |
|              |            | `e0/0`       | `100.65.0.1`   | `100.65.0.0/31`   | `2001:AAAA:BB01:100::1:E0/64`  | `2001:AAAA:BB01:100::/64`  |                            |
|              |            |              |                |                   | `FE80::19:E0`                  | `FE80::/10`                |                            |
|              | `R12`      | `Lo0`        | `192.168.1.12` | `192.168.1.12/32` | `2001:AAAA:BB01:192::12/128`   | `2001:AAAA:BB01:192::/64`  | `Loopback R12`             |
|              |            | `e0/2`       | `100.65.0.3`   | `100.65.0.2/31`   | `2001:AAAA:BB01:102::3:E2/64`  | `2001:AAAA:BB01:102::/64`  |                            |
|              |            |              |                |                   | `FE80::12:E2`                  | `FE80::/10`                |                            |
|              |            | `e0/3`       | `100.65.0.7`   | `100.65.0.6/31`   | `2001:AAAA:BB01:106::7:E3/64`  | `2001:AAAA:BB01:106::/64`  |                            |
|              |            |              |                |                   | `FE80::12:E3`                  | `FE80::/10`                |                            |
|              |            | `e0/1`       | `100.65.0.12`  | `100.65.0.12/31`  | `2001:AAAA:BB01:112::12:E1/64` | `2001:AAAA:BB01:112::/64`  |                            |
|              |            |              |                |                   | `FE80::12:E1`                  | `FE80::/10`                |                            |
|              |            | `e0/0`       | `N/A`          | `N/A`             | `N/A`                          | `N/A`                      |                            |
|              |            | `e0/0.10`    | `172.22.1.2`   | `172.22.1.0/24`   | `2001:AAAA:BB01:172::2/64`     | `2001:AAAA:BB01:172::/64`  | `Управление  Vlan 10`      |
|              |            |              | `172.22.1.1`   | `172.22.1.0/24`   | `2001:AAAA:BB01:172::1/64`     | `2001:AAAA:BB01:172::/64`  | `Виртуальный IP`           |
|              |            | `e0/0.11`    | `10.1.0.2`     | `10.1.0.0/24`     | `2001:AAAA:BB01:1011::2/64`    | `2001:AAAA:BB01:1011::/64` | `Пользовательский Vlan 11` |
|              |            |              | `10.1.0.1`     | `10.1.0.0/24`     | `2001:AAAA:BB01:1011::1/64`    | `2001:AAAA:BB01:1011::/64` | `Виртуальный IP`           |
|              |            | `e0/0.12`    | `10.1.1.2`     | `10.1.1.0/24`     | `2001:AAAA:BB01:1012::2/64`    | `2001:AAAA:BB01:1012::/64` | `Пользовательский Vlan 12` |
|              |            |              | `10.1.1.1`     | `10.1.1.0/24`     | `2001:AAAA:BB01:1012::1/64`    | `2001:AAAA:BB01:1012::/64` | `Виртуальный IP`           |
|              | `R13`      | `Lo0`        | `192.168.1.13` | `192.168.1.13/32` | `2001:AAAA:BB01:192::13/128`   | `2001:AAAA:BB01:192::/64`  | `Loopback R13`             |
|              |            | `e0/1`       | `100.65.0.13`  | `100.65.0.12/31`  | `2001:AAAA:BB01:112::13:E1/64` | `2001:AAAA:BB01:112::/64`  |                            |
|              |            |              |                |                   | `FE80::13:E1`                  | `FE80::/10`                |                            |
|              |            | `e0/3`       | `100.65.0.5`   | `100.65.0.4/31`   | `2001:AAAA:BB01:104::5:E3/64`  | `2001:AAAA:BB01:104::/64`  |                            |
|              |            |              |                |                   | `FE80::13:E3`                  | `FE80::/10`                |                            |
|              |            | `e0/2`       | `100.65.0.9`   | `100.65.0.8/31`   | `2001:AAAA:BB01:108::9:E2/64`  | `2001:AAAA:BB01:108::/64`  |                            |
|              |            |              |                |                   | `FE80::13:E2`                  | `FE80::/10`                |                            |
|              |            | `e0/0`       | `N/A`          | `N/A`             | `N/A`                          | `N/A`                      |                            |
|              |            | `e0/0.10`    | `172.22.1.3`   | `172.22.1.0/24`   | `2001:AAAA:BB01:172::3/64`     | `2001:AAAA:BB01:172::/64`  | `Управление  Vlan 10`      |
|              |            |              | `172.22.1.1`   | `172.22.1.0/24`   | `2001:AAAA:BB01:172::1/64`     | `2001:AAAA:BB01:172::/64`  | `Виртуальный IP`           |
|              |            | `e0/0.11`    | `10.1.0.3`     | `10.1.0.0/24`     | `2001:AAAA:BB01:1011::3/64`    | `2001:AAAA:BB01:1011::/64` | `Пользовательский Vlan 11` |
|              |            |              | `10.1.0.1`     | `10.1.0.0/24`     | `2001:AAAA:BB01:1011::1/64`    | `2001:AAAA:BB01:1011::/64` | `Виртуальный IP`           |
|              |            | `e0/0.12`    | `10.1.1.3`     | `10.1.1.0/24`     | `2001:AAAA:BB01:1012::3/64`    | `2001:AAAA:BB01:1012::/64` | `Пользовательский Vlan 12` |
|              |            |              | `10.1.1.1`     | `10.1.1.0/24`     | `2001:AAAA:BB01:1012::1/64`    | `2001:AAAA:BB01:1012::/64` | `Виртуальный IP`           |
|              | `R20`      | `Lo0`        | `192.168.1.20` | `192.168.1.20/32` | `2001:AAAA:BB01:192::20/128`   | `2001:AAAA:BB01:192::/64`  | `Loopback R19`             |
|              |            | `e0/0`       | `100.65.0.11`  | `100.65.0.10/31`  | `2001:AAAA:BB01:110::11:E0/64` | `2001:AAAA:BB01:110::/64`  |                            |
|              |            |              |                |                   | `FE80::20:E0`                  | `FE80::/10`                |                            |
|              | `SW2`      | `Int Vlan10` | `172.22.1.12`  | `172.22.1.0/24`   | `2001:AAAA:BB01:172::2c/64`    | `2001:AAAA:BB01:172::/64`  |                            |
|              |            |              |                |                   | `FE80::c2:10`                  | `FE80::/10`                |                            |
|              | `SW3`      | `Int Vlan10` | `172.22.1.13`  | `172.22.1.0/24`   | `2001:AAAA:BB01:172::3c/64`    | `2001:AAAA:BB01:172::/64`  |                            |
|              |            |              |                |                   | `FE80::c3:10`                  | `FE80::/10`                |                            |
|              | `SW4`      | `Int Vlan10` | `172.22.1.14`  | `172.22.1.0/24`   | `2001:AAAA:BB01:172::4c/64`    | `2001:AAAA:BB01:172::/64`  |                            |
|              |            |              |                |                   | `FE80::c4:10`                  | `FE80::/10`                |                            |
|              | `SW5`      | `Int Vlan10` | `172.22.1.15`  | `172.22.1.0/24`   | `2001:AAAA:BB01:172::5c/64`    | `2001:AAAA:BB01:172::/64`  |                            |
|              |            |              |                |                   | `FE80::c5:10`                  | `FE80::/10`                |                            |
|              | `VPC1`     |              | `10.1.0.4`     | `10.1.0.0/24`     | `autoconfig`                   | `2001:AAAA:BB01:1011::/64` |                            |
|              | `VPC7`     |              | `10.1.1.7`     | `10.1.1.0/24`     | `autoconfig`                   | `2001:AAAA:BB01:1012::/64` |                            |
| `Ламас`      |            |              |                |                   |                                |                            |                            |
|              | `R21`      | `Lo0`        | `192.168.6.21` | `192.168.6.21/32` | `2001:AAAA:BB06:192::21/128`   | `2001:AAAA:BB06:192::/64`  | `Loopback R21`             |
|              |            | `e0/1`       | `100.64.0.3`   | `100.64.0.2/31`   | `2001:AAAA:BB00:102::3:E1/64`  | `2001:AAAA:BB00:102::/64`  |                            |
|              |            |              |                |                   | `FE80::21:E1`                  | `FE80::/10`                |                            |
|              |            | `e0/0`       | `100.68.0.0/`  | `100.68.0.0/31`   | `2001:AAAA:BB06:100::E0/64`    | `2001:AAAA:BB06:100::/64`  |                            |
|              |            |              |                |                   | `FE80::21:E0`                  | `FE80::/10`                |                            |
|              |            | `e0/2`       | `100.68.0.2`   | `100.68.0.2/31`   | `2001:AAAA:BB06:102::2:E2/64`  | `2001:AAAA:BB06:102::/64`  |                            |
|              |            |              |                |                   | FE80::21:E2                    | FE80::/10                  |                            |



#### 1.4 Разделение сети на зоны.

По условию лабораторной работы:

1. Маршрутизаторы R14-R15 находятся в зоне 0 - и являются ASBR (с выходом к провайдеру).
2. Маршрутизаторы R12-R13 находятся в зоне 10, но так как между маршрутизаторами R14 и R15, находящимися в area 0 нет прямых линков, интерфейсы e0/2 и e0/3 на маршрутизаторах R12 и R13 так же определила в area 0.
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию, из этого можно сделать вывод, что area 101 является "Totally Stubby Area".
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов сетей зоны 101 (настрою с помощью фильтрации далее).

Разделение на зоны  рисунок 1.

![Рисунок 1.](https://github.com/darkmikos/otus.ru/blob/master/lab13/ospf_area.png)



#### 2. Настройка сетевого оборудования.

#### 2.1 Настройка на маршрутизаторах протокола OSPF.

В данном разделе настраиваем на маршрутизаторах протокол OSPF. Ниже описаны команды для настройки маршрутизаторов.

------
**Маршрутизатор R14:**

Статический маршрут по-умолчанию для IPv4 и IPv6.

```
conf t
!
ip route 0.0.0.0 0.0.0.0 100.64.0.0
ipv6 route ::/0 2001:AAAA:BB00:100::E0
```

Включаем OSPF процесс и  указываем идентификатор маршрутизатора.

```
conf t
!
router ospf 10
!
 router-id 14.14.14.14
```

Объявляем соседям, что маршрутизатор имеет маршрут по-умолчанию.

```
conf t
!
router ospf 10
!
 default-information originate
```

Переводим интерфейсы  участвующие в процессе OSPF в пассивное состояние. На нужных интерфейсах отключаем пассивное состояние.

```
conf t
!
router ospf 10
!
 passive-interface default
 !
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/3
```

Указываем, что Area 101 является "Totally Stubby Area".

```
conf t
!
router ospf 10
!
 area 101 stub no-summary
```

По аналогии с IPv4 добавляем  настройки для IPv6.

```
conf t
!
ipv6 router ospf 10
 router-id 14.14.14.14
 default-information originate
 area 101 stub no-summary

```

Далее настраиваем интерфейсы, участвующие в процессе с учетом соответствующих area. Для интерфейса Ethernet0/3 указываем настройку point-to-point, так как с маршрутизатором R19 прямое соединение, данная настройка позволит исключить маршрутизатор R19 из выборов DR, BDR.


```
conf t
!
interface Loopback0
 ip ospf 10 area 0
 ipv6 ospf 10 area 0
 exit
!
interface Ethernet0/0
 ip ospf 10 area 0
 ipv6 ospf 10 area 0
 exit
!
interface Ethernet0/1
 ip ospf 10 area 0
 ipv6 ospf 10 area 0
 exit
!
interface Ethernet0/3
 ip ospf network point-to-point
 ip ospf 10 area 101
 ipv6 ospf 10 area 101
 exit
```

Настройка маршрутизатора R14 находятся в папке [cfg](https://github.com/darkmikos/otus.ru/tree/master/lab13/cfg).

По окончанию настройки проведем некоторые проверки.

* С помощью команды show ip route static проверим статический маршрут по-умолчанию.

```
R14#sh ip route static 
Gateway of last resort is 100.64.0.0 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 100.64.0.0
```

* С помощью команды show ip protocols можно посмотреть какие интерфейсы являются пассивными, проверить Router ID, наличие фильтров и другое.

```
Routing Protocol is "ospf 10"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 14.14.14.14
  It is an autonomous system boundary router
 Redistributing External Routes from,
  Number of areas in this router is 2. 1 normal 1 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
  Routing on Interfaces Configured Explicitly (Area 0):
    Loopback0
    Ethernet0/1
    Ethernet0/0
  Passive Interface(s):
    Ethernet0/2
    Loopback0
    RG-AR-IF-INPUT1
    VoIP-Null0
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 110)
```

------

**Маршрутизатор R15:**

Основные настройки совпадают с настройками на R14.


```
conf t
!
ip route 0.0.0.0 0.0.0.0 100.68.0.0
ipv6 route ::/0 2001:AAAA:BB06:100::E0
!
router ospf 10
 router-id 15.15.15.15
 default-information originate
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/3
 exit
!
ipv6 router ospf 10
 router-id 15.15.15.15
 default-information originate
 exit
!
interface Loopback0
 ip ospf 10 area 0
 ipv6 ospf 10 area 0
 exit
!
interface Ethernet0/0
 ip ospf 10 area 0
 ipv6 ospf 10 area 0
 exit
!
interface Ethernet0/1
 ip ospf 10 area 0
 ipv6 ospf 10 area 0
 exit
!
interface Ethernet0/3
 ip ospf network point-to-point
 ip ospf 10 area 102
 ipv6 ospf 10 area 102
 exit
exit
```

Настройки маршрутизатора R15 находятся в папке [cfg](https://github.com/darkmikos/otus.ru/tree/master/lab13/cfg).

По окончанию настройки проведем некоторые проверки.

* С помощью команды show ip route static проверим статический маршрут по-умолчанию.

```
R15#sh ip route     
Gateway of last resort is 100.68.0.0 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 100.68.0.0

```

* С помощью команды show ip protocols можно посмотреть какие интерфейсы являются пассивными, проверить Router ID, наличие фильтров и другое.

  ```
  R15#sh ip protocols 
  *** IP Routing is NSF aware ***
  
  Routing Protocol is "ospf 10"
    Outgoing update filter list for all interfaces is not set
    Incoming update filter list for all interfaces is not set
    Router ID 15.15.15.15
    It is an area border and autonomous system boundary router
   Redistributing External Routes from,
    Number of areas in this router is 2. 2 normal 0 stub 0 nssa
    Maximum path: 4
    Routing for Networks:
    Routing on Interfaces Configured Explicitly (Area 0):
      Loopback0
      Ethernet0/1
      Ethernet0/0
    Routing on Interfaces Configured Explicitly (Area 102):
      Ethernet0/3
    Passive Interface(s):
      Ethernet0/2
      Loopback0
      RG-AR-IF-INPUT1
      VoIP-Null0
    Routing Information Sources:
      Gateway         Distance      Last Update
    Distance: (default is 110)
  ```

------

**Маршрутизатор R12:**


```
conf t
!
router ospf 10
 router-id 12.12.12.12
 passive-interface default
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 no passive-interface Ethernet0/3
 exit
ipv6 router ospf 10
 router-id 12.12.12.12 
 exit
!
interface Loopback0
 ip ospf 10 area 10
 ipv6 ospf 10 area 10
 exit
!
interface Ethernet0/0.10
 ip ospf 10 area 10
 ipv6 ospf 10 area 10
 exit
!
interface Ethernet0/0.11
 ip ospf 10 area 10
 ipv6 ospf 10 area 10
 exit
!
interface Ethernet0/0.12
 ip ospf 10 area 10
 pv6 ospf 10 area 10
 exit
!
interface Ethernet0/1
 ip ospf 10 area 10
 ipv6 ospf 10 area 10
 exit
!
interface Ethernet0/2
 ip ospf 10 area 0
 ipv6 ospf 10 area 0
 exit
!
interface Ethernet0/3
 ip ospf 10 area 0
 ipv6 ospf 10 area 0
 exit
exit
```

 Настройки маршрутизатора R12 находятся в папке [cfg](https://github.com/darkmikos/otus.ru/tree/master/lab13/cfg).

* Проверим наличие маршрутов полученных по протоколу OSPF для IPv4.

```
R12#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 100.65.0.6 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 100.65.0.6, 03:33:34, Ethernet0/3
                [110/1] via 100.65.0.2, 03:33:29, Ethernet0/2
      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        10.1.0.0/24 is directly connected, Ethernet0/0.11
L        10.1.0.2/32 is directly connected, Ethernet0/0.11
C        10.1.1.0/24 is directly connected, Ethernet0/0.12
L        10.1.1.2/32 is directly connected, Ethernet0/0.12
      100.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
O IA     100.65.0.0/31 [110/20] via 100.65.0.2, 03:33:29, Ethernet0/2
C        100.65.0.2/31 is directly connected, Ethernet0/2
L        100.65.0.3/32 is directly connected, Ethernet0/2
O        100.65.0.4/31 [110/20] via 100.65.0.2, 03:33:29, Ethernet0/2
C        100.65.0.6/31 is directly connected, Ethernet0/3
L        100.65.0.7/32 is directly connected, Ethernet0/3
O        100.65.0.8/31 [110/20] via 100.65.0.6, 03:33:34, Ethernet0/3
O IA     100.65.0.10/31 [110/20] via 100.65.0.6, 03:33:34, Ethernet0/3
C        100.65.0.12/31 is directly connected, Ethernet0/1
L        100.65.0.12/32 is directly connected, Ethernet0/1
      172.22.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.22.1.0/24 is directly connected, Ethernet0/0.10
L        172.22.1.2/32 is directly connected, Ethernet0/0.10
      192.168.1.0/32 is subnetted, 3 subnets
C        192.168.1.12 is directly connected, Loopback0
O        192.168.1.14 [110/11] via 100.65.0.2, 03:33:29, Ethernet0/2
O        192.168.1.15 [110/11] via 100.65.0.6, 03:33:34, Ethernet0/3
```

Помимо других маршрутов, маршрутизатор видит два шлюза по-умолчанию. 

```
O*E2  0.0.0.0/0 [110/1] via 100.65.0.6, 03:33:34, Ethernet0/3
                [110/1] via 100.65.0.2, 03:33:29, Ethernet0/2
```

* Проверим наличие маршрутов полученных по протоколу OSPF для IPv6.

```
R12#sh ipv6 route
IPv6 Routing Table - default - 24 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, l - LISP
OE2 ::/0 [110/1], tag 10
     via FE80::15:E1, Ethernet0/3
     via FE80::14:E0, Ethernet0/2
OI  2001:AAAA:BB01:100::/64 [110/20]
     via FE80::14:E0, Ethernet0/2
C   2001:AAAA:BB01:102::/64 [0/0]
     via Ethernet0/2, directly connected
L   2001:AAAA:BB01:102::2:E2/128 [0/0]
     via Ethernet0/2, receive
O   2001:AAAA:BB01:104::/64 [110/20]
     via FE80::14:E0, Ethernet0/2
C   2001:AAAA:BB01:106::/64 [0/0]
     via Ethernet0/3, directly connected
L   2001:AAAA:BB01:106::7:E3/128 [0/0]
     via Ethernet0/3, receive
O   2001:AAAA:BB01:108::/64 [110/20]
     via FE80::15:E1, Ethernet0/3
OI  2001:AAAA:BB01:110::/64 [110/20]
     via FE80::15:E1, Ethernet0/3
C   2001:AAAA:BB01:112::/64 [0/0]
     via Ethernet0/1, directly connected
L   2001:AAAA:BB01:112::12:E1/128 [0/0]
     via Ethernet0/1, receive
C   2001:AAAA:BB01:172::/64 [0/0]
     via Ethernet0/0.10, directly connected
L   2001:AAAA:BB01:172::1/128 [0/0]
     via Ethernet0/0.10, receive
L   2001:AAAA:BB01:172::2/128 [0/0]
     via Ethernet0/0.10, receive
LC  2001:AAAA:BB01:192::12/128 [0/0]
     via Loopback0, receive
O   2001:AAAA:BB01:192::14/128 [110/10]
     via FE80::14:E0, Ethernet0/2
O   2001:AAAA:BB01:192::15/128 [110/10]
     via FE80::15:E1, Ethernet0/3
C   2001:AAAA:BB01:1011::/64 [0/0]
     via Ethernet0/0.11, directly connected
L   2001:AAAA:BB01:1011::1/128 [0/0]
     via Ethernet0/0.11, receive
L   2001:AAAA:BB01:1011::2/128 [0/0]
     via Ethernet0/0.11, receive
C   2001:AAAA:BB01:1012::/64 [0/0]
     via Ethernet0/0.12, directly connected
L   2001:AAAA:BB01:1012::1/128 [0/0]
     via Ethernet0/0.12, receive
L   2001:AAAA:BB01:1012::2/128 [0/0]
     via Ethernet0/0.12, receive
L   FF00::/8 [0/0]
     via Null0, receive
```

Протокол OSPF при работе с IPv6 использует link-local адреса интерфейсов, через которые доступен шлюз по-умолчанию.

```
OE2 ::/0 [110/1], tag 10
     via FE80::15:E1, Ethernet0/3
     via FE80::14:E0, Ethernet0/2
```

------
 **Маршрутизатор R13**

```
conf t
!
ipv6 unicast-routing
!
router ospf 10
 router-id 13.13.13.13
 passive-interface default
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 no passive-interface Ethernet0/3
 exit
ipv6 router ospf 10
 router-id 13.13.13.13 
 exit
!
interface Loopback0
 ip ospf 10 area 10
 ipv6 ospf 10 area 10
 exit
!
interface Ethernet0/0.10
 ip ospf 10 area 10
 ipv6 ospf 10 area 10
 exit
!
interface Ethernet0/0.11
 ip ospf 10 area 10
 ipv6 ospf 10 area 10
 exit
!
interface Ethernet0/0.12
 ip ospf 10 area 10
 pv6 ospf 10 area 10
 exit
!
interface Ethernet0/1
 ip ospf 10 area 10
 ipv6 ospf 10 area 10
 exit
!
interface Ethernet0/2
 ip ospf 10 area 0
 ipv6 ospf 10 area 0
 exit
!
interface Ethernet0/3
 ip ospf 10 area 0
 ipv6 ospf 10 area 0
 no shutdown
 exit
exit
```

 Настройки маршрутизатора R13 находятся в папке [cfg](https://github.com/darkmikos/otus.ru/tree/master/lab13/cfg).

* Проверим наличие маршрутов по умолчанию полученных по протоколу OSPF для IPv4.

```
R13#sh ip route

O*E2  0.0.0.0/0 [110/1] via 100.65.0.8, 00:04:55, Ethernet0/2
                [110/1] via 100.65.0.4, 00:00:15, Ethernet0/3
```

* Проверим наличие маршрутов по умолчанию полученных по протоколу OSPF для IPv6.

```
R13#sh ipv6 route

OE2 ::/0 [110/1], tag 10
     via FE80::15:E0, Ethernet0/2
     via FE80::14:E1, Ethernet0/3
```



#### 2.2. Фильтрация OSPF.

Варианты фильтрации показаны в этом разделе на примере маршрутизаторов R19 (определение area, как total stub, командой area 101 stub no-summary) и R20 (distribute-list для IPv6) в паре с R15 (prefix-list для IPv4).

------

**Маршрутизатор R19**

```
conf t
!
router ospf 10
 router-id 19.19.19.19
 area 101 stub no-summary
 passive-interface default
 no passive-interface Ethernet0/0 
 exit
ipv6 router ospf 10
 router-id 19.19.19.19
 area 101 stub no-summary
 exit
!
interface Loopback0
 ip ospf 10 area 101
 ipv6 ospf 10 area 101
!
interface Ethernet0/0
 ip ospf network point-to-point
 ip ospf 10 area 101
 ipv6 ospf 10 area 101
 exit
exit
```

* Проверим наличие маршрутов полученных по протоколу OSPF для IPv4.

```
R19#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 100.65.0.0 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 100.65.0.0, 00:35:42, Ethernet0/0
      100.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        100.65.0.0/31 is directly connected, Ethernet0/0
L        100.65.0.1/32 is directly connected, Ethernet0/0
      192.168.1.0/32 is subnetted, 1 subnets
C        192.168.1.19 is directly connected, Loopback0
```

* Проверим наличие маршрутов полученных по протоколу OSPF для IPv6.

  ```
  R19#sh ipv6 route
  IPv6 Routing Table - default - 5 entries
  Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
         B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
         H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
         IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
         ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
         O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
         ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, l - LISP
  OI  ::/0 [110/11]
       via FE80::14:E3, Ethernet0/0
  C   2001:AAAA:BB01:100::/64 [0/0]
       via Ethernet0/0, directly connected
  L   2001:AAAA:BB01:100::1:E0/128 [0/0]
       via Ethernet0/0, receive
  LC  2001:AAAA:BB01:192::19/128 [0/0]
       via Loopback0, receive
  L   FF00::/8 [0/0]
       via Null0, receive
  ```

  R19 получил информацию о шлюзе по-умолчанию по протоколу OSPF от маршрутизатора R14. Так как R14 является ABR (граничным), а area 101, в которой находятся оба маршрутизатора, является totally stubby area, то он (R14) объявляет себя шлюзом по-умолчанию.

------



**Маршрутизатор R15**

По условию лабораторной работы, необходимо отфильтровать приходящий на маршрутизатор R20 трафик из Area 101. Для этого необходимо сделать настройки на двух маршрутизаторах R15 и R20.

Создадим на маршрутизаторе R 15 prefix-list и активируем его для Area 102. Весь трафик входящий в area 102 будет фильтроваться согласно prefix-list Area101-Deny.

```
conf t
!
ip prefix-list Area101-Deny seq 10 deny 10.1.0.0/31
ip prefix-list Area101-Deny seq 15 deny 192.168.1.19/32
ip prefix-list Area101-Deny seq 99 permit 0.0.0.0/0 le 32
!
router ospf 10
 area 102 filter-list prefix Area101-Deny in
```


Проверила фильтрацию трафика по IPv4 на маршрутизаторе R20. Действительно, маршруты, запрещенные в prefix-list не попадают в таблицу маршрутизации (рис.8).

Рисунок 8.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab006_OSPF/Filter_R20.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab006_OSPF/Filter_R20.png)
------
**Маршрутизатор R20**

Для настройки фильтрации трафика по IPV6 на маршрутизаторе R20 сделела следующие настройки:

```
conf t
!
ipv6 prefix-list Area101-Deny-IPV6 seq 5 deny 2001:AAAA:BB01:192::19/128
ipv6 prefix-list Area101-Deny-IPV6 seq 10 deny 2001:AAAA:BB01:100::/64
ipv6 prefix-list Area101-Deny-IPV6 seq 99 permit ::/0 le 128
!
ipv6 router ospf 10
 router-id 20.20.20.20
 distribute-list prefix-list Area101-Deny-IPV6 in
```

Весь трафик приходящий на маршрутизатор R20 будет фильтроваться согласно prefix-list Area101-Deny-IPV6.

Проверила (рис.9)

Рисунок 9.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab006_OSPF/Filter_R20_IPV6.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab006_OSPF/Filter_R20_IPV6.png)

Трафик фильтруется.

Файлы с полной конфигурацией маршрутизаторов находятся в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab006_OSPF/configs) в файлах ***RRR-int.txt\***. Первые символы в названии файлов соответствуют именам сетевых устройств.

#### ***IV. Итоговая схема.\***

На рисунке 10 размещены используемые сети, IPv4 и IPv6 адреса маршрутизаторов, коммутаторов и персональных компьютеров, а так же испльзуемые VLAN.

Рисунок 10.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab006_OSPF/Shema_Lab6.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab006_OSPF/Shema_Lab6.png)