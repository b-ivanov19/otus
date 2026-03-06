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


Часть 3. Настройки безопасности коммутатора.
Шаг 1. Релизация магистральных соединений 802.1Q.
a.	Настройте все магистральные порты Fa0/1 на обоих коммутаторах для использования VLAN 333 в качестве native VLAN.
b.	Убедитесь, что режим транкинга успешно настроен на всех коммутаторах.
S1# show interface trunk

Port Mode Encapsulation Status Native vlan
Fa0/1 on 802.1q trunking 333

Port Vlans allowed on trunk
Fa0/1 1-4094

Port Vlans allowed and active in management domain
Fa0/1 1,10,333,999

Port Vlans in spanning tree forwarding state and not pruned
Fa0/1 1,10,333,999

S2# show interface trunk

Port Mode Encapsulation Status Native vlan
Fa0/1 on 802.1q trunking 333

Port Vlans allowed on trunk
Fa0/1 1-4094

Port Vlans allowed and active in management domain
Fa0/1 1,10,333,999

Port Vlans in spanning tree forwarding state and not pruned
Fa0/1 1,10,333,999
c.	Отключить согласование DTP F0/1 на S1 и S2. 
d.	Проверьте с помощью команды show interfaces.
S1# show interfaces f0/1 switchport | include Negotiation
Negotiation of Trunking: Off

S1# show interfaces f0/1 switchport | include Negotiation
Negotiation of Trunking: Off
Шаг 2. Настройка портов доступа
a.	На S1 настройте F0/5 и F0/6 в качестве портов доступа и свяжите их с VLAN 10.
b.	На S2 настройте порт доступа Fa0/18 и свяжите его с VLAN 10.
Шаг 3. Безопасность неиспользуемых портов коммутатора
a.	На S1 и S2 переместите неиспользуемые порты из VLAN 1 в VLAN 999 и отключите неиспользуемые порты.
b.	Убедитесь, что неиспользуемые порты отключены и связаны с VLAN 999, введя команду  show.
S1# show interfaces status

Port Name Status Vlan Duplex Speed Type
Fa0/1 Link to S2 connected trunk a-full a-100 10/100BaseTX
Fa0/2 disabled 999 auto auto 10/100BaseTX
Fa0/3 disabled 999 auto auto 10/100BaseTX
Fa0/4 disabled 999 auto auto 10/100BaseTX
Fa0/5 Link to R1 connected 10 a-full a-100 10/100BaseTX
Fa0/6 Link to PC-A connected 10 a-full a-100 10/100BaseTX
Fa0/7 disabled 999 auto auto 10/100BaseTX
Fa0/8 disabled 999 auto auto 10/100BaseTX
Fa0/9 disabled 999 auto auto 10/100BaseTX
Fa0/10 disabled 999 auto auto 10/100BaseTX
<output omitted>
S2# show interfaces status

Port Name Status Vlan Duplex Speed Type
Fa0/1 Link to S1 connected trunk a-full a-100 10/100BaseTX
Fa0/2 disabled 999 auto auto 10/100BaseTX
Fa0/3 disabled 999 auto auto 10/100BaseTX
<output omitted>
Fa0/14 disabled 999 auto auto 10/100BaseTX
Fa0/15 disabled 999 auto auto 10/100BaseTX
Fa0/16 disabled 999 auto auto 10/100BaseTX
Fa0/17 disabled 999 auto auto 10/100BaseTX
Fa0/18 Link to PC-B connected 10 a-full a-100 10/100BaseTX
Fa0/19 disabled 999 auto auto 10/100BaseTX
Fa0/20 disabled 999 auto auto 10/100BaseTX
Fa0/21 disabled 999 auto auto 10/100BaseTX
Fa0/22 disabled 999 auto auto 10/100BaseTX
Fa0/23 disabled 999 auto auto 10/100BaseTX
Fa0/24 disabled 999 auto auto 10/100BaseTX
Gi0/1 disabled 999 auto auto 10/100/1000BaseTX
Gi0/2 disabled 999 auto auto 10/100/1000BaseTX






