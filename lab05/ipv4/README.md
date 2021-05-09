## **Lab - Implement DHCPv4**

### Схема подключения:

Рис. 1

![Topology](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/topology.png)

Рис. 2

![Connection diagram](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/%D1%81onnection_diagram.png)

### Таблица адресации:

Табица 1

| Device | Interface   | IP Address   | Subnet Mask     |              |
| ------ | ----------- | ------------ | --------------- | ------------ |
| R1     | G0/0/0      | 10.0.0.1     | 255.255.255.252 | N/A          |
|        | G0/0/1      | N/A          | N/A             |              |
|        | G0/0/1.100  | 192.168.1.1  | 255.255.255.192 |              |
|        | G0/0/1.200  | 192.168.1.65 | 255.255.255.224 |              |
|        | G0/0/1.1000 | N/A          | N/A             |              |
| R2     | G0/0/0      | 10.0.0.2     | 255.255.255.252 | N/A          |
|        | G0/0/1      | 192.168.1.97 | 255.255.255.240 |              |
| S1     | VLAN 200    | 192.168.1.2  | 255.255.255.192 | 192.168.1.1  |
| S2     | VLAN 1      | 192.168.1.98 | 255.255.255.240 | 192.168.1.97 |
| PC-A   | NIC         | DHCP         | DHCP            | DHCP         |
| PC-B   | NIC         | DHCP         | DHCP            | DHCP         |

### VLAN Table

Табица 2

| VLAN | Name        | Interface Assigned          |
| ---- | ----------- | --------------------------- |
| 1    | N/A         | S2: F0/18                   |
| 100  | Clients     | S1: F0/6                    |
| 200  | Management  | S1: VLAN 200                |
| 999  | Parking_Lot | S1: F0/1-4, F0/7-24, G0/1-2 |
| 1000 | Native      | N/A                         |



### Задание:

Необходимо настроить оборудование и организовать раздачу ip-адресов клиентам.

1. Собрать сеть и настроить основные параметры устройства.
2. Настроить и проверить два DHCPv4 сервера на роутере R1.
3. Настроить и проверить DHCP-трансляцию на роутере R2.

### Ход выполнения:

Для выполнения лабораторной работы использовал эмулятор Cisco Packet Tracer 8.0.

#### I. Создание сети и настройка основных параметров устройств.

#### Сбор схемы:

1. Подключил устройства, как показано на [рис. 1](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/topology.png) и [рис. 2](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/%D1%81onnection_diagram.png)

#### Создание схемы адресации:

По условию лабораторной работы мне выделена сеть 192.168.1.0 с маской 255.255.255.0 (24) для организации адресных пространств. 192.168.1.0./24 разделил на три подсети: подсеть "А" - 192.168.1.0/26, подсеть "В" - 192.168.1.64/27, подсеть "С" - 192.168.1.96/28.

- Записал первый IP-адрес подсети "А" (192.168.1.1) в таблицу адресации (Таблица 1) для R1 G0/0/1.100. Из этой же подсети записала второй IP-адрес (192.168.1.2) в таблицу адресов для S1 VLAN 200 и указала соответствующий шлюз по умолчанию.
- Следующим этапом записал первый IP-адрес подсети "В" в таблицу адресации для R1 G0/0/1.200. Второй IP-адрес в таблице адресов - для S1 VLAN 1 и указала соответствующий шлюз по умолчанию.
- Далее добавил в таблицу для R2 G0/0/1 первый IP-адрес из подсети "С".

#### Настройка базовых параметров коммутаторов и маршрутизаторов:

Настройки коммутаторов и маршрутизаторов находятся в файлах  [R1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/R1.cfg), [R2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/R2.cfg), [S1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/S1.cfg), [S2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/S2.cfg) соответственно. Для правильного ввода последовательности параметров команд на устройствах использую вопросительный знак (?).

```
conf t
!
hostname *
no ip domain-lookup
enable secret class
!
line console 0
password cisco
login
logging synchronous
exit
!
line vty 0 15
password cisco
login
exit
!
service password-encryption
!
banner motd > Attention! Unauthorized entry is prohibited. Contacts email: ng.vlasov@ya.ru
>
!
exit
!
clock set 16:44 9 May 2021
!
write
```



#### Настройка маршрутизации между VLAN на маршрутизаторе R1.

Включил интерфейс G0/0/1 на маршрутизаторе. Настроил субинтерфейсы для каждой VLAN в соответствии с таблицей 1. Включил на субинтерфейсах инкапсуляцию 802.1Q и назначил следующий (после шлюза по-умолчпнию) IP-адрес. Для G0/0/1.1000 native VLAN IP-адрес не назначал. Добавил описание на субинтерфейсы  в соответствии с таблицей 2. Проверил, что они в состоянии "UP". Конфигурационный файл [R1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/R1.cfg).

```
conf t
!
interface g0/0/1
 no shutdown
 exit
!
interface g0/0/1.100
 description Clients
 encapsulation dot1Q 100
 ip address 192.168.1.1 255.255.255.192
 exit
!
interface g0/0/1.200
 description Management
 encapsulation dot1Q 200
 ip address 192.168.1.65 255.255.255.224
 exit
!
interface g0/0/1.1000
 description Nativ
 encapsulation dot1Q 1000 native
 no ip address
 exit
exit
!
```

#### Настройка G0/0/1 на R2. Настройка интерфейсов G0/0/0 и статической маршрутизации для обоих маршрутизаторов.

Настроила интерфейс G0/0/0 для R1 на основе приведенной выше таблицы IP-адресации.

```
conf t
!
interface g0/0/0
 no shutdown
 ip address 10.0.0.1 255.255.255.252
 exit
exit
!
```

Настроил маршрут по умолчанию на R1, указывающий на IP-адрес G0/0/0 на R2.

```
conf t
 ip route 0.0.0.0 0.0.0.0 10.0.0.2
 exit
```

Настроил G0/0/1 на R2 с первым IP-адресом из подсети "С". 

```
conf t
!
interface g0/0/1
 no shutdown
 ip address 192.168.1.97 255.255.255.240
 exit
exit
!
```

Настроила интерфейс G0/0/0 для R2 на основе приведенной выше таблицы IP-адресации.

```
conf t
!
interface g0/0/0
 no shutdown
 ip address 10.0.0.2 255.255.255.252
 exit
exit
!
```

 Настроил маршрут по умолчанию на R2, указывающий на IP-адрес G0/0/0 на R1. 

```
conf t
 ip route 0.0.0.0 0.0.0.0 10.0.0.1
 exit
```

Конфигурационные файлы [R1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/R1.cfg) и [R2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/R2.cfg).

Провел тест подтверждающий работу статической маршрутизации:

```
R1#ping 192.168.1.97

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms
```

#### Создание VLAN на S1.

- Создал необходимые сети VLAN на коммутаторе S1 из таблицы VLAN (Таблица 2). Настроил интерфейс управления на S1 (VLAN 200), использовал IP-адрес 192.168.1.2 и установил шлюз по умолчанию на S1 - 192.168.1.1. 

  ```
  conf t
  !
  vlan 100
  name Clients
  vlan 200
  name Management
  vlan 999
  name Parking_Lot
  vlan 1000
  name Native
  exit
  !
  interface vlan 200
   ip address 192.168.1.2 255.255.255.192
   no shutdown
   exit
  ip default-gateway 192.168.1.1
  exit
  ```

- Настроил интерфейс управления на S2 (VLAN 1), используя IP-адрес 192.168.1.98. Установила шлюз по умолчанию 192.168.1.97.

  ```
  conf t
   interface vlan 1
   ip address 192.168.1.98 255.255.255.240
   no shutdown
   exit
  !
  ip default-gateway 192.168.1.97
  !
  exit
  ```

- По условию лабораторной работы отправил все неиспользованные порты на S1 в VLAN Parking_Lot, настроил их для статического режима доступа и деактивировал.

  ```
  conf t
  int range fa0/1-4, fa0/7-24, gi0/1-2
   switchport mode access
   switchport access vlan 999
   shutdown
   exit
  exit
  ```

- На коммутаторе S2 деактивировала все не используемые порты.

  ```
  conf t
  int range fa0/1-4, fa0/6-17, fa0/19-24, gi0/1-2
   switchport mode access
   switchport access vlan 999
   shutdown
   exit
  exit
  ```

Для групповой настройки портов пользовался командой interface range. Конфигурационные файлы [S1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/S1.cfg) и [S2cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/S2.cfg).

#### Назначение VLAN на интерфейсы.

Назначил интерфейсам, которые выделены для связи c компьютерами (на S1 - F0/6, на S2 - F0/18), соответствующий VLAN и перевел в статический режим (static access mode).

S1

```
conf t
interface fa0/6
 switchport mode access
 switchport access vlan 100
 no shutdown
 exit
exit
```

S2

```
conf t
interface fa0/18
 switchport mode access
 switchport access vlan 1
 no shutdown
 exit
exit
```

*Вопрос:* Почему интерфейс fa0/18 указан в VLAN 1?

*Ответ:* Так как интерфейс fa0/18 не был сконфигурирован – он по умолчанию остается в VLAN 1.

#### Настройка интерфейса fa0/5 на S1 как транковый 802.1Q.

Настроил интерфейс fa0/5 в режиме транк. Указал native VLAN в значение 1000. Включил прохождение через этот порт VLAN-ам 100, 200 и 1000. 

```
conf t
interface fa0/5
 switchport trunk allowed vlan 100,200,1000
 switchport trunk native vlan 1000
 switchport mode trunk
 exit
exit
```

Конфигурационный файл [S1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/S1.cfg).

Для проверки правильности настройки использовала команду **show interface trunk**.

```
S1#show interfaces trunk 
Port        Mode         Encapsulation  Status        Native vlan
Fa0/5       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/5       100,200,1000

Port        Vlans allowed and active in management domain
Fa0/5       100,200,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/5       100,200,1000
```

*Вопрос:* Какой IP-адрес был бы у ПК, если бы он был подключен к сети с помощью DHCP?

*Ответ:* Если не настроены исключения, то первый используемый адрес сети.

#### II. Настройка и проверка двух серверов DHCPv4 на маршрутизаторе R1.

#### Настройка R1 с пулами DHCPv4 для двух поддерживаемых подсетей.

Для того, что бы сервер DHCP выдавал IP-адреса предсказуемо, необходимо настроить исключения. Из каждого пула исключил первые пять адресов. После этого можно создавать пулы с испольованием уникальных имен. Настроил доменное имя ***ccna-lab.com\***, настроил шлюзы по умолчанию для каждого пула. Выставила срок аренды на 2 дня 12 часов 30 минут. Для второго пула использовала имя ***R2_Client_LAN\*** и использовала доменное имя ***ccna-lab.com\*** и срок аренды, такие же как и в первом пуле. Конфигурационный файл [R1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/R1.cfg).

```
conf t
 ip dhcp excluded-address 192.168.1.1 192.168.1.5
 ip dhcp excluded-address 192.168.1.97 192.168.1.101
 ip dhcp pool CLIENTS_POOL
  network 192.168.1.0 255.255.255.192
  default-router 192.168.1.1
  lease 2 12 30
  domain-name ccna-lab.com
  exit
 ip dhcp pool R2_Client_LAN
  network 192.168.1.96 255.255.255.240
  default-router 192.168.1.97
  lease 2 12 30
  domain-name ccna-lab.com
  exit
 exit
```

#### Проверка конфигурации DHCPv4-сервера.

- Выполнив команду **show ip dhcp pool** мы видим, что у нас настроено два пула, видим адреса пулов; видим их адресацию; видим, сколько адресов в пуле.

  ```
  R1#show ip dhcp pool 
  
  Pool CLIENTS_POOL :
   Utilization mark (high/low)    : 100 / 0
   Subnet size (first/next)       : 0 / 0 
   Total addresses                : 62
   Leased addresses               : 0
   Excluded addresses             : 2
   Pending event                  : none
  
   1 subnet is currently in the pool
   Current index        IP address range                    Leased/Excluded/Total
   192.168.1.1          192.168.1.1      - 192.168.1.62      0    / 2     / 62
  
  Pool R2_Client_LAN :
   Utilization mark (high/low)    : 100 / 0
   Subnet size (first/next)       : 0 / 0 
   Total addresses                : 14
   Leased addresses               : 0
   Excluded addresses             : 2
   Pending event                  : none
  
   1 subnet is currently in the pool
   Current index        IP address range                    Leased/Excluded/Total
   192.168.1.97         192.168.1.97     - 192.168.1.110     0    / 2     / 14
  ```

- Выполнив следующую команду **show ip dhcp binding** видим, что пока нет выданных адресов.

  ```
  R1#show ip dhcp b
  R1#show ip dhcp binding 
  IP address       Client-ID/              Lease expiration        Type
                   Hardware address
  ```

- Для проверки других сообщение DHCP выполнила команду **show ip dhcp server statistics**.

  Рис. 3


![Рис. 3](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/show_ip_dhcp_server_statistics.png)

#### Получение IP-адреса от DHCP на PC-A.

- На компьютере PC-A в настройках сетевой карты установил параметр "Получать данные автоматически". Затем открыла cmd и ввел команды ipconfig и ipconfig /renew для получения новой информации.

  ```
  Packet Tracer PC Command Line 1.0
  C:\>ipconfig
  
  FastEthernet0 Connection:(default port)
  
     Connection-specific DNS Suffix..: ccna-lab.com
     Link-local IPv6 Address.........: FE80::2E0:A3FF:FE16:59AA
     IPv6 Address....................: ::
     IPv4 Address....................: 192.168.1.6
     Subnet Mask.....................: 255.255.255.192
     Default Gateway.................: ::
                                       192.168.1.1
  
  Bluetooth Connection:
  
     Connection-specific DNS Suffix..: ccna-lab.com
     Link-local IPv6 Address.........: ::
     IPv6 Address....................: ::
     IPv4 Address....................: 0.0.0.0
     Subnet Mask.....................: 0.0.0.0
     Default Gateway.................: ::
                                       0.0.0.0
  
  C:\>ipconfig /renew
  
     IP Address......................: 192.168.1.6
     Subnet Mask.....................: 255.255.255.192
     Default Gateway.................: 192.168.1.1
     DNS Server......................: 0.0.0.0
  
  C:\>
  ```

- Проверил возможность подключения, отправив эхо-запрос на IP-адрес интерфейса G0/0/1 маршрутизатора R1

  ```
  C:\>ping 192.168.1.1
  
  Pinging 192.168.1.1 with 32 bytes of data:
  
  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
  Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
  
  Ping statistics for 192.168.1.1:
      Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
  Approximate round trip times in milli-seconds:
      Minimum = 0ms, Maximum = 0ms, Average = 0ms
  ```

#### III. Настройка и проверка DHCP-ретрансляции на R2.

Необходимо настроить R2 для ретрансляции запросов DHCP из локальной сети по интерфейсу G0/0/1 на сервер DHCP (R1).

#### Настройка R2 в качестве агента ретрансляции DHCP для локальной сети на 0/1.

На интерфейсе G0/0/1 настроила ip helper-address, указав IP-адрес G0/0/0 маршрутизатора R1. 

```
conf t
interface GigabitEthernet0/0/1
 ip helper-address 10.0.0.1
```

#### Получение IP-адреса от DHCP на PC-B.

- На компьютере PC-B в настройках сетевой карты установил параметр "Получать данные автоматически". Затем открыла cmd и ввел команды ipconfig и ipconfig /renew для получения новой информации.

  ```
  C:\>ipconfig
  
  FastEthernet0 Connection:(default port)
  
     Connection-specific DNS Suffix..: ccna-lab.com
     Link-local IPv6 Address.........: FE80::290:21FF:FE87:B91A
     IPv6 Address....................: ::
     IPv4 Address....................: 192.168.1.102
     Subnet Mask.....................: 255.255.255.240
     Default Gateway.................: ::
                                       192.168.1.97
  
  Bluetooth Connection:
  
     Connection-specific DNS Suffix..: ccna-lab.com
     Link-local IPv6 Address.........: ::
     IPv6 Address....................: ::
     IPv4 Address....................: 0.0.0.0
     Subnet Mask.....................: 0.0.0.0
     Default Gateway.................: ::
                                       0.0.0.0
  
  C:\>ipconfig /renew
  
     IP Address......................: 192.168.1.102
     Subnet Mask.....................: 255.255.255.240
     Default Gateway.................: 192.168.1.97
     DNS Server......................: 0.0.0.0
  ```

  

- Проверил доступность, отправив эхо-запрос на IP-адрес интерфейса G0/0/1 маршрутизатора R1.

```
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time=1ms TTL=254

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

```

- После того, как я увидел, что компьютер получил адрес и связность в сети есть, повторно запустила на маршрутизаторе R1 команду show ip dhcp binding. Картина изменилась, в таблице появились IP-адреса выданные компьютерам.

```
R1#show ip dhcp binding 
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      00E0.A316.59AA           --                     Automatic
192.168.1.102    0090.2187.B91A           --                     Automatic
```

- С помощью команды show ip dhcp server statistics проверяем статистику на R1 (Рис.4) и R2 (Рис.5).

Рис. 4

![](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/show_ip_dhcp_server_statistics_end.png)

Рис. 5

![](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/R2_show_ip_dhcp_server_statistics_end.png)

Как видно из выводов, на втором роутере таблица статистики пуста, так как он является всего лишь ретранслятором запросов.

