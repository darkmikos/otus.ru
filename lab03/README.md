## **Лабораторная работа. Развертывание коммутируемой сети с резервными каналами**

### Topology

Рис.1

![topology](https://github.com/darkmikos/otus.ru/blob/master/lab03/topology.png)



### Connection diagram:

Рис. 2

![connection_diagram](https://github.com/darkmikos/otus.ru/blob/master/lab03/%D1%81onnection_diagram.png)



### Таблица адресации:

Таб. 1

| Устройство | Интерфейс | IP-адрес    | Маска подсети |
| ---------- | --------- | ----------- | ------------- |
| S1         | VLAN 1    | 192.168.1.1 | 255.255.255.0 |
| S2         | VLAN 1    | 192.168.1.2 | 255.255.255.0 |
| S3         | VLAN 1    | 192.168.1.3 | 255.255.255.0 |

### Задание:

Необходимо настроить оборудование и организовать сетевую связанность согласно условию лабораторной работы.

1. Создать сеть и настроить основные параметры устройства.
2. Необходимо выбрать корневой мост.
3. Пронаблюдать за процессом выбора протоколом STP порта, исходя из стоимости портов.
4. Пронаблюдать за процессом выбора протоколом STP порта, исходя из приоритета портов.

### Ход выполнения:

Для выполнения лабораторной работы использовал эмулятор Cisco Packet Tracer 8.0.

#### I. Создание сети и настройка основных параметров устройства.

#### Сбор схемы:

1. Собрал схему, как показано на рисунке 1.

#### Инициализация коммутаторов:

1. Подключился к коммутатору с помощью консоли, защел в привилигированный режим.

2. Проверил созданы на коммутаторе VLAN. Файла **vlan.dat** не обнаружен. В случае обнаружения во флеш-памяти, файл нужно удалить с помощью команды.

   ``` 
   show flash:
   delete vlan.dat

3. Следующим этапом необходимо удалить файл загрузочной конфигурации из NVRAM.

   ``` 
   erase startup-config

4. Перезапустил коммутатор.

   ```
   reload
   ```

5. Повторил процедуру на всех используемых коммутаторах.

#### Настройка базовых параметров коммутатора:

Настройки коммутаторов находятся в файлах [S1.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab03/S1.cfg), [S2.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab03/S2.cfg), [S3.cfg](https://github.com/darkmikos/otus.ru/blob/master/lab03/S3.cfg).

```

conf t
!
hostname S*(1-3)
no ip domain-lookup
!
enable secret class
!
line console 0
password cisco
login
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
clock set 19:50 3 may 2021
!
write
!
conf t
!
 interface vlan 1
 ip address 192.168.1.*(1-3) 255.255.255.0
 no shutdown
 exit
exit
!
exit
!
write
```

#### Проверка связи между коммутаторами:

1.  Эхо-запрос от коммутатора S1 на коммутатор S2.

   ```
   S1>ping 192.168.1.2
   
   Type escape sequence to abort.
   Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
   !!!!!
   Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
   ```

2. Эхо-запрос от коммутатора S1 на коммутатор S3.

   ```
   S1>ping 192.168.1.3
   
   Type escape sequence to abort.
   Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
   !!!!!
   Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
   ```

3. Эхо-запрос от коммутатора S2 на коммутатор S3.

   ```
   S2>ping 192.168.1.3
   
   Type escape sequence to abort.
   Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
   !!!!!
   Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
   ```

​	*Данных проверок достаточно, так как проведена проверка связи между всеми устройствами участвующими в данном занятии.*

#### II. Определение корневого моста.

1. Отключил все порты на коммутаторах S1, S2, S3.

   ```
   conf t
   !
   interface range fastEthernet 0/1-24, gigabitEthernet 0/1-2
    shutdown
    exit
   exit
   ```

2. Настроил подключенные порты в качестве транковых ([Рис.1](https://github.com/darkmikos/otus.ru/blob/master/lab03/topology.png)).

   ```
   conf t
   !
   interface range fastEthernet 0/1-4
    switchport mode trunk
    exit
   exit

3. Включил порты F0/2 и F0/4 на всех коммутаторах.

```
conf t
!
interface range f0/2, f0/4
 no shutdown
 exit
exit
```

4. Отобразил данные протокола spanning-tree.

```
S1#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.1668.6C7E
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0002.1668.6C7E
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```

```
S2#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.1668.6C7E
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.D397.5070
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Root FWD 19        128.2    P2p
Fa0/4            Altn BLK 19        128.4    P2p
```

```
S3#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.1668.6C7E
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0C71.051B
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

​	Изучив вывод команды **show spanning-tree** со всех устройств, заполнил схему ([Рис. 3](https://github.com/darkmikos/otus.ru/blob/master/lab03/pic3.png))

![Рис. 3](https://github.com/darkmikos/otus.ru/blob/master/lab03/pic3.png)

​	С учетом выходных данных, поступающих с коммутаторов, получил ответы на следующие вопросы

*Вопрос_1:* Какой коммутатор является корневым мостом?

*Ответ:* Коммутатор S1.

*Вопрос_2:* Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?

*Ответ:* Так как выбор корневого моста складывается из значений приоритета (по умолчанию 32768), расширенного идентификатора системы - VLAN (в моем случае – это vlan 1 на всех коммутаторах) и MAC-адреса коммутатора (уникальные для каждого коммутатора), а коммутатор с самым низким значением MAC-адреса, при равных остальных условиях, становится корневым мостом, то следовательно, в моем случае – это коммутатор S1 ([Рис. 3](https://github.com/darkmikos/otus.ru/blob/master/lab03/pic3.png))

*Вопрос_3:* Какие порты на коммутаторе являются корневыми портами?

*Ответ:* Порт коммутатора, который имеет кратчайший путь к корневому коммутатору называется корневым портом. У любого не корневого коммутатора может быть только один корневой порт. В моем случае – это порт F0/2 на коммутаторе S2 и порт F0/4 на коммутаторе S3.

*Вопрос_4:* Какие порты на коммутаторе являются назначенными портами?

*Ответ:* Коммутатор в сегменте сети, имеющий наименьшее расстояние до корневого коммутатора называется назначенным коммутатором (мостом). Порт этого коммутатора, который подключен к рассматриваемому сегменту сети, называется назначенным портом. В моем случае – это порт F0/2 на коммутаторе S3.

*Вопрос_5:* Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?

*Ответ:* В моем случае – это порт F0/4 на коммутаторе S2  ([Рис. 3](https://github.com/darkmikos/otus.ru/blob/master/lab03/pic3.png)).

*Вопрос_6:* Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?

*Ответ:* Порт коммутатора, имеющего самое наибольшее расстояние до корневого коммутатора, и, к тому же, который подключен к назначенному коммутатору называется альтернативным и блокируется до тех пор, пока основной путь не будет разорван.

В результате работы протокола spanning-tree получается древовидная структура (математический граф) с вершиной в виде корневого коммутатора.

#### III. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов.

1. В моей текущей конфигурации только один коммутатор может содержать заблокированный протоколом STP порт. Выполнил команду **show spanning-tree** на обоих коммутаторах некорневого моста. В моем примере протокол spanning-tree блокирует порт F0/4 на коммутаторе с самым высоким идентификатором BID (S2).

2. Изменил стоимость, оставшегося не заблокированным, порта на коммутаторе S2, понизив его на единицу.

```
conf t
 interface f0/2
 spanning-tree vlan 1 cost 18
 exit
exit
```

3. В течении 30 секунд spanning-tree перестроился:

![Рис. 4](https://github.com/darkmikos/otus.ru/blob/master/lab03/pic4.png)

​	Запустил команду **show spanning-tree** на коммутаторах S2 и S3.

````
S2#sh sp
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.1668.6C7E
             Cost        18
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.D397.5070
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Root FWD 18        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p
````

```
S3#sh sp
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.1668.6C7E
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0C71.051B
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

*Вопрос:* Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?

*Ответ:* Алгоритм протокола spanning-tree (STA) использует корневой мост как точку привязки, после чего определяет, какие порты будут заблокированы, исходя из стоимости пути. Порт с более низкой стоимостью пути является предпочтительным, По-этому протокол заблокировал порт Fa0/2 со стоимостью 19 на коммутаторе S3.

4. Вернул стоимость порта F0/2 на коммутаторе S2 в первоначальное состояние.

```
conf t
 interface f0/2
 no spanning-tree vlan 1 cost 18
 exit
exit
```

#### IV. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов.

1.  Включил порты F0/1 и F0/3 на всех коммутаторах.

   ```
   conf t
    interface range fa0/1, fa0/3
     no shutdown 
     exit
   exit
   ```

2.  Через 30 секунд, после того как протокол STP завершит процесс перестроения дерева, мы видим, что порт корневого моста переместился на порт с меньшим номером, связанный с коммутатором корневого моста, и заблокировал предыдущий порт корневого моста.

![Рис. 5](https://github.com/darkmikos/otus.ru/blob/master/lab03/pic5.png) 

Так же это можно увидеть с помощью команды **show spanning-tree**, запущенной на некорневых коммутаторах.

```
S2#sh sp
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.1668.6C7E
             Cost        19
             Port        1(FastEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.D397.5070
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Root FWD 19        128.1    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/3            Altn BLK 19        128.3    P2p
Fa0/4            Altn BLK 19        128.4    P2p
```

```
S3#sh sp
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0002.1668.6C7E
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0C71.051B
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/3            Root FWD 19        128.3    P2p
Fa0/4            Altn BLK 19        128.4    P2p
```

*Вопрос:* Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?

*Ответ:* На коммутаторе S2 - это порт F0/1, на коммутаторе S3 - порт F0/3.

*Вопрос:* Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?

*Ответ:* Потому, что это порты с наименьшим номером, что является более приоритетным.

#### Вопросы для повторения.

*Вопрос_1:* Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?

*Ответ:* Этим значением является стоимость порта (cost).

*Вопрос_2:* Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?

*Ответ:* Далее протокол проверяет приоритет порта (по умолчанию - 128).

*Вопрос_3:* Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?

*Ответ:* Номер порта.

В итоге получаем:

Выбор корневого моста складывается из:

```
         BID = Pm + Vlan + MAC, где

   BID - идентификатора моста,
   Pm - приоритет моста (значение приоритета может находиться в диапазоне от 0 до 65535 с шагом 4096. По умолчанию используется значение 32768),
   Vlan - номер сети VLAN,
   MAC - mac-адрес коммутатора.
```

Коммутатор с наименьшим BID становится корневым.

Выбор корневого порта складывается из:

```
          BID = Cost + Prio + Np, где

   Cost - стоимость порта,
   Prio - приоритет порта (по умолчанию - 128),
   Np - номар порта.
```

Порт с наименьшим BID становится корневым.