## **Настройка DHCPv6**

### Схема подключения:

Рис. 1

![](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/topology.png)

Рис. 2

![](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/%D1%81onnection_diagram.png)

### Таблица адресации:

Табица 1

| Device | Interface | IPv6 Address          |
| ------ | --------- | --------------------- |
| R1     | G0/0/0    | 2001:db8:acad:2::1/64 |
|        |           | fe80::1               |
|        | G0/0/1    | 2001:db8:acad:1::1/64 |
|        |           | fe80::1               |
| R2     | G0/0/0    | 2001:db8:acad:2::2/64 |
|        |           | fe80::2               |
|        | G0/0/1    | 2001:db8:acad:3::1/64 |
|        |           | fe80::1               |
| PC-A   | NIC       | DHCP                  |
| PC-B   | NIC       | DHCP                  |

### Задание:

1. Построить сеть и настроить основные параметры устройств.
2. Проверить назначение адреса SLAAC от R1.
3. Настроить и проверить DHCPv6-сервер без сохранения состояния на маршрутизаторе R1.
4. Настроить и проверить сервер DHCPv6 с отслеживанием состояния на маршрутизаторе R1.
5. Настроить и проверить DHCPv6 Relay на R2.

### Ход выполнения:

Для выполнения лабораторной работы использовался эмулятор EVE-NG, терминальный клиент PuTTY.

#### ***I. Создание сети и настройка основных параметров устройств.\***

#### Сбор схемы:

1. Подключила устройства, как показано на рисунке 1.

#### Настройка базовых параметров коммутаторов и маршрутизаторов:

Базовые настройки коммутаторов и маршрутизаторов находятся в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/configs) в файлах **OP_S1.txt**, **OP_S2.txt**, **OP_R1.txt**, **OP_R2.txt** соответственно. Для правильного ввода последовательности параметров команд на устройствах использую вопросительный знак (?). На коммутаторах выключила все неиспользованные порты (**S1.txt** и **S2.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/configs)). А на маршрутизаторах включила маршрутизацию IPv6 (**R1.txt** и **R2.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/configs)).

#### Настройка интерфейсов и маршрутизации для обоих маршрутизаторов:

Сначала настроила интерфейсы E0/0 и E0/1 на маршрутизаторах R1 и R2 с адресами IPv6, указанными в таблице выше. Далее настроила маршрут по умолчанию на каждом маршрутизаторе, указывающий на IP-адрес E0/0 на другом маршрутизаторе.

Настройки описаны в конфигурационных файлах **R1.txt** и **R2.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/configs).

Для проверки работы маршрутизации выполнила команду ping с маршрутизатора R1 на адрес E0/1 R2 (результат на рис.2).

Рис.2

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/R1_ping_R2_E0_1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/R1_ping_R2_E0_1.png)

#### ***II. Проверка назначения адреса методом SLAAC от маршрутизатора R1.\***

Для проверки автоматической раздачи IPv6-адресов запустила компьютер. Сетевая карта настроена на автоматическое получение адресов. В командной строке ввела команду ***ipconfig\***. Вывод команды на рисунке 3.

Рис.3

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_IPv6.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_IPv6.png)

*Вопрос:* Откуда взялась часть адреса, содержащая идентификатор хоста?

*Ответ:* Если не используется DHCP сервер, то это часть адреса генерируется автоматически (генератор случайных чисел).

#### ***III. Настройка и проверка сервера DHCPv6 на маршрутизаторе R1.\***

В этой части лабораторной работы настроила DHCP-сервер без отслеживания состояния на маршрутизаторе R1. Нужно сделать так, чтобы PC-A получил информацию о DNS-сервере и домене.

#### Изучение более подробной конфигурации PC-A.

Для получения более расширенной информации о настройках PC выполнила команду ***ipconfig /all\***. Вывод на рис.4.

Рис.4

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_ipcongig_all_1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_ipcongig_all_1.png)

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_ipcongig_all_2.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_ipcongig_all_2.png)

Из вывода видим, что нет суффикса первичного DNS. Также видим, что предоставленные адреса DNS-серверов являются «site local anycast», а не unicast адресами, как можно было бы ожидать.

#### Настройка R1 для предоставления DHCPv6 без сохранения состояния для PC-A.

- На маршрутизаторе R1 создала пул DHCP IPv6 с именем R1-STATELESS, в жтом пуле назначила адрес DNS-сервера и имя домена.
- Настроила интерфейс E0/1 на роутере R1 для предоставления флага конфигурации OTHER для локальной сети маршрутизатора R1, а так же указала созданный пул как DHCP ресурс для этого интерфейса.

Создание пула и настройка интерфейса описаны в конфигурационном файле **R1.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/configs).

- После этого, перезагружаем компьютер и повторно вводим команду ***ipconfig /all\***. Вывод на рис.5. в котором видим, что появился Connection-specific DNS Suffix - STATELESS.com

Рис.5

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_ipcongig_all_1_DHCP.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_ipcongig_all_1_DHCP.png)

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_ipcongig_all_2_DHCP.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_ipcongig_all_2_DHCP.png)

- Для проверки подключения, отправила эхо-запрос на IP-адрес интерфейса E0/1 R2 - 2001:db8:acad:3::1/64 (рисунок 6).

Рис.6

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_ping_R2_E0_1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-A_ping_R2_E0_1.png)

#### ***IV. Настройка DHCPv6-сервера с отслеживанием состояния на R1.\***

В этой части работы я создала пул DHCPv6 на маршрутизаторе R1 для сети 2001:db8:acad:3:aaaa::/80. Это предоставит адреса локальной сети, подключенной к интерфейсу E0/1 на R2. В составе пула установила DNS-сервер - 2001:db8:acad::254 и установила имя домена - STATEFUL.com. Далее назначила созданный пул на интерфейс E0/0.

Создание пула и настройка интерфейса описаны в конфигурационном файле **R1.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/configs).

#### ***IV. Настройка и проверка ретранслятора DHCPv6 на R2.\***

Настройка ретрансляции на R2 позволит получать адрем IPv6 на компьютере PC-B.

#### Проверка на компьютере PC-B генерируемого им адреса SLAAC.

- Для проверки ввела команду ***ipconfig /all\***. Вывод на рис.7. в котором видим сгенерированный автоматически кусок адреса и префикс 2001:db8:acad:3 .

Рис.7

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-B_ipcongig_all_1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-B_ipcongig_all_1.png)

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-B_ipcongig_all_2.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-B_ipcongig_all_2.png)

#### Настройка R2 в качестве агента ретрансляции DHCPv6 для LAN на E0/1.

На интерфейсе E0/1 маршрутизатора R2 ввела команду ***ipv6 dhcp relay\*** с адресом интерфейса E0/0 на R1, а так же задаела режим сервера с отслеживанием состояния командой ***managed-config-flag\***. Настройка интерфейса описана в конфигурационном файле **R2.txt** в папке [configs](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/configs).

#### Получение IPv6-адреса от DHCPv6 на PC-B.

- Перезагрузила компьюте PC-B.
- В командной строке ввела команду ***ipconfig /all\***. Вывод на риунке 8. Теперь видим адрес DNS-сервера и DNS Suffix.

Рис.8

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-B_ipcongig_all_1_DHCP.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-B_ipcongig_all_1_DHCP.png)

![](PC-B_ipcongig_all_2_DHCP.png

- Последним шагом проверяю возможность подключения, отправив эхо-запрос на IP-адрес интерфейса E0/1 маршрутизатора R1. Результат на рис.9

Рис.9

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-B_ping_R1_E0_1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-B_ping_R1_E0_1.png)