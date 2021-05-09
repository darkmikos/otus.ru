## **Lab - Implement DHCPv4**

### Схема подключения:

Рис. 1

![Topology](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/topology.png)

Рис. 2

![Connection diagram](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/%D1%81onnection_diagram.png)

### Таблица адресации:

Табица 1

| Device | Interface | IP Address   | Subnet Mask     |              |
| ------ | --------- | ------------ | --------------- | ------------ |
| R1     | G0/0      | 192.168.1.1  | 255.255.255.252 | N/A          |
|        | G0/1      | N/A          | N/A             |              |
|        | G0/1.100  | 192.168.1.1  | 255.255.255.192 |              |
|        | G0/1.200  | 192.168.1.65 | 255.255.255.224 |              |
|        | G0/1.1000 | N/A          | N/A             |              |
| R2     | G0/0      | 10.0.0.2     | 255.255.255.252 | N/A          |
|        | G0/1      | 192.168.1.97 | 255.255.255.240 |              |
| S1     | VLAN 200  | 192.168.1.2  | 255.255.255.192 | 192.168.1.1  |
| S2     | VLAN 1    | 192.168.1.98 | 255.255.255.240 | 192.168.1.97 |
| PC-A   | NIC       | DHCP         | DHCP            | DHCP         |
| PC-B   | NIC       | DHCP         | DHCP            | DHCP         |

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

- Записал первый IP-адрес подсети "А" (192.168.1.1) в таблицу адресации (Таблица 1) для R1 G0/1.100. Из этой же подсети записала второй IP-адрес (192.168.1.2) в таблицу адресов для S1 VLAN 200 и указала соответствующий шлюз по умолчанию.
- Следующим этапом записал первый IP-адрес подсети "В" в таблицу адресации для R1 G0/1.200. Второй IP-адрес в таблице адресов - для S1 VLAN 1 и указала соответствующий шлюз по умолчанию.
- Далее добавил в таблицу для R2 G0/1 первый IP-адрес из подсети "С".

#### Настройка базовых параметров коммутаторов и маршрутизаторов:

Базовые настройки коммутаторов и маршрутизаторов Настройки коммутаторов находятся в файлах  [R1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/R1.cfg), [R2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/R2.cfg), [S1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/S1.cfg), [S2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv4/S2.cfg) соответственно. Для правильного ввода последовательности параметров команд на устройствах использую вопросительный знак (?).

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

Включил интерфейс G0/1 на маршрутизаторе. Настроил субинтерфейсы для каждой VLAN в соответствии с таблицей 1. Включил на субинтерфейсах инкапсуляцию 802.1Q и назначил следующий (после шлюза по-умолчпнию) IP-адрес. Для E0/1.1000 native VLAN IP-адрес не назначала. Повесила на субинтерфейсы описание. Проверила, что они в состоянии "UP". Все команды описаны в конфигурационном файле **R1.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/configs).

#### Настройка E0/1 на R2. Настройка интерфейсов E0/0 и статической маршрутизации для обоих маршрутизаторов.

Настроила E0/1 на R2 с первым IP-адресом из подсети "С". Настроила интерфейс E0/0 для каждого маршрутизатора на основе приведенной выше таблицы IP-адресации. Настроила маршрут по умолчанию на каждом маршрутизаторе, указывающий на IP-адрес E0/0 на другом маршрутизаторе. Все команды описаны в конфигурационных файлах **R1.txt** и **R2.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/configs). Провела тест подтверждающий работу статической маршрутизации:

```
R1# ping 192.168.1.97
Ответ: .!!!
```

#### Создание VLAN на S1.

- Создала необходимые сети VLAN на коммутаторе 1 из таблицы VLAN (Таблица 2). Настроила интерфейс управления на S1 (VLAN 200), использовала IP-адрес 192.168.1.2 и установила шлюз по умолчанию на S1 - 192.168.1.1. Активировала интерфейс.
- Настроила и активировала интерфейс управления на S2 (VLAN 1), используя IP-адрес 192.168.1.98. Установила шлюз по умолчанию 192.168.1.97.
- По условию лабораторной работы отправила все неиспользованные порты на S1 в VLAN Parking_Lot, настроила их для статического режима доступа и деактивировала.
- На коммутаторе S2 деактивировала все не используемые порты.

Для групповой настройки портов использовала команду interface range. Все настройки описаны в конфигурационных файлах **R1.txt** и **R2.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/configs).

#### Назначение VLAN на интерфейсы.

Назначила порты, которые выделены для связи c компьютерами (на S1 - E1/1, на S2 - E4/1), соответствующей VLAN и статический режим (static access mode).

*Вопрос:* Почему интерфейс E1/0 указан в VLAN 1?

*Ответ:* Так как интерфейс E1/0 не был сконфигурирован – он по-умолчанию остается в VLAN 1.

#### Настройка интерфейса E1/0 на S1 как транковый 802.1Q.

Настроила интерфейс E1/0 как транковый. Установила native VLAN в значение 1000. И разрешила прохождение через этот порт VLAN-ам 100, 200 и 1000. Все команды описаны в конфигурационном файле **S1.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/configs). Для проверки правильности настройки использовала команду ***show interface trunk\***.

Рис.2

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/S1_sh_int_trunk.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/S1_sh_int_trunk.png)

*Вопрос:* Какой IP-адрес был бы у ПК, если бы он был подключен к сети с помощью DHCP?

*Ответ:* Если не настроены исключения, то первый используемый адрес сети.

#### ***II. Настройка и проверка двух серверов DHCPv4 на маршрутизаторе R1.\***

#### Настройка R1 с пулами DHCPv4 для двух поддерживаемых подсетей.

Для того, что бы сервер DHCP выдавал IP-адреса предсказуемо, необходимо настроить исключения. Из каждого пула исключила первые пять адресов. После этого можно создавать пулы с испольованием уникальных имен. Так же настроила доменное имя ***ccna-lab.com\***, настроила шлюзы по умолчанию для каждого пула. Выставила срок аренды на 2 дня 12 часов 30 минут. Для второго пула ВРСЗм4 использовала имя ***R2_Client_LAN\*** и использовала доменное имя ***ccna-lab.com\*** и срок аренды, такие же как и в первом пуле. Все команды описаны в конфигурационном файле **R1.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/configs).

#### Проверка конфигурации DHCPv4-сервера.

- Выполнив команду ***show ip dhcp pool\*** мы видим, что у нас настроено два пула, видим адреса пулов; видим их адресацию; видим, сколько адресов в пуле. Вывод команды приводится на рисунке 3.

Рис.3

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/show_ip_dhcp_pool.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/show_ip_dhcp_pool.png)

- Выполнив следующую команду - ***show ip dhcp binding\*** видим, что пока нет выданных адресов (рисунок 4):

Рис.4

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/show_ip_dhcp_binding.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/show_ip_dhcp_binding.png)

- Для проверки других сообщение DHCP выполнила команду ***show ip dhcp server statistics\***. Результат вывода на рисунке 5.

Рис.5

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/show_ip_dhcp_server_statistics.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/show_ip_dhcp_server_statistics.png)

#### Получение IP-адреса от DHCP на PC-A.

- В настройках сети компьютера установила параметр "Получать данные автоматически". Затем открыла ***cmd\*** и ввела команду ***ipconfig /renew\*** для получения новой информации (рис.6).

Рис.6

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/PC-A_Win_IP_DHCP.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/PC-A_Win_IP_DHCP.png)

- Проверила возможность подключения, отправив эхо-запрос на IP-адрес интерфейса E0/1 маршрутизатора R1 (Рис.7)

Рис.7

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/PC-A_Win_IP_ping_E0_1_R1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/PC-A_Win_IP_ping_E0_1_R1.png)

#### ***III. Настройка и проверка DHCP-ретрансляции на R2.\***

Необходимо настроить R2 для ретрансляции запросов DHCP из локальной сети по интерфейсу E0/1 на сервер DHCP (R1).

#### Настройка R2 в качестве агента ретрансляции DHCP для локальной сети на E0/1.

На интерфейсе E0/1 настроила второй адрес командой ***ip helper-address\***, указав IP-адрес E0/0 маршрутизатора R1. Команда описана в конфигурационном файле **R2.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/configs).

#### Получение IP-адреса от DHCP на PC-B.

- В настройках сети компьютера установила параметр "Получать данные автоматически". Затем открыла ***cmd\*** и ввела команду ***ipconfig /renew\*** для получения новой информации (рис.8).

Рис.8

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/PC-B_Win_IP_DHCP.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/PC-B_Win_IP_DHCP.png)

- Проверила возможность подключения, отправив эхо-запрос на IP-адрес интерфейса E0/1 маршрутизатора R1 (Рис.9).

Рис.9

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/PC-B_Win_IP_ping_E0_1_R1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/PC-B_Win_IP_ping_E0_1_R1.png)

- После того, как я увидела, что компьютер получил адрес и связность в сети есть, повторно запустила на маршрутизаторе R1 команду ***show ip dhcp binding\***. Картина изменилась, в таблице появились IP-адреса выданные компьютерам (Рис.10).

Рис.10

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/show_ip_dhcp_binding_end.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/show_ip_dhcp_binding_end.png)

- С помощью команды ***show ip dhcp server statistics\*** проверяем статистику на R1 (Рис.11) и R2 (Рис.12).

Рис.11

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/show_ip_dhcp_server_statistics_end.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/show_ip_dhcp_server_statistics_end.png)

Рис.12

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/R2_show_ip_dhcp_server_statistics_end.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv4/R2_show_ip_dhcp_server_statistics_end.png)

Как видно из выводов, на втором роутере таблица статистики пуста, так как он является всего лишь ретранслятором запросов.