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

Для выполнения лабораторной работы использовал эмулятор Cisco Packet Tracer 8.0, эмулятор EVE-NG.

#### I. Создание сети и настройка основных параметров устройств.

#### Сбор схемы:

1. Подключила устройства, как показано на [рис. 1](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/topology.png) и [рис. 2](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/%D1%81onnection_diagram.png).

#### Настройка базовых параметров коммутаторов и маршрутизаторов:

Настройки коммутаторов и маршрутизаторов находятся в файлах [R1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/R1.cfg), [R2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/R2.cfg), [S1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/S1.cfg), [S2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/S2.cfg) соответственно. Для правильного ввода последовательности параметров команд на устройствах использую вопросительный знак (?). 

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
clock set 22:56 9 May 2021
!
write

```

На коммутаторе S1 выключил все неиспользованные интерфейсы.

```
conf t
int range fa0/1-4, fa0/7-24, gi0/1-2
 shutdown
```

На коммутаторе S2 выключил все неиспользованные интерфейсы.

```
conf t
int range fa0/1-4, fa0/6-17, fa0/19-24, gi0/1-2
 shutdown
```

На маршрутизаторах включил маршрутизацию IPv6.

```
conf t
ipv6 unicast-routing
```

Настройка интерфейсов и маршрутизации для обоих маршрутизаторов.

Настройка интерфейсы G0/0/0 и G0/0/1 на R1 с адресами IPv6, указанными в таблице выше:

```
conf t
int g0/0/0
 ipv6 address 2001:DB8:ACAD:2::1/64
 ipv6 address fe80::1 link-local
 no shutdown
 exit
!
int g0/0/1
 ipv6 address 2001:DB8:ACAD:1::1/64
 ipv6 address fe80::1 link-local
 no shutdown
 exit
```

Настройка интерфейсы G0/0/0 и G0/0/1 на R2 с адресами IPv6, указанными в таблице выше:

```
conf t
int g0/0/0
 ipv6 address 2001:DB8:ACAD:2::2/64
 ipv6 address fe80::2 link-local
 no shutdown
 exit
!
int g0/0/1
 ipv6 address 2001:DB8:ACAD:3::1/64
 ipv6 address fe80::1 link-local
 no shutdown
 exit
```

Далее настроила маршрут по умолчанию на каждом маршрутизаторе, указывающий на IP-адрес G0/0/0 на другом маршрутизаторе.

R1

```
conf t
ipv6 route ::/0 g0/0/0 2001:DB8:ACAD:2::2
```

R2

```
conf t
ipv6 route ::/0 g0/0/0 2001:DB8:ACAD:2::1
```

Конфигурационные файлы [R1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/R1.cfg) и [R2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/R2.cfg).

Провел тест подтверждающий работу маршрутизации. Для этого отправил эхо-запрос на на адрес G0/0/1 R2:

```
R1#ping 2001:db8:acad:3::1 

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:acad:3::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```

#### II. Проверка назначения адреса методом SLAAC от маршрутизатора R1.

Для проверки автоматической раздачи IPv6-адресов запустил компьютер PC-A. Сетевая карта настроена на автоматическое получение адресов. В командной строке ввел команду ipconfig.

```
C:\>ipconfig 

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::260:3EFF:FE68:3377
   IPv6 Address....................: 2001:DB8:ACAD:1:260:3EFF:FE68:3377
   Autoconfiguration IPv4 Address..: 169.254.51.119
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
```

*Вопрос:* Откуда взялась часть адреса, содержащая идентификатор хоста?

*Ответ:* Если не используется DHCP сервер, то это часть адреса генерируется автоматически (генератор случайных чисел).

#### III. Настройка и проверка сервера DHCPv6 на маршрутизаторе R1.

В этой части лабораторной работы настроил DHCP-сервер без отслеживания состояния на маршрутизаторе R1. Нужно сделать так, чтобы PC-A получил информацию о DNS-сервере и домене.

#### Изучение более подробной конфигурации PC-A.

Для получения более расширенной информации о настройках PC выполнила команду ipconfig /all. 

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0060.3E68.3377
   Link-local IPv6 Address.........: FE80::260:3EFF:FE68:3377
   IPv6 Address....................: 2001:DB8:ACAD:1:260:3EFF:FE68:3377
   Autoconfiguration IP Address....: 169.254.51.119
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-0A-74-39-24-00-60-3E-68-33-77
   DNS Servers.....................: ::
                                     0.0.0.0
```

#### Настройка R1 для предоставления DHCPv6 без сохранения состояния для PC-A.

- На маршрутизаторе R1 создал пул DHCP IPv6 с именем R1-STATELESS. В пуле назначил адрес DNS-сервера и имя домена.

  ```
  conf t
  ipv6 dhcp pool R1-STATELESS
    dns-server 2001:db8:acad::254
    domain-name STATELESS.com
    exit
  ```

  Настроила интерфейс E0/1 на роутере R1 для предоставления флага конфигурации OTHER для локальной сети маршрутизатора R1, а так же указала созданный пул как DHCP ресурс для этого интерфейса.

  ```
  conf t
  int g0/0/1
    ipv6 nd other-config-flag
    ipv6 dhcp server R1-STATELESS
    exit
  ```

Конфигурационный файл [R1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/R1.cfg).

- После этого, перезагружаем компьютер и повторно вводим команду ipconfig /all. Вывод в котором видим, что появился Connection-specific DNS Suffix - STATELESS.com

  ```
  C:\>ipconfig /all
  
  FastEthernet0 Connection:(default port)
  
     Connection-specific DNS Suffix..: STATELESS.com 
     Physical Address................: 0060.3E68.3377
     Link-local IPv6 Address.........: FE80::260:3EFF:FE68:3377
     IPv6 Address....................: 2001:DB8:ACAD:1:260:3EFF:FE68:3377
     Autoconfiguration IP Address....: 169.254.51.119
     Subnet Mask.....................: 255.255.0.0
     Default Gateway.................: FE80::1
                                       0.0.0.0
     DHCP Servers....................: 0.0.0.0
     DHCPv6 IAID.....................: 695836627
     DHCPv6 Client DUID..............: 00-01-00-01-0A-74-39-24-00-60-3E-68-33-77
     DNS Servers.....................: 2001:DB8:ACAD::254
                                       0.0.0.0
  ```

- Для проверки подключения, отправила эхо-запрос на IP-адрес интерфейса G0/0/1 R2 - 2001:db8:acad:3::1/64.

  ```
  C:\>ping 2001:db8:acad:3::1
  
  Pinging 2001:db8:acad:3::1 with 32 bytes of data:
  
  Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
  Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
  Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
  Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
  
  Ping statistics for 2001:DB8:ACAD:3::1:
      Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
  Approximate round trip times in milli-seconds:
      Minimum = 0ms, Maximum = 0ms, Average = 0ms
  ```

#### IV. Настройка DHCPv6-сервера с отслеживанием состояния на R1.

В этой части работы я создал пул DHCPv6 на маршрутизаторе R1 для сети 2001:db8:acad:3:aaaa::/80. Это предоставит адреса локальной сети, подключенной к интерфейсу g0/0/1 на R2. В составе пула установила DNS-сервер - 2001:db8:acad::254 и установил имя домена - STATEFUL.com. Далее назначил созданный пул на интерфейс G0/0/0.

```
conf t
ipv6 dhcp pool R2-STATEFUL
 address prefix 2001:db8:acad:3:aaa::/80
 dns-server 2001:db8:acad::254
 domain-name STATEFUL.com
 exit
!
interface g0/0/0
 ipv6 dhcp server R2-STATEFUL
 exit
```

Конфигурационный файл [R1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/R1.cfg).

#### IV. Настройка и проверка ретранслятора DHCPv6 на R2.

Настройка ретрансляции на R2 позволит получать адрем IPv6 на компьютере PC-B.

#### Проверка на компьютере PC-B генерируемого им адреса SLAAC.

- Для проверки ввел команду ipconfig /all. Вывод в котором видим сгенерированный автоматически кусок адреса и префикс 2001:db8:acad:3

  ```
  C:\>ipconfig /all
  
  FastEthernet0 Connection:(default port)
  
     Connection-specific DNS Suffix..: 
     Physical Address................: 0001.42E2.E789
     Link-local IPv6 Address.........: FE80::201:42FF:FEE2:E789
     IPv6 Address....................: 2001:DB8:ACAD:3:201:42FF:FEE2:E789
     Autoconfiguration IP Address....: 169.254.231.137
     Subnet Mask.....................: 255.255.0.0
     Default Gateway.................: FE80::1
                                       0.0.0.0
     DHCP Servers....................: 0.0.0.0
     DHCPv6 IAID.....................: 
     DHCPv6 Client DUID..............: 00-01-00-01-D0-61-26-94-00-01-42-E2-E7-89
     DNS Servers.....................: ::
                                       0.0.0.0
  ```

#### Настройка R2 в качестве агента ретрансляции DHCPv6 для LAN на G0/0/1.

На интерфейсе G0/0/1 маршрутизатора R2 ввел команду ipv6 dhcp relay с адресом интерфейса G0/0/0 на R1, а так же задал режим сервера с отслеживанием состояния командой managed-config-flag. 

```
conf t
interface g0/0/1
 ipv6 nd managed-config-flag
 ipv6 dhcp relay destination 2001:db8:acad:2::1 g0/0/0
 exit
```

Конфигурационный файл [R2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab05/ipv6/R2.cfg).

#### Получение IPv6-адреса от DHCPv6 на PC-B.

- Перезагрузила компьюте PC-B.
- В командной строке ввел команду ipconfig /all. Теперь мы видим адрес DNS-сервера и DNS Suffix.

```
C:\Users\Student> ipconfig /all
Windows IP Configuration

   Host Name . . . . . . . . . . . . : DESKTOP-3FR7RKA
   Primary Dns Suffix  . . . . . . . : 
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : STATEFUL.com

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : STATEFUL.com
   Description . . . . . . . . . . . : Intel(R) 852574L Gigabit Network Connection
   Physical Address. . . . . . . . . : 00-50-56-B3-7B-06
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : 2001:db8:acad3:aaaa:7104:8b7d:5402(Preferred)
   Lease Obtained. . . . . . . . . . : Sunday, October 6, 2019 3:27:13 PM
   Lease Expires . . . . . . . . . . : Tuesday, October 8, 2019 3:27:13 PM
   Link-local IPv6 Address . . . . . : fe80::a0f3:3d39:f9fb:a020%6(Preferred)
   IPv4 Address. . . . . . . . . . . : 169.254.160.32(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : fe80::2%6
   DHCPv6 IAID . . . . . . . . . . . : 50334761
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-24-F2-08-38-00-50-56-B3-7B-06
   DNS Servers . . . . . . . . . . . : 2001:db8:acad::254
   NetBIOS over Tcpip. . . . . . . . : Enabled
   Connection-specific DNS Suffix Search List  :
                                       STATEFUL.com
```

- Последним шагом проверяю возможность подключения, отправив эхо-запрос на IP-адрес интерфейса G0/0/1 маршрутизатора R1. Результат на рис. 3

Рис. 3

[![img](https://github.com/wiseowl-lna/net_engineer/raw/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-B_ping_R1_E0_1.png)](https://github.com/wiseowl-lna/net_engineer/blob/master/labs/Lab003_DHCPv4_6_SLAAC/Lab003_DHCPv6/PC-B_ping_R1_E0_1.png)