## **Маршрутизация на основе политик (PBR)**

### Цель:

Цель:

Настроить политику маршрутизации в офисе пгт. Чокурдах. Распределить трафик между 2 линками.

В этой самостоятельной работе будет выполнены следующие цели:

1. Необходимо настроить политику маршрутизации для сетей офиса
2. Распределить трафик между двумя линками с провайдером
3. Настроить отслеживание линка через технологию IP SLA
4. Выполнить настройку для офиса Лабытнанги маршрута по-умолчанию
5. План работы и изменения зафиксировать в документации

### Шаги выполнения:

1. [Документирование адресного пространства для лабораторного стенда.](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/README.md#I-документирование-адресного-пространства-для-лабораторного-стенда)

   a. [Таблица выделенных подсетей.](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/README.md#a-таблица-выделенных-подсетей)

   b. [Таблица IP адресов.](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/README.md#b-таблица-ip-адресов)

2. [Настройка сетевого оборудования.](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/README.md#II-настройка-сетевого-оборудования)

   a. [Настройка маршрута по умолчанию для офиса Лабытнанги.](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/README.md#a-настройка-маршрута-по-умолчанию-для-офиса-лабытнанги)

   b. [Настройка маршрута по умолчанию для сетей офиса Чокурдах на роутерах Триады.](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/README.md#b-настройка-маршрута-по-умолчанию-для-сетей-офиса-чокурдах-на-роутерах-триады)

   c. [Распределение трафика между двумя линками с провайдером.](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/README.md#c-распределение-трафика-между-двумя-линками-с-провайдером)

   d. [Настройка отслеживания линка через технологию IP SLA.](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/README.md#c-настройка-отслеживания-линка-через-технологию-ip-sla)

3. [Проверка работоспособности системы.](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/README.md#III-проверка-работоспособности-системы)

4. [Итоговая схема.](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/README.md#IV-итоговая-схема)

### Ход выполнения:

```
Документация оформлена на github, использован markdown.
Для выполнения лабораторной работы использовался эмулятор EVE-NG, терминал Gnome 3.38.
```

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

#### 1.2 Таблица выделенных подсетей для использования в данной работе.

Таблица 2

| Расположение | AS    | IPv4 сеть        | Родительская сеть | IPv6 сеть                  | Родительская сеть     | Описание               |
| ------------ | ----- | ---------------- | ----------------- | -------------------------- | --------------------- | ---------------------- |
| `Чокурдах`   |       |                  |                   |                            |                       |                        |
|              |       | `192.168.3.0/24` |                   | `2001:AAAA:BB03:192::/64`  | `2001:AAAA:BB03::/48` | `Loopback's`           |
|              |       | `172.22.3.0/24`  |                   | `2001:AAAA:BB03:172::/64`  | `2001:AAAA:BB03::/48` | `Управление Vlan 10`   |
|              |       | `10.3.0.0/24`    | `10.3.0.0/21`     | `2001:AAAA:BB03:1021::/64` | `2001:AAAA:BB03::/48` | `Пользователи Vlan 11` |
|              |       | `10.3.1.0/24`    | `10.3.0.0/21`     | `2001:AAAA:BB03:1022::/64` | `2001:AAAA:BB03::/48` | `Пользователи Vlan 12` |
| `Лабытнанги` |       |                  |                   |                            |                       |                        |
|              |       | `192.168.4.0/24` |                   | `2001:AAAA:BB04:192::/64`  | `2001:AAAA:BB04::/48` | `Loopback's`           |
| `Триада`     |       |                  |                   |                            |                       |                        |
|              | `520` | `100.67.0.0/31`  | `100.67.0.0/23`   | `2001:AAAA:BB05:100::/64`  | `2001:AAAA:BB05::/48` | `R23e0/2 - R24e0/2`    |
|              | `520` | `100.67.0.2/31`  | `100.67.0.0/23`   | `2001:AAAA:BB05:102::/64`  | `2001:AAAA:BB05::/48` | `R23e0/1 - R25e0/0`    |
|              | `520` | `100.67.0.4/31`  | `100.67.0.0/23`   | `2001:AAAA:BB05:104::/64`  | `2001:AAAA:BB05::/48` | `R24e0/1 - R26e0/0`    |
|              | `520` | `100.67.0.6/31`  | `100.67.0.0/23`   | `2001:AAAA:BB05:106::/64`  | `2001:AAAA:BB05::/48` | `R24e0/3 - R18e0/2`    |
|              | `520` | `100.67.0.8/31`  | `100.67.0.0/23`   | `2001:AAAA:BB05:108::/64`  | `2001:AAAA:BB05::/48` | `R25e0/2 - R26e0/2`    |
|              | `520` | `100.67.0.10/31` | `100.67.0.0/23`   | `2001:AAAA:BB05:110::/64`  | `2001:AAAA:BB05::/48` | `R25e0/3 - R28e0/1`    |
|              | `520` | `100.67.0.12/31` | `100.67.0.0/23`   | `2001:AAAA:BB05:112::/64`  | `2001:AAAA:BB05::/48` | `R25e0/1 - R27e0/0`    |
|              | `520` | `100.67.0.14/31` | `100.67.0.0/23`   | `2001:AAAA:BB05:114::/64`  | `2001:AAAA:BB05::/48` | `R26e0/3 - R18e0/3`    |
|              | `520` | `100.67.0.16/31` | `100.67.0.0/23`   | `2001:AAAA:BB05:116::/64`  | `2001:AAAA:BB05::/48` | `R26e0/1 - R28e0/0`    |
|              | `520` | `192.168.5.0/24` |                   | `2001:AAAA:BB05:192::/64`  | `2001:AAAA:BB05::/48` | `Loopback's`           |

#### 1.3 Таблица IP адресов.

| Расположение | Устройство | Интерфейс    | IPv4 адрес     | Родит. сеть       | IPv6 адрес                     | Родительская сеть          | Описание                   |
| ------------ | ---------- | ------------ | -------------- | ----------------- | ------------------------------ | -------------------------- | -------------------------- |
| `Чокурдах`   |            |              |                |                   |                                |                            |                            |
|              | `R28`      | `Lo0`        | `192.168.3.28` | `192.168.3.28/32` | `2001:AAAA:BB03:192::28/128`   | `2001:AAAA:BB03:192::/64`  | `Loopback R28`             |
|              |            | `e0/0`       | `100.67.0.17`  | `100.67.0.16/31`  | `2001:AAAA:BB05:116::17:E0/64` | `2001:AAAA:BB05:116::/64`  |                            |
|              |            |              |                |                   | `FE80::28:E0`                  | `FE80::/10`                |                            |
|              |            | `e0/1`       | `100.67.0.11`  | `100.67.0.10/31`  | `2001:AAAA:BB05:110::11:E1/64` | `2001:AAAA:BB05:110::/64`  |                            |
|              |            |              |                |                   | `FE80::28:E1`                  | `FE80::/10`                |                            |
|              |            | `e0/2`       | `N/A`          | `N/A`             | `N/A`                          | `N/A`                      |                            |
|              |            | `e0/2.10`    | `172.22.3.1`   | `172.22.3.0/24`   | `2001:AAAA:BB03:172::1/64`     | `2001:AAAA:BB03:172::/64`  | `Управление  Vlan 10`      |
|              |            |              |                |                   | `FE80::28:E210`                | `FE80::/10`                |                            |
|              |            | `e0/2.11`    | `10.3.0.1`     | `10.3.0.0/24`     | `2001:AAAA:BB03:1011::1/64`    | `2001:AAAA:BB03:1011::/64` | `Пользовательский Vlan 11` |
|              |            |              |                |                   | `FE80::28:E211`                | `FE80::/10`                |                            |
|              |            | `e0/2.12`    | `10.3.1.1`     | `10.3.1.0/24`     | `2001:AAAA:BB03:1012::1/64`    | `2001:AAAA:BB03:1012::/64` | `Пользовательский Vlan 12` |
|              |            |              |                |                   | `FE80::28:E212`                | `FE80::/10`                |                            |
|              | `SW29`     | `Int Vlan10` | `172.22.3.29`  | `172.22.3.0/24`   | `2001:AAAA:BB03:172::29с/64`   | `2001:AAAA:BB03:172::/64`  |                            |
|              |            |              |                |                   | `FE80::с29:10`                 | `FE80::/10`                |                            |
|              | `VPC30`    |              | `DHCP`         | `10.3.0.0/24`     | `autoconfig`                   | `2001:AAAA:BB03:1011::/64` |                            |
|              | `VPC31`    |              | `DHCP`         | `10.3.1.0/24`     | `autoconfig`                   | `2001:AAAA:BB03:1012::/64` |                            |
| `Лабытнанги` |            |              |                |                   |                                |                            |                            |
|              | `R27`      | `Lo0`        | `192.168.4.27` | `192.168.4.27/32` | `2001:AAAA:BB04:192::27/128`   | `2001:AAAA:BB04:192::/64`  | `Loopback R27`             |
|              |            | `e0/0`       | `100.67.0.13`  | `100.67.0.12/31`  | `2001:AAAA:BB05:112::13:E0/64` | `2001:AAAA:BB05:116::/64`  |                            |
|              |            |              |                |                   | `FE80::27:E0`                  | `FE80::/10`                |                            |
| `Триада`     |            |              |                |                   |                                |                            |                            |
|              | `R23`      | `Lo0`        | `192.168.5.23` | `192.168.5.23/32` | `2001:AAAA:BB05:192::23/128`   | `2001:AAAA:BB05:192::/64`  | `Loopback R23`             |
|              |            | `e0/0`       | `100.64.0.5`   | `100.64.0.4/31`   | `2001:AAAA:BB00:104::5:E0/64`  | `2001:AAAA:BB00:104::/64`  |                            |
|              |            |              |                |                   | `FE80::23:E0`                  | `FE80::/10`                |                            |
|              |            | `e0/2`       | `100.67.0.0`   | `100.67.0.0/31`   | `2001:AAAA:BB05:100::E2/64`    | `2001:AAAA:BB05:100::/64`  |                            |
|              |            |              |                |                   | `FE80::23:E2`                  | `FE80::/10`                |                            |
|              |            | `e0/1`       | `100.67.0.2`   | `100.67.0.2/31`   | `2001:AAAA:BB05:102::2:E1/64`  | `2001:AAAA:BB05:102::/64`  |                            |
|              |            |              |                |                   | `FE80::23:E1`                  | `FE80::/10`                |                            |
|              | `R24`      | `Lo0`        | `192.168.5.24` | `192.168.5.24/32` | `2001:AAAA:BB05:192::24/128`   | `2001:AAAA:BB05:192::/64`  | `Loopback R24`             |
|              |            | `e0/0`       | `100.68.0.3`   | `100.68.0.2/31`   | `2001:AAAA:BB06:102::3:E0/64`  | `2001:AAAA:BB06:102::/64`  |                            |
|              |            |              |                |                   | `FE80::24:E0`                  | `FE80::/10`                |                            |
|              |            | `e0/2`       | `100.67.0.1`   | `100.67.0.0/31`   | `2001:AAAA:BB05:100::1:E2/64`  | `2001:AAAA:BB05:100::/64`  |                            |
|              |            |              |                |                   | `FE80::24:E2`                  | `FE80::/10`                |                            |
|              |            | `e0/1`       | `100.67.0.4`   | `100.67.0.4/31`   | `2001:AAAA:BB05:104::4:E1/64`  | `2001:AAAA:BB05:104::/64`  |                            |
|              |            |              |                |                   | `FE80::24:E1`                  | `FE80::/10`                |                            |
|              |            | `e0/3`       | `100.67.0.6`   | `100.67.0.6/31`   | `2001:AAAA:BB05:106::6:E3/64`  | `2001:AAAA:BB05:106::/64`  |                            |
|              |            |              |                |                   | `FE80::24:E3`                  | `FE80::/10`                |                            |
|              | `R26`      | `Lo0`        | `192.168.5.26` | `192.168.5.26/32` | `2001:AAAA:BB05:192::26/128`   | `2001:AAAA:BB05:192::/64`  | `Loopback R26`             |
|              |            | `e0/0`       | `100.67.0.5`   | `100.67.0.4/31`   | `2001:AAAA:BB05:104::5:E0/64`  | `2001:AAAA:BB05:104::/64`  |                            |
|              |            |              |                |                   | `FE80::26:E0`                  | `FE80::/10`                |                            |
|              |            | `e0/3`       | `100.67.0.14`  | `100.67.0.14/31`  | `2001:AAAA:BB05:114::14:E3/64` | `2001:AAAA:BB05:114::/64`  |                            |
|              |            |              |                |                   | `FE80::26:E3`                  | `FE80::/10`                |                            |
|              |            | `e0/1`       | `100.67.0.16`  | `100.67.0.16/31`  | `2001:AAAA:BB05:116::16:E1/64` | `2001:AAAA:BB05:116::/64`  |                            |
|              |            |              |                |                   | `FE80::26:E1`                  | `FE80::/10`                |                            |
|              |            | `e0/2`       | `100.67.0.9`   | `100.67.0.8/31`   | `2001:AAAA:BB05:108::9:E2/64`  | `2001:AAAA:BB05:108::/64`  |                            |
|              |            |              |                |                   | `FE80::26:E2`                  | `FE80::/10`                |                            |
|              | `R25`      | `Lo0`        | `192.168.5.25` | `192.168.5.25/32` | `2001:AAAA:BB05:192::25/128`   | `2001:AAAA:BB05:192::/64`  | `Loopback R25`             |
|              |            | `e0/0`       | `100.67.0.3`   | `100.67.0.2/31`   | `2001:AAAA:BB05:102::3:E0/64`  | `2001:AAAA:BB05:102::/64`  |                            |
|              |            |              |                |                   | `FE80::25:E0`                  | `FE80::/10`                |                            |
|              |            | `e0/2`       | `100.67.0.8`   | `100.67.0.8/31`   | `2001:AAAA:BB05:108::8:E2/64`  | `2001:AAAA:BB05:108::/64`  |                            |
|              |            |              |                |                   | `FE80::25:E2`                  | `FE80::/10`                |                            |
|              |            | `e0/3`       | `100.67.0.10`  | `100.67.0.10/31`  | `2001:AAAA:BB05:110::10:E3/6`  | `2001:AAAA:BB05:110::/64`  |                            |
|              |            |              |                |                   | `FE80::25:E3`                  | `FE80::/10`                |                            |
|              |            | `e0/1`       | `100.67.0.12`  | `100.67.0.12/31`  | `2001:AAAA:BB05:112::12:E1/64` | `2001:AAAA:BB05:112::/64`  |                            |
|              |            |              |                |                   | FE80::25:E1                    | FE80::/10                  |                            |

#### 2. Настройка сетевого оборудования.

#### 2.1 Настройка маршрута по умолчанию для офиса Лабытнанги.

В данном разделе показано как настроить маршрут по умолчанию. Файл с настройкой маршрута по умолчанию находится в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/configs) в файле **R27_int.txt**.

------

```
conf terminal
!
 ip route 0.0.0.0 0.0.0.0 10.5.0.12 name R25_e_0_1
 ipv6 route ::/0 2001:AAAA:BB05:112::12:E1 name R25_e_0_1
!
```

------

#### b. Настройка маршрута по умолчанию для сетей офиса Чокурдах на роутерах Триады.

Выбрала для проверки направления трасс маршрутов маршрутизатор R27 в Лабытнангах. Для того, чтобы проверить трассировку до данного маршрутизатора необходимо на роутерах R25 и R26 прописать статические маршруты. Файлы с настройками на устройствах находятся в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/configs) в файлах **--_int.txt**. Первые символы в названии файлов соответствуют именам сетевых устройств.

------

```
На маршрутизаторе R25

conf terminal
!
 ip route 100.3.0.0 255.255.254.0 10.5.0.11 name To_Net_Chokurdah
 ip route 100.3.0.0 255.255.254.0 10.5.0.9 150
!
 ipv6 route 2001:AAAA:BB03::/48 2001:AAAA:BB05:108::9:E2 150 name To_Net_Chokurdah
 ipv6 route 2001:AAAA:BB03::/48 2001:AAAA:BB05:110::11:E1 name To_Net_Chokurdah
!

На маршрутизаторе R26

conf terminal
!
 ip route 10.5.0.12 255.255.255.254 10.5.0.8 name To_Labytnangi
 ip route 100.3.0.0 255.255.254.0 10.5.0.17 name To_Chokurdah
!
 ipv6 route 2001:AAAA:BB03::/48 2001:AAAA:BB05:116::17:E0 name To_Chokurdah
 ipv6 route 2001:AAAA:BB05:112::/64 2001:AAAA:BB05:108::8:E2 name To_Labytnangi
!
```

------

#### c. Распределение трафика между двумя линками с провайдером.

Для того, что бы распределить трафик между двумя линками необходимо создать access-list с разрешениями для подсетей и route-map с правилами. Настройки ACL и PBR находятся в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/configs) в файлt **R28_int.txt**.

------

```
Создала ACL:

!
conf terminal
!
 ip access-list extended ACL-VL11
  permit ip 100.3.0.0 0.0.0.255 any
  exit
 ip access-list extended ACL-VL12
  permit ip 100.3.1.0 0.0.0.255 any
  exit
 ipv6 access-list ACL-VL11-IPV6
  permit ipv6 2001:AAAA:BB03:1011::/64 any
  exit
 ipv6 access-list ACL-VL12-IPV6
  permit ipv6 2001:AAAA:BB03:1012::/64 any
  exit
exit
!

Создала маршрутные карты route-map:

!
conf terminal
route-map PBR_TRAFFIC permit 10
 match ip address ACL-VL11
 set ip next-hop verify-availability 10.5.0.16 10 track 1
 set ip next-hop verify-availability 10.5.0.10 20 track 2
 exit
exit
!
conf terminal
route-map PBR_TRAFFIC permit 20
 match ip address ACL-VL12
 set ip next-hop verify-availability 10.5.0.10 10 track 2
 set ip next-hop verify-availability 10.5.0.16 20 track 1
 exit
exit
!
conf terminal
route-map PBR_TRAFFIC permit 30
 match ipv6 address ACL-VL11-IPV6
 set ipv6 next-hop 2001:AAAA:BB05:116::16:E1/64 10 track 3
 set ipv6 next-hop 2001:AAAA:BB05:110::10:E3/64 20 track 4
 exit
exit
!
conf terminal
route-map PBR_TRAFFIC permit 40
 match ipv6 address ACL-VL12-IPV6
 set ipv6 next-hop 2001:AAAA:BB05:110::10:E3/64 10 track 4
 set ipv6 next-hop 2001:AAAA:BB05:116::16:E1/64 20 track 3
 exit
exit
!
```

------

Для того, что бы правила route-map начали работать необходимо настроить политики на интерфейсе.

------

```
conf terminal
!
interface Ethernet0/2.11
 ip policy route-map PBR_TRAFFIC
 exit
interface Ethernet0/2.12
 ip policy route-map PBR_TRAFFIC
 exit 
!
```

------

#### d. Настройка отслеживания линка через технологию IP SLA.

На данном этапе лабораторной работы наобходимо настроить отслеживание линков с помощью технологии IP SLA. Для этой задачи использовала протокол ICMP. Пинги запускаются каждые 5 секунд с маршрутизатора R28 на ip адреса интерфейсов маршрутизаторов R25 и R26 (настройки ниже).

------

```
conf terminal
!
ip sla 1
 icmp-echo 10.5.0.16 source-ip 10.5.0.17
 frequency 5
exit
ip sla schedule 1 life forever start-time now
!
ip sla 2
 icmp-echo 10.5.0.10 source-ip 10.5.0.11
 frequency 5
exit
ip sla schedule 2 life forever start-time now
!
ip sla 3
 icmp-echo 2001:AAAA:BB05:116::16:E1 source-ip 2001:AAAA:BB05:116::17:E0
 frequency 5
exit
ip sla schedule 3 life forever start-time now
!
ip sla 4
 icmp-echo 2001:AAAA:BB05:116::10:E3 source-ip 2001:AAAA:BB05:116::11:E1
 frequency 5
exit
ip sla schedule 4 life forever start-time now
!
```

------

Для связи route-map и ip sla используются track.

------

```
!
track 1 ip sla 1 reachability
exit
track 2 ip sla 2 reachability
exit
track 3 ip sla 3 reachability
exit
track 4 ip sla 4 reachability
exit
!
```

------

Настройки оборудования выполнены. Следующий шаг - проверка выполненных работ.

#### ***III. Проверка работоспособности системы.\***

На персональных компьютерах адресация IPv4 раздается DHCP-сервером. DHCP-сервер поднят на роутере R28, Конфигурация находится в файле ***"R28_int.txt"\***, который расположен в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/configs).

Для проверки прохождения трафика, как конечный хост, использую ip-адрес роутера R27 - 10.5.0.13. С помощью команды ***trace 10.5.0.13 -P 1\*** проверила трассировку до R27 c компьютеров VPC30 (рис.2) и VPC31 (рис.3).

Рисунок 2.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Trace_VPC30_1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Trace_VPC30_1.png)

Рисунок 3.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Trace_VPC31_1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Trace_VPC31_1.png)

Как видно из вышеприведенных рисунков, трафик до R27 разделился на два маршрута.

Теперь проверю будет ли доходить трафик, если будет недоступен интерфейс e0/3 на роутере R25 ip адрес которого 10.5.0.10. Для доставерности запустила команду ping 10.5.0.10 на роутере R28. Убедилась, что интерфейс не доступен.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Ping_R25e0_3_down.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Ping_R25e0_3_down.png)

Далее повторила команду ***trace 10.5.0.13 -P 1\*** и проверила трассировку до R27 c компьютеров VPC30 (рис.4) и VPC31 (рис.5).

Рисунок 4.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Trace_VPC30_R25e0_3_down.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Trace_VPC30_R25e0_3_down.png)

Рисунок 5.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Trace_VPC31_R25e0_3_down.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Trace_VPC31_R25e0_3_down.png)

Вижу, что весь трафик пошел через интерфейс e0/1 маршрутизатора R26. Отслеживание линка через технологию IP SLA работает.

Аналогичные действия провожу с интерфейсом e0/1 на маршрутизаторе R26, убедилась, что интерфейс не доступен.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Ping_R26e0_1_down.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Ping_R26e0_1_down.png)

Запустила трассировку с компьютеров VPC30 (рис.6) и VPC31 (рис.7).

Рисунок 6.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Trace_VPC30_R26e0_1_down.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Trace_VPC30_R26e0_1_down.png)

Рисунок 7.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Trace_VPC31_R26e0_1_down.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Trace_VPC31_R26e0_1_down.png)

Наблюдаем, что весь трафик пошел через интерфейс e0/3 маршрутизатора R25.

**Из вышеприведенных примеров вижу, что отслеживание линка через технологию IP SLA работает!**

Адресация IPv6 устанавливается на ПК способом автоматической настройки адреса без отслеживания состояния SLAAC, который позволяет устройству получить свой префикс, длину префикса и адрес шлюза по умолчанию от маршрутизатора IPv6 без помощи DHCPv6-сервера.

Для проверки прохождения трафика, как конечный хост, так же использую ipv6-адрес роутера R27 - 2001:AAAA:BB05:112::13:E0. С помощью команды ***trace 2001:AAAA:BB05:112::13:E0\*** проверила трассировку до R27 c компьютеров VPC30 (рис.8) и VPC31 (рис.9).

Рисунок 8.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Trace_VPC30_IPV6_1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Trace_VPC30_IPV6_1.png)

Рисунок 9.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Trace_VPC31_IPV6_1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Trace_VPC31_IPV6_1.png)

В данном случае распределения трафика нет, так как IOS оборудования в лабораторном стенде не поддерживает данный функционал. Конфигурация распределения трафика находится в файле ***"R28_int.txt"\***, который расположен в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/configs).

Теперь проверю будет ли доходить трафик, если будет недоступен интерфейс e0/1 на роутере R26 ipv6 адрес которого 2001:AAAA:BB05:116::16:E1. Так же С помощью команды ***trace 2001:AAAA:BB05:112::13:E0\*** проверила трассировку до R27 c компьютеров VPC30 (рис.10) и VPC31 (рис.11).

Рисунок 10.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Trace_VPC30_R26e0_IPV6_1_down.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Trace_VPC30_R26e0_IPV6_1_down.png)

Рисунок 11.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Trace_VPC31_R26e0_IPV6_1_down.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Trace_VPC31_R26e0_IPV6_1_down.png)

Трафик изменил свое направление.

**Из вышеприведенных примеров вижу, что отслеживание линка через технологию IP SLA для IPV6 так же работает!**

#### ***IV. Итоговая схема.\***

На рис.5 размещены используемые сети, IPv4 и IPv6 адреса маршрутизаторов, коммутаторов и персональных компьютеров а так же испльзуемые VLAN.

Рисунок 5.

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab005_PBR/Shema_lab5.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab005_PBR/Shema_lab5.png)