# **Lab - Configure Router-on-a-Stick Inter-VLAN Routing**



### Topology

![Топология](https://github.com/darkmikos/otus.ru/blob/master/lab02/topology.png)

Рисунок 1

### Connection diagram



![connection diagram](https://github.com/darkmikos/otus.ru/blob/master/lab02/connection_diagram_pt.png)

Рисунок 2

### Таблица ip адресации:

Табица 1

| Device | Interface | IP Address   | Subnet Mask   | Default Gateway |
| ------ | --------- | ------------ | ------------- | --------------- |
| R1     | G0/0/1.3  | 192.168.3.1  | 255.255.255.0 | N/A             |
|        | G0/0/1.4  | 192.168.4.1  | 255.255.255.0 | N/A             |
|        | G0/0/1.8  | N/A          | N/A           | N/A             |
| S1     | VLAN 3    | 192.168.3.11 | 255.255.255.0 | 192.168.3.1     |
| S2     | VLAN 3    | 192.168.3.12 | 255.255.255.0 | 192.168.3.1     |
| PC-A   | NIC       | 192.168.3.3  | 255.255.255.0 | 192.168.3.1     |
| PC-B   | NIC       | 192.168.4.3  | 255.255.255.0 | 192.168.4.1     |

### Таблица VLAN:

Табица 2

| VLAN | Name       | Interface Assigned            |
| ---- | ---------- | ----------------------------- |
| 3    | Management | S1: VLAN 3                    |
|      |            | S2: VLAN 3                    |
|      |            | S1: F0/6                      |
| 4    | Operations | S2: F0/18                     |
| 7    | ParkingLot | S1: F0/2-4, F0/7-24, G0/1-2   |
|      |            | S2: F0/2-17, F0/19-24, G0/1-2 |
| 8    | Native     | N/A                           |

### Задание:

Необходимо настроить оборудование и организовать сетевую связанность согласно условию лабораторной работы:

1. Создать сеть и настроить основные параметры устройства.
2. Создать VLAN и назначить порты коммутаторов.
3. Настроить транки 802.1Q между коммутаторами.
4. Настроить маршрутизацию между VLAN на роутере.
5. Проверить работоспособность сети.

### Ход выполнения:

Для выполнения лабораторной работы использовал эмулятор Cisco Packet Tracer 8.0.

#### Сборка схемы лабороторной работы

Собрал схему Рисунок 1 в Cisco Packet Tracer Рисунок 2.

#### Настройка основных параметров роутера:

1. Подключился к роутеру, зашел в превилигированный режим, далее в режим конфигурации.

2. Выполнил настройку роутера R1, конфиг находится в файле  [R1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab02/R1.cfg).

   ```
   conf t
   hostname R1
   no ip domain-lookup
   enable secret class
   line console 0
   password cisco
   login
   exit
   line vty 0 15
   password cisco
   login
   exit
   service password-encryption
   banner motd > Attention! Unauthorized entry is prohibited. Contacts email: ng.vlasov@ya.ru>
   exit
   clock set 12:05:00 13 April 2021
   write
   ```

#### Настройка основных параметров для каждого коммутатора:

1. Подключился к коммутаторам, зашел в превилигированный режим, далее в режим конфигурации.

2. Выполнил настройку коммутаторов S1 и S2. Конфиги соответственно [S1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab02/S1.cfg) и [S2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab02/S2.cfg).

3. Сохранил текущую конфигурацию в файл стартовой конфигурации.

   ```
   conf t
   hostname S1(S2)
   no ip domain-lookup
   enable secret class
   line console 0
   password cisco
   login
   exit
   line vty 0 15
   password cisco
   login
   exit
   service password-encryption
   banner motd > Attention! Unauthorized entry is prohibited. Contacts email: ng.vlasov@ya.ru>
   exit
   clock set 15:23:00 13 April 2021
   write
   ```

   

#### Настройка компьютеров (hosts: PC-A, PC-B):

1. Настроил IP-адрес, маску подсети и шлюз по умолчанию, согласно таблице 1.

#### Создание VLAN и назначение портов коммутаторов:

1. На коммутаторе S1, S2 создал VLAN и присвоил им имена согласно таблице 2.

   ```
   conf t
   vlan 3
   name Management
   exit
   vlan 4
   name Operations
   exit
   vlan 7
   name ParkingLot
   exit
   vlan 8
   name Native
   exit
   ```

   

2. Настроил интерфейс управления и шлюз по умолчанию на каждом коммутаторе, используя информацию об IP-адресе в таблице адресации.

   ```
   conf t
   interface vlan 3
    ip address 192.168.3.11(12) 255.255.255.0
    no shutdown
    exit
    ip default-gateway 192.168.3.1
   ```

3. Назначил все неиспользуемые порты на обоих коммутаторах в ParkingLot VLAN, настроил их для статического режима доступа (static access mode) и отключил (shutdown).

4. При конфигурации нескольких интерфейсов с одинаковыми параметрами использовал команду interface range.

   ```
   S1:
   conf t
   interface range f0/2-4
    switchport mode access
    switchport access vlan 7
    shutdown
    exit
   interface range f0/7-24
    switchport mode access
    switchport access vlan 7
    shutdown
    exit
   interface range g0/1-2
    switchport mode access
    switchport access vlan 7
    shutdown
    exit
   ```

   ```
   S2:
   conf t
   interface range f0/2-17
    switchport mode access
    switchport access vlan 7
    shutdown
    exit
   interface range f0/19-24
    switchport mode access
    switchport access vlan 7
    shutdown
    exit
    interface range g0/1-2
    switchport mode access
    switchport access vlan 7
    shutdown
    exit
   ```

5. Назначил используемым портам соответствующей VLAN (указанные в таблице VLAN) и настроил их для статического режима доступа (static access mode).

   ```
   S1:
   conf t
   interface f0/6
    switchport mode access
    switchport access vlan 3
    exit
   ```

   ```
   S2:
   conf t
   interface f0/18
    switchport mode access
    switchport access vlan 4
    exit
   ```

6. Командой show vlan brief проверил, что правильно назначил VLAN на порты.

#### Настройка магистрали (trunk) 802.1Q между коммутаторами:

1. Настроил на  S1, S2 коммутаторах интерфейсы F0/1 в режим trank (switchport mode trunk).

2. Настроил на  S1, S2 коммутаторах интерфейсы F0/1 nativ vlan (switchport trunk native vlan 8).

3. Настроил на  S1, S2 коммутаторах интерфейсы транк для пропуска VLAN 3,4,8 ( switchport trunk allowed vlan 3-4,8).

   ```
   S1:
   conf t
   interface f0/1
    switchport mode trunk
    switchport trunk allowed vlan 3-4,8
    switchport trunk native vlan 8
    exit
   ```

   ```
   S2:
   conf t
   interface f0/1
    switchport trunk allowed vlan 3,4,8
    switchport trunk native vlan 8
    switchport mode trunk
    exit
   ```

4. Проверил настройки транковых портов использовал команду show interfaces trunk.

5. На коммутаторе S1 настроила порт F0/5 так же как и F0/1.

   ```
   conf t
   interface f0/5
    switchport mode trunk
    switchport trunk allowed vlan 3-4,8
    switchport trunk native vlan 8
    exit
   ```

6. Сохранил текущую конфигурацию коммутаторах S1, S2.

7. Выполнил команду show interfaces trunk для проверки транкинга.

8. Файлы с конфигурацией интерфейсов коммутаторов (S1.cfg и S2.cfg) находятся в папке [lab02](https://github.com/darkmikos/otus.ru/tree/master/lab02).

#### Вопрос:

Почему F0/5 не отображается в списке транков?

#### Ответ:

Не включен интерфейс со стороны роутера.

#### Настройка маршрутизации между VLAN на роутере:

1. Включил интерфейс G0/0/1 на маршрутизаторе.

   ```
   conf t
   interface G0/0/1
    no shutdown
    exit
   ```

2. Настроил субинтерфейс для каждой VLAN, как указано в таблице IP-адресации. Все субинтерфейсы используют инкапсуляцию 802.1Q.

3. Настроил описание субинтерфейсов.

4. Для nativ vlan 8 IP-адрес не назначал.

   ```
   conf t
   interface G0/0/1.3
    description Net_3
    encapsulation dot1Q 3
    ip address 192.168.3.1 255.255.255.0
    exit
   interface G0/0/1.4
    description Net_4
    encapsulation dot1Q 4
    ip address 192.168.4.1 255.255.255.0
    exit
   interface G0/0/1.8
    description Nativ_8
    encapsulation dot1Q 8 native
    no ip address
    exit
   exit
   ```

5. Чтобы проверить работоспособность субинтерфейсов использовал команду show ip interface brief.

6. Файл с конфигурацией интерфейсов роутера (R1.cfg) находятся в папке [lab02](https://github.com/darkmikos/otus.ru/tree/master/lab02).

#### Проверка работоспособности системы:

1. С компьютера PC-A запустил команду ping на шлюз по умолчанию для этого компьютера.

2. С компьютера PC-A запустил команду ping на компьютер PC-B.

3. С компьютера PC-A запустил команду ping на коммутатор S2.

   

   Результат тестов с компьютера PC-A:

   ![ping_pc-a](https://github.com/darkmikos/otus.ru/blob/master/lab02/ping_pc-a.png)

4. С компьютера PC-B запустил команду tracert на адрес PC-A.

Результат теста с компьютера PC-B:

   ![pc-b_tracert](https://github.com/darkmikos/otus.ru/blob/master/lab02/pc-b_tracert_pv-a.png)

#### Вопрос:

Какие промежуточные IP-адреса отображаются в результатах?

#### Ответ:

В результате отображается только один промежуточный IP-адрес. IP-адрес субинтерфейса роутера, который является для PC-B шлюзом по умолчанию.

