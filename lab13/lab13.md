# Настройка протоколов CDP, LLDP и NTP
### Топология
![](топология.png)
### Таблица адресации
| Устройство | Интерфейс | IP‑адрес | Маска подсети | Шлюз по умолчанию |
|----------|----------|----------|--------------|----------------|
| R1 | Loopback1 | 172.16.1.1 | 255.255.255.0 | -        |
| R1 | G0/0/1 | 10.22.0.1   | 255.255.255.0 | -          |
| S1 | SVI VLAN 1 | 10.22.0.2 | 255.255.255.0 |10.22.0.1|
| S2 | SVI VLAN 1 | 10.22.0.3 | 255.255.255.0 |10.22.0.1|
### Задачи
1. Создание сети и настройка основных параметров устройства
2. Обнаружение сетевых ресурсов с помощью протокола CDP
3. Обнаружение сетевых ресурсов с помощью протокола LLDP
4. Настройка и проверка NTP
### Часть 1. Создание сети и настройка основных параметров устройства
Создаем топологию сети и настраиваем базовые параметры для маршрутизатора и коммутаторов
#### Шаг 1. Создайте сеть согласно топологии.
Подключаем устройства, как показано в топологии, и подсоединяем необходимые кабели.
#### Шаг 2. Настройте базовые параметры для маршрутизатора.
Входим в привилегированный режим.    
Входим в режим глобальной конфигурации.   
Задаем имя маршрутизатору.   
Отключаем интерпретацию команды как DNS имя - на случай ввода команды с ошибкой.    
Включаем шифрование паролей.   
Устанавливаем пароль для доступа к маршрутизатору через консольный кабель и включаем доступ к пользовательскому режиму.   
Устанавливаем локальный пароль доступа в привилегированный режим консоли.   
Устанавливаем пароль VTY и включаем вход в систему по паролю.    
Задаем баннерное сообщение при входе в систему.    
```
enable
configure terminal
hostname R1
no ip domain-lookup
service password-encryption
line console 0
password cisco
login
enable secret class
line vty 0 4
password cisco
login
banner motd @--- Unauthorized access is strictly prohibited ---@
```
Настраиваем интерфейсы на R1:
```
interface Loopback 1
ip address 172.16.1.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet 0/0/1
ip address 10.22.0.1 255.255.255.0
no shutdown
```
Сохраняем текущую конфигурацию в файл загрузочной конфигурации с помощью команды ***copy running-config startup-config***
#### Шаг 3. Настройте базовые параметры каждого коммутатора.
Входим в привилегированный режим.    
Входим в режим глобальной конфигурации.   
Задаем имя коммутатора.   
Отключаем интерпретацию команды как DNS имя - на случай ввода команды с ошибкой.   
Включаем шифрование паролей.   
Устанавливаем пароль для доступа к коммутатору через консольный кабель и включаем доступ к пользовательскому режиму.   
Устанавливаем локальный пароль доступа в привилегированный режим консоли.   
Задаем баннерное сообщение при входе в систему.    
```
enable
configure terminal
hostname S1
no ip domain-lookup
service password-encryption
line console 0
password cisco
login
enable secret class
banner motd @--- Unauthorized access is strictly prohibited ---@
```
**Повторяем процедуру для второго коммутатора.**    
Выключаем все интерфейсы, которые не будут использоваться.    
```
S1:
interface range FastEthernet 0/2-4, FastEthernet 0/6-24, gigabitEthernet 0/1-2
shutdown
S2:
interface range FastEthernet 0/2-24, gigabitEthernet 0/1-2
shutdown
```
Настраиваем IP-адресации интерфейса, как указано в таблице выше.
```
S1:
interface Vlan1
ip address 10.22.0.2 255.255.255.0
no shutdown
S2:
interface Vlan1
ip address 10.22.0.3 255.255.255.0
no shutdown
```
Сохраняем текущую конфигурацию в файл загрузочной конфигурации с помощью команды ***copy running-config startup-config***
### Часть 2. Обнаружение сетевых ресурсов с помощью протокола CDP
На устройствах Cisco протокол CDP включен по умолчанию. Воспользуемся CDP, чтобы обнаружить порты, к которым подключены кабели.    
На R1 используем команду ***show cdp interface***, чтобы определить, сколько интерфейсов включено CDP, сколько из них включено и сколько отключено.
```
R1#show cdp interface 
Vlan1 is administratively down, line protocol is down
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
GigabitEthernet0/0/0 is administratively down, line protocol is down
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
GigabitEthernet0/0/1 is up, line protocol is up
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
```
Видим, что все интерфейсы участвует в объявлениях CDP, но активен только интерфейс GigabitEthernet0/0/1   
      
На R1 используем соответствующую команду ***show cdp entry***, чтобы определить версию IOS, используемую на S1.
```
R1#show cdp entry  S1
Device ID: S1
Entry address(es): 
  IP address : 10.22.0.2
Platform: cisco 2960, Capabilities: Switch
Interface: GigabitEthernet0/0/1, Port ID (outgoing port): FastEthernet0/5
Holdtime: 168

Version :
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen

advertisement version: 2
Duplex: full
```
На коммутаторе S1 используется версия IOS 15.0(2)SE4.   
       
На S1 используем соответствующую команду ***show cdp traffic***, чтобы определить, сколько пакетов CDP было выданных.
```
S1# show cdp traffic
CDP counters : 
        Total packets output: 179, Input: 148 
        Hdr syntax: 0, Chksum error: 0, Encaps failed: 0 
        No memory: 0, Invalid packet: 0, 
        CDP version 1 advertisements output: 0, Input: 0 
        CDP version 2 advertisements output: 179, Input: 148
```
C момента последнего сброса счетчика маршрутизатор отправил 179 CDP пакетов.    
      
SVI для VLAN 1 на S1 и S2 Уже настроены. Настраиваем шлюз по умолчанию для каждого коммутатора на основе таблицы адресов.
```
ip default-gateway 10.22.0.1
```
На R1 выполняем команду ***show cdp entry S1***.
```
R1#show cdp entry  S1

Device ID: S1
Entry address(es): 
  IP address : 10.22.0.2
Platform: cisco 2960, Capabilities: Switch
Interface: GigabitEthernet0/0/1, Port ID (outgoing port): FastEthernet0/5
Holdtime: 121

Version :
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen

advertisement version: 2
Duplex: full
```
После настройки адресации на коммутаторах становится доступна информация о IP-адресе соседнего устройства.   
     
Отключаем CDP глобально на всех устройствах с помощью команды ***no cdp run***, проверяем результат: 
```
R1#show cdp interface 
% CDP is not enabled
```
### Часть 3. Обнаружение сетевых ресурсов с помощью протокола LLDP
На устройствах Cisco протокол LLDP может быть включен по умолчанию. Воспользуемся LLDP, чтобы обнаружить порты, к которым подключены кабели.
Вводим  команду ***lldp run***, чтобы включить LLDP на всех устройствах в топологии.    
На S1 выполняем команду ***show lldp neighbor detail***, чтобы предоставить подробную информацию о S2. 
```
Chassis id: 0060.708A.3001
Port id: Fa0/1
Port Description: FastEthernet0/1
System Name: S2
System Description:
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen
Time remaining: 90 seconds
System Capabilities: B
Enabled Capabilities: B
Management Addresses - not advertised
Auto Negotiation - supported, enabled
Physical media capabilities:
    100baseT(FD)
    100baseT(HD)
    1000baseT(HD)
Media Attachment Unit type: 10
Vlan ID: 1
```
chassis ID  для коммутатора S2 - это его MAC-адрес.   
       
Настраиваем возможность SSH подключения на R1:
```
ip domain-name otus.ru
username SSHadmin secret cisco
crypto key generate rsa
How many bits in the modulus [512]: 1024
ip ssh version 2
line vty 0 15
transport input ssh
login local
```
Настраиваем возможность SSH подключения на S1 и S2:
```
ip domain-name otus.ru
username SSHadmin secret cisco
crypto key generate rsa
How many bits in the modulus [512]: 1024
ip ssh version 2
line vty 0 15
transport input ssh
login local
```
Соединяемся через консоль на всех устройствах и используем команды LLDP, необходимые для отображения топологии физической сети только из выходных данных команды ***show***:
```
R1#show lldp neighbors
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID           Local Intf     Hold-time  Capability      Port ID
S1                  Gig0/0/1       120        B               Fa0/5

Total entries displayed: 1
```
```
S2>show lldp neighbors
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID           Local Intf     Hold-time  Capability      Port ID
S1                  Fa0/1          120        B               Fa0/1

Total entries displayed: 1
```
```
S1#show lldp neighbors
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID           Local Intf     Hold-time  Capability      Port ID
R1                  Fa0/5          120        R               Gig0/0/1
S2                  Fa0/1          120        B               Fa0/1

Total entries displayed: 2
```
Изучив результаты вывода команд ***show lldp neighbors*** на каждом устройстве, мы отметили, что к S1 подключены 2 устройства: R1 и S2. При этом к маршрутизатору R1 и коммутатору S2 подключен только S1. Таким образом, схема сети следующая: R1--S1--S2
### Часть 4. Настройка NTP
#### Шаг 1. Выведите на экран текущее время.
Вводим команду ***show clock*** для отображения текущего времени на R1. 
```
R1#show clock
*1:21:44.794 UTC Mon Mar 1 1993
```		
#### Шаг 2. Установите время.
С помощью команды ***clock set*** устанавливаем время на маршрутизаторе R1 в формате UTC. 
```
clock set 01:36:55 april 28 2026
clock timezone MSK 3
```		
#### Шаг 3. Настройте главный сервер NTP.
Настраиваем R1 в качестве хозяина NTP с уровнем слоя 4.
```
ntp master 4
```
Проверяем настройки NTP:
```
R1(config)#do show ntp status
Clock is synchronized, stratum 4, reference is 127.127.1.1
nominal freq is 250.0000 Hz, actual freq is 249.9990 Hz, precision is 2**24
reference time is ED71B046.000001DD (1:41:26.477 UTC Tue Apr 28 2026)
clock offset is 0.00 msec, root delay is 0.00  msec
root dispersion is 0.00 msec, peer dispersion is 0.12 msec.
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is - 0.000001193 s/s system poll interval is 4, last update was 3 sec ago.
```
#### Шаг 4. Настройте клиент NTP.
Вводим команду ***show clock*** для отображения текущего времени на S1 и S2, чтобы просмотреть настроенное время.
```
S1>show clock
*1:32:34.234 UTC Mon Mar 1 1993

S2>show clock
*1:32:57.441 UTC Mon Mar 1 1993
```	
Настраиваем S1 и S2 в качестве клиентов NTP.
```
ntp server 10.22.0.1
clock timezone MSK 3
```	
#### Шаг 5. Проверьте настройку NTP.
Используем команду ***show ntp status*** , чтобы убедиться, что S1 и S2 синхронизированы с R1.
```
S1(config)#do show ntp status
CClock is synchronized, stratum 5, reference is 10.22.0.1
nominal freq is 250.0000 Hz, actual freq is 249.9990 Hz, precision is 2**24
reference time is ED718FAE.00000080 (23:22:22.128 UTC Mon Apr 27 2026)
clock offset is 0.00 msec, root delay is 0.00  msec
root dispersion is 10.14 msec, peer dispersion is 0.12 msec.
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is - 0.000001193 s/s system poll interval is 5, last update was 32 sec ago.
```
Видим, что часы синхронизированы, на S1 установлен стратум 5, данные по времени берутся с 10.22.0.1 - R1.    
```
S2#show ntp associations

address         ref clock       st   when     poll    reach  delay          offset            disp
*~10.22.0.1     127.127.1.1     4    6        16      37     0.00           0.00              0.00
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
```
Видим адрес NTP сервера, стратум R1 - 4.    
Выполняем команду ***show clock*** на S1 и S2, чтобы просмотреть настроенное время и сравнить ранее записанное время.
```
S1#sh clock
2:25:4.492 MSK Tue Apr 28 2026

S2#sh clock
2:25:22.531 MSK Tue Apr 28 2026
```
Видим, что время на коммутаторах соответствует времени на маршрутизаторе.
