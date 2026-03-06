# Конфигурация безопасности коммутатора
### Топология
![](топология.png)
### Таблица адресации
| Устройство | Interface/VLAN | IP-адрес | Маска подсети |
|-----------|---------------|----------|--------------|
| R1 | G0/0/1 | 192.168.10.1 | 255.255.255.0 |
| R1 | Loopback 0 | 10.10.1.1 | 255.255.255.0 |
| S1 | VLAN 10 | 192.168.10.201 | 255.255.255.0 |
| S2 | VLAN 10 | 192.168.10.202 | 255.255.255.0 |
| PC A | NIC | DHCP | 255.255.255.0 |
| PC B | NIC | DHCP | 255.255.255.0 |
### Цели
##### Часть 1. Настройка основного сетевого устройства
•	Создайте сеть.    
•	Настройте маршрутизатор R1.    
•	Настройка и проверка основных параметров коммутатора    
##### Часть 2. Настройка сетей VLAN
•	Сконфигруриуйте VLAN 10.    
•	Сконфигруриуйте SVI для VLAN 10.    
•	Настройте VLAN 333 с именем Native на S1 и S2.    
•	Настройте VLAN 999 с именем ParkingLot на S1 и S2.    
##### Часть 3: Настройки безопасности коммутатора.
•	Реализация магистральных соединений 802.1Q.    
•	Настройка портов доступа    
•	Безопасность неиспользуемых портов коммутатора    
•	Документирование и реализация функций безопасности порта.    
•	Реализовать безопасность DHCP snooping.    
•	Реализация PortFast и BPDU Guard    
•	Проверка сквозной связанности.   
### 1. Настройка основного сетевого устройства
#### Шаг 1. Создайте сеть.
Создаем сеть согласно топологии, инициализируем устройства.
#### Шаг 2. Настройте маршрутизатор R1.
Загружаем следующий конфигурационный скрипт на R1:
```
enable
configure terminal
hostname R1
no ip domain lookup
ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp excluded-address 192.168.10.201 192.168.10.202
ip dhcp relay information trust-all
!
ip dhcp pool Students
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 domain-name CCNA2.Lab-11.6.1
!
interface Loopback0
 ip address 10.10.1.1 255.255.255.0
!
interface GigabitEthernet0/0/1
 description Link to S1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
!
line con 0
 logging synchronous
 exec-timeout 0 0
```
Проверяем текущую конфигурацию на R1, используя команду ***show ip interface brief***.   
```
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   192.168.10.1    YES manual up                    up 
Loopback0              10.10.1.1       YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```
IP-адресация и интерфейсы находятся в состоянии up / up.
#### Шаг 3. Настройка и проверка основных параметров коммутатора
Настраиваем имена хостов для коммутаторов S1 и S2.   
Запрещаем нежелательный поиск в DNS командой ***no ip domain-lookup***.
Настраиваем описания интерфейса для портов, которые используются в S1:
```
interface fa0/6
 description Link to PC-A
 no shutdown
!
interface fa0/5
 description Link to R1
 no shutdown
!
interface fa0/1
 description Link to S1
 no shutdown
```
**Повторяем процедуру для S2**     
Устанавливаем для шлюза по умолчанию для VLAN управления значение 192.168.10.1 на обоих коммутаторах:
```
ip default-gateway 192.168.10.1
```
### Часть 2. Настройка сетей VLAN на коммутаторах.
#### Шаг 1. Сконфигруриуйте VLAN 10.
Добавляем VLAN 10 на S1 и S2 с названием VLAN "Management".
```
vlan 10
 name Management
```
#### Шаг 2. Сконфигруриуйте SVI для VLAN 10.
Настраиваем IP-адрес в соответствии с таблицей адресации для SVI для VLAN 10 на S1 и S2. Включаем интерфейсы SVI и предоставляем описание для интерфейса.
```
interface vlan 10
 ip address 192.168.10.201 255.255.255.0
 description Management
 no shutdown
```
**Повторяем процедуру для S2**     
#### Шаг 3. Настройте VLAN 333 с именем Native на S1 и S2.
```
vlan 333
 name Native
```
#### Шаг 4. Настройте VLAN 999 с именем ParkingLot на S1 и S2.
```
vlan 999
 name ParkingLot
```
### Часть 3. Настройки безопасности коммутатора.
#### Шаг 1. Релизация магистральных соединений 802.1Q.
Настраиваем все магистральные порты Fa0/1 на обоих коммутаторах для использования VLAN 333 в качестве native VLAN.
```
interface fa0/1
 switchport mode trunk
 switchport trunk native vlan 333
 no shutdown
```
Убеждаемся, что режим транкинга успешно настроен на всех коммутаторах, используя команду ***show interface trunk***
```
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      333

Port        Vlans allowed on trunk
Fa0/1       1-1005

Port        Vlans allowed and active in management domain
Fa0/1       1,10,333,999

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,333,999
```
Отключаем согласование DTP F0/1 на S1 и S2:
```
interface fa0/1
switchport nonegotiate
```
Проверяем с помощью команды ***show interfaces f0/1 switchport | include Negotiation***
```
Negotiation of Trunking: Off
```
#### Шаг 2. Настройка портов доступа
На S1 настраиваем F0/5 и F0/6 в качестве портов доступа и связываем их с VLAN 10.
```
interface range fa0/5 - 6
switchport mode access
switchport access vlan 10
no shutdown
```
На S2 настраиваем порт доступа Fa0/18 и связываем его с VLAN 10.
```
interface fa0/18
switchport mode access
switchport access vlan 10
no shutdown
```
#### Шаг 3. Безопасность неиспользуемых портов коммутатора
На S1 и S2 перемещаем неиспользуемые порты из VLAN 1 в VLAN 999 и отключаем неиспользуемые порты.
```
interface range fastEthernet 0/2-4, fastEthernet 0/7-24, gigabitEthernet 0/1-2
switchport mode access 
switchport access vlan 999
shutdown

interface range fastEthernet 0/2-17, fastEthernet 0/19-24, gigabitEthernet 0/1-2
switchport mode access 
switchport access vlan 999
shutdown
```
Убеждаемся, что неиспользуемые порты отключены и связаны с VLAN 999, введя команду  ***show interfaces status***
```
S1# show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     Link to S2         connected    trunk      auto    auto  10/100BaseTX
Fa0/2                        disabled     999        auto    auto  10/100BaseTX
Fa0/3                        disabled     999        auto    auto  10/100BaseTX
Fa0/4                        disabled     999        auto    auto  10/100BaseTX
Fa0/5     Link to R1         connected    10         auto    auto  10/100BaseTX
Fa0/6     Link to PC-A       connected    10         auto    auto  10/100BaseTX
Fa0/7                        disabled     999        auto    auto  10/100BaseTX
Fa0/8                        disabled     999        auto    auto  10/100BaseTX
Fa0/9                        disabled     999        auto    auto  10/100BaseTX
Fa0/10                       disabled     999        auto    auto  10/100BaseTX
Fa0/11                       disabled     999        auto    auto  10/100BaseTX
Fa0/12                       disabled     999        auto    auto  10/100BaseTX
Fa0/13                       disabled     999        auto    auto  10/100BaseTX
Fa0/14                       disabled     999        auto    auto  10/100BaseTX
Fa0/15                       disabled     999        auto    auto  10/100BaseTX
Fa0/16                       disabled     999        auto    auto  10/100BaseTX
Fa0/17                       disabled     999        auto    auto  10/100BaseTX
Fa0/18                       disabled     999        auto    auto  10/100BaseTX
Fa0/19                       disabled     999        auto    auto  10/100BaseTX
Fa0/20                       disabled     999        auto    auto  10/100BaseTX
Fa0/21                       disabled     999        auto    auto  10/100BaseTX
Fa0/22                       disabled     999        auto    auto  10/100BaseTX
Fa0/23                       disabled     999        auto    auto  10/100BaseTX
Fa0/24                       disabled     999        auto    auto  10/100BaseTX
Gig0/1                       disabled     999        auto    auto  10/100BaseTX
Gig0/2                       disabled     999        auto    auto  10/100BaseTX

S2# show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     Link to S1         connected    trunk      auto    auto  10/100BaseTX
Fa0/2                        disabled     999        auto    auto  10/100BaseTX
Fa0/3                        disabled     999        auto    auto  10/100BaseTX
Fa0/4                        disabled     999        auto    auto  10/100BaseTX
Fa0/5                        disabled     999        auto    auto  10/100BaseTX
Fa0/6                        disabled     999        auto    auto  10/100BaseTX
Fa0/7                        disabled     999        auto    auto  10/100BaseTX
Fa0/8                        disabled     999        auto    auto  10/100BaseTX
Fa0/9                        disabled     999        auto    auto  10/100BaseTX
Fa0/10                       disabled     999        auto    auto  10/100BaseTX
Fa0/11                       disabled     999        auto    auto  10/100BaseTX
Fa0/12                       disabled     999        auto    auto  10/100BaseTX
Fa0/13                       disabled     999        auto    auto  10/100BaseTX
Fa0/14                       disabled     999        auto    auto  10/100BaseTX
Fa0/15                       disabled     999        auto    auto  10/100BaseTX
Fa0/16                       disabled     999        auto    auto  10/100BaseTX
Fa0/17                       disabled     999        auto    auto  10/100BaseTX
Fa0/18    Link to PC-B       connected    10         auto    auto  10/100BaseTX
Fa0/19                       disabled     999        auto    auto  10/100BaseTX
Fa0/20                       disabled     999        auto    auto  10/100BaseTX
Fa0/21                       disabled     999        auto    auto  10/100BaseTX
Fa0/22                       disabled     999        auto    auto  10/100BaseTX
Fa0/23                       disabled     999        auto    auto  10/100BaseTX
Fa0/24                       disabled     999        auto    auto  10/100BaseTX
Gig0/1                       disabled     999        auto    auto  10/100BaseTX
Gig0/2                       disabled     999        auto    auto  10/100BaseTX
```
#### Шаг 4. Документирование и реализация функций безопасности порта.
Интерфейсы F0/6 на S1 и F0/18 на S2 настроены как порты доступа. На этом шаге мы также настраиваем безопасность портов на этих двух портах доступа.    
На S1, вводим команду ***show port-security interface f0/6***  для отображения настроек по умолчанию безопасности порта для интерфейса F0/6. 

|Конфигурация безопасности порта по умолчанию|
|--------------------------------------------|

|Функция|	Настройка по умолчанию|
|-------|-------------|
|Защита портов|Отключена (Disabled)  |	
|Максимальное количество записей MAC-адресов| 1   |
|Режим проверки на нарушение безопасности	|Shutdown (порт отключается при нарушении)|	
|Aging Time	|0 минут (время старения не задано)|	
|Aging Type	|Absolute (абсолютный тип старения: таймер отсчитывает время с момента добавления MAC‑адреса)|	
|Secure Static Address Aging	|Отключено (Disabled)|	
|Sticky MAC Address	|0 (Sticky MAC‑адреса не настроены и не изучены)|	

На S1 включаем защиту порта на F0 / 6 со следующими настройками:   
-	Максимальное количество записей MAC-адресов: 3
-	Режим безопасности: restrict
-	Aging time: 60 мин.
-	Aging type: неактивный
```
interface fa0/6
switchport port-security
switchport port-security maximum 3
switchport port-security violation restrict
switchport port-security aging time 60
switchport port-security aging type inactivity - команда отсутствует в СРТ
```
Проверяем настройки безопасности на интерфейсе F0/6 коммутатора S1.
```
S1# show port-security interface f0/6
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 3
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 000D.BD0D.574D:10
Security Violation Count   : 0
```
Выполняем команду ***show port-security address*** на S1:
```
               Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports   Remaining Age
                                                                   (mins)
----    -----------       ----                          -----   -------------
10	000D.BD0D.574D	DynamicConfigured	FastEthernet0/6		-
-----------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 1024
```
Включаем безопасность порта для F0/18 на S2. Настраиваем каждый активный порт доступа таким образом, чтобы он автоматически добавлял адреса МАС, изученные на этом порту, в текущую конфигурацию.   
Настраиваем следующие параметры безопасности порта на S2 F0/18:
-	Максимальное количество записей MAC-адресов: 2
-	Тип безопасности: Protect
-	Aging time: 60 мин.
```
interface fastethernet 0/18
switchport port-security
switchport port-security mac-address sticky
switchport port-security maximum 2
switchport port-security violation protect
switchport port-security aging time 60
```
Проверка функции безопасности портов на S2 F0/18:   
```
S2# show port-security interface f0/18
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Protect
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 2
Total MAC Addresses        : 0
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0000.0000.0000:0
Security Violation Count   : 0
```
Выполняем команду ***show port-security address*** на S2:
```
               Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports   Remaining Age
                                                                   (mins)
----    -----------       ----                          -----   -------------
  10    0005.5E08.1933    SecureSticky                  Fa0/18       -
-----------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 1024
```
#### Шаг 5. Реализовать безопасность DHCP snooping.
На S2 включаем DHCP snooping и настраиваем DHCP snooping во VLAN 10.     
Настраиваем магистральные порты на S2 как доверенные порты.     
Ограничиваем ненадежный порт Fa0/18 на S2 пятью DHCP-пакетами в секунду.    
Отключаем опцию 82 - добавляет в DHCP‑запросы информацию о порте и коммутаторе.
```
ip dhcp snooping
ip dhcp snooping vlan 10
interface fa0/1
ip dhcp snooping trust
!
interface fa0/18
ip dhcp snooping limit rate 5
!
no ip dhcp snooping information option
```
Проверяем DHCP Snooping на S2:
```
S2# show ip dhcp snooping
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
10
Insertion of option 82 is enabled
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled
Interface                  Trusted    Rate limit (pps)
-----------------------    -------    ----------------
FastEthernet0/1            yes        unlimited       
FastEthernet0/18           no         5
```
В командной строке на PC-B освобождаем, а затем обновляем IP-адрес.   
```
C:\Users\Student> ipconfig /release
C:\Users\Student> ipconfig /renew
```
Проверяем привязку отслеживания DHCP с помощью команды ***show ip dhcp snooping binding***.
```
S2# show ip dhcp snooping binding 
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  -----------------
00:05:5E:08:19:33   192.168.10.11    0           dhcp-snooping  10    FastEthernet0/18
Total number of bindings: 1
```
#### Шаг 6. Реализация PortFast и BPDU Guard
Настраиваем PortFast на всех портах доступа, которые используются на обоих коммутаторах.    
```
S1:
interface range fa0/5 - 6
spanning-tree portfast
S2:
interface fa0/18
spanning-tree portfast
```
Включаем защиту BPDU на портах доступа VLAN 10 S1 и S2, подключенных к PC-A и PC-B.
```
S1:
interface fa0/6
spanning-tree bpduguard enable

S2:
interface fa0/18
spanning-tree bpduguard enable
```
Убеждаемся, что защита BPDU и PortFast включены на соответствующих портах.
```
S1# show spanning-tree interface f0/6 detail
Port 6 (FastEthernet0/6) of VLAN0010 is designated forwarding
  Port path cost 19, Port priority 128, Port Identifier 128.6
  Designated root has priority 32778, address 0001.4371.0EB0
  Designated bridge has priority 32778, address 0009.7C75.9D80
  Designated port id is 128.6, designated path cost 19
  Timers: message age 16, forward delay 0, hold 0
  Number of transitions to forwarding state: 1
  The port is in the portfast mode
  Link type is point-to-point by default
```
Водим команду ***show running-config*** и смотрим на настройки интерфейса 0/6:
```
interface FastEthernet0/6
 description Link to PC-A
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security maximum 3
 switchport port-security violation restrict 
 switchport port-security aging time 60
 spanning-tree portfast
 spanning-tree bpduguard enable
```
#### Шаг 7. Проверьте наличие сквозного ⁪подключения.
Проверяем PING свзяь между всеми устройствами в таблице IP-адресации. Отмечаем наличие сквозного подключения между устройствами. 
