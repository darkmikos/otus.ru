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

Для выполнения лабораторной работы использовал эмулятор Cisco Packet Tracer 8.0

### Сборка схемы лабороторной работы

Собрал схему 

[Рисунок 1]: https://github.com/darkmikos/otus.ru/blob/master/lab02/topology.png

 в Cisco Packet Tracer 

[Рисунок 2]: https://github.com/darkmikos/otus.ru/blob/master/lab02/connection_diagram_pt.png

