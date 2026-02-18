# Реализация DHCPv4 
### Топология
![](топология1.png)
### Таблица адресации
| Устройство  | Интерфейс | IP-адрес   | Маска подсети |Шлюз по умолчанию|
|-------------|-----------|------------|---------------|-----------------|
|R1           |G0/0/0     |10.0.0.1    |255.255.255.252| —            |
|R1           |G0/0/1     | —          | —             | —             |
|R1           |G0/0/1.100 |            |               | —           |
|R1           |G0/0/1.200 |            |               | —           |
|R1           |G0/0/1.1000| —          | —             | —           |
|R2           |G0/0/0     |10.0.0.2    |255.255.255.252| —            |
|R2           |G0/0/1     |            |               | —             |
|S1           |VLAN200    |            |               |              |
|S2           |VLAN1    |            |               |              |
| PC-A        | NIC       | DHCP  | DHCP    |DHCP  |
| PC-B        | NIC       | DHCP  | DHCP    |DHCP  |
### Таблица VLAN
| VLAN | Имя          | Назначенный интерфейс               |
|------|--------------|-------------------------------------|
| 1    | Нет          | S2: F0/18                         |
| 100  | Клиенты      | S1: F0/6                         |
| 200  | Управление   | S1: VLAN 200                      |
| 999  | Parking_Lot | S1: F0/1-4, F0/7-24, G0/1-2    |
| 1000 | Собственная  | —                                |
### Задачи
1. Создание сети и настройка основных параметров устройства
2. Настройка и проверка двух серверов DHCPv4 на R1
3. Настройка и проверка DHCP-ретрансляции на R2
### 1. Создание сети и настройка основных параметров устройства
#### Шаг 1. Создание схемы адресации
**Подсеть A** должна поддерживать 58 хостов (клиентская VLAN на R1). Будем использовать маску /26 (62 хоста). Записываем в таблицу адресации первый IP-адрес для R1 G0/0/1.100.     
**Подсеть В** должна поддерживать 28 хостов (управляющая VLAN на R1). Будем использовать маску /27 (30 хостов). Записываем в таблицу адресации первый IP-адрес для R1 G0/0/1.200, второй IP-адрес для S1 VLAN 200, а также шлюз по умолчанию.     
**Подсеть С** должна поддерживать 12 хостов (клиентская сеть на R2). Будем использовать маску /28 (14 хостов). Записываем в таблицу адресации первый IP-адрес для R2 G0/0/1.
##### Заполненная таблица адресации:
| Устройство | Интерфейс     | IP‑адрес       | Маска подсети        | Шлюз по умолчанию |
|-----------|--------------|----------------|---------------------|------------------|
| R1       | G0/0/0       | 10.0.0.1       | 255.255.255.252    | —                |
| R1       | G0/0/1       | —              | —                   | —                |
| R1       | G0/0/1.100  | 192.168.1.1    | 255.255.255.192    | —                  |
| R1       | G0/0/1.200  | 192.168.1.65   | 255.255.255.224    | —                  |
| R1       | G0/0/1.1000 | —              | —                   | —                |
| R2       | G0/0/0      | 10.0.0.2       | 255.255.255.252    | —                |
| R2       | G0/0/1      | 192.168.1.97   | 255.255.255.240    | —                |
| S1       | VLAN 200     | 192.168.1.66   | 255.255.255.224    | 192.168.1.65     |
| S2       | VLAN 1       | —              | —                   | —                |
| PC‑A     | NIC          | DHCP           | DHCP                | DHCP             |
| PC‑B     | NIC          | DHCP           | DHCP                | DHCP             |
#### Шаг 2. Создание сети согласно топологии
Подключаем устройства, как показано в топологии, и подсоединяем необходимые кабели.
#### Шаг 3. Шаг 3.	Произведите базовую настройку маршрутизаторов
Подключаемся к маршрутизатору с помощью консоли.    
Входим в привилегированный режим.    
Входим в режим глобальной конфигурации.   
Задаем имя маршрутизатору.   
```
enable
configure terminal
hostname R1
```
Отключаем интерпретацию команды как DNS имя - на случай ввода команды с ошибкой.    
Включаем шифрование паролей.   
Устанавливаем пароль для доступа к маршрутизатору через консольный кабель и включаем доступ к пользовательскому режиму.   
Устанавливаем локальный пароль доступа в привилегированный режим консоли.   
Устанавливаем пароль VTY и включаем вход в систему по паролю.    
Задаем баннерное сообщение при входе в систему.    
Настраиваем на маршрутизаторе время.    
Сохраняем текущую конфигурацию в файл загрузочной конфигурации.    
```
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
exit
clock set 13:05:00 17 feb 2026
copy running-config startup-config
```
**Повторяем процедуру для второго маршрутизатора.**
#### Шаг 4.	Настройка маршрутизации между сетями VLAN на маршрутизаторе R1
Активируем интерфейс G0/0/1 на маршрутизаторе.    
```
interface GigabitEthernet0/0/1
no shutdown
```
Настраиваем подинтерфейсы для каждой VLAN в соответствии с требованиями таблицы IP-адресации.      
На всех субинтерфейсах включаем инкапсуляцию 802.1Q.    
Назначаем первый полезный адрес из вычисленного пула IP-адресов.     
Убеждаемся, что подинтерфейсу для native VLAN не назначен IP-адрес.      
Добавляем описание для каждого подинтерфейса.     
```
interface GigabitEthernet0/0/1.100
encapsulation dot1Q 100
ip address 192.168.1.1 255.255.255.192
description Clients

interface GigabitEthernet0/0/1.200
encapsulation dot1Q 200
ip address 192.168.1.65 255.255.255.224
description Management

interface GigabitEthernet0/0/1.1000
encapsulation dot1Q 1000
description Native
```
Убеждаемся, что вспомогательные интерфейсы работают.
```
show ip interface brief 
Interface                 IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0      unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1      unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.100  192.168.1.1     YES manual up                    up 
GigabitEthernet0/0/1.200  192.168.1.65    YES manual up                    up 
GigabitEthernet0/0/1.1000 unassigned      YES unset  up                    up 
Vlan1                     unassigned      YES unset  administratively down down
```
#### Шаг 5.	Настройте G0/0/1 на R2, затем G0/0/0 и статическую маршрутизацию для обоих маршрутизаторов
Настраиваем G0/0/1 на R2 с первым IP-адресом подсети C, рассчитанным ранее.
```
interface GigabitEthernet0/0/1
ip address 192.168.1.97 255.255.255.240
no shutdown
```
Настраиваем	интерфейс G0/0/0 для маршрутизатора R1 на основе таблицы IP-адресации.
```
interface GigabitEthernet0/0/0
ip address 10.0.0.1 255.255.255.252
no shutdown
```
Настраиваем	интерфейс G0/0/0 для маршрутизатора R2 на основе таблицы IP-адресации.
```
interface GigabitEthernet0/0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
```
Настраиваем маршрут по умолчанию на каждом маршрутизаторе, указываемом на IP-адрес G0/0/0 на другом маршрутизаторе.
```
R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2

R2(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1
```
Убеждаемся, что статическая маршрутизация работает. Посылаем пинг от R1 до адреса G0/0/1 R2 (192.168.1.97).
```
R1#ping 192.168.1.97
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms
```
Сохраняем текущую конфигурацию в файл загрузочной конфигурации с помощью команды ***copy running-config startup-config***.
#### Шаг 6.	Настройте базовые параметры каждого коммутатора.
Входим в привилегированный режим.    
Входим в режим глобальной конфигурации.   
Отключаем интерпретацию команды как DNS имя - на случай ввода команды с ошибкой.   
Задаем имя коммутатора.   
Включаем шифрование паролей.   
Устанавливаем пароль для доступа к коммутатору через консольный кабель и включаем доступ к пользовательскому режиму.   
Устанавливаем локальный пароль доступа в привилегированный режим консоли.   
Задаем баннерное сообщение при входе в систему.    
Настраиваем на коммутаторе время.    
Сохраняем текущую конфигурацию в качестве начальной конфигурации.    
```
enable
configure terminal
no ip domain-lookup
hostname S1
service password-encryption
line console 0
password cisco
login
enable secret class
banner motd @--- Unauthorized access is strictly prohibited ---@
exit
clock set 13:15:00 17 feb 2026
copy running-config startup-config
```
**Повторяем процедуру для второго коммутатора.**
#### Шаг 7.	Создайте сети VLAN на коммутаторе S1.
Создаем необходимые VLAN на коммутаторе S1 и присваиваем им имена из приведенной выше таблицы.
```
vlan 100
name Clients
vlan 200
name Management
vlan 999
name Parking_Lot
vlan 1000
name Native
```
Настраиваем и активируем интерфейс управления на S1 (VLAN 200), используя второй IP-адрес из подсети. Устанавливаем шлюз по умолчанию на S1.
```
interface vlan 200
ip address 192.168.1.66 255.255.255.224
no shutdown 
exit
ip default-gateway 192.168.1.65
```
Настраиваем и активируем интерфейс управления на S2 (VLAN 1), используя второй IP-адрес из подсети, рассчитанный ранее.Устанавливаем шлюз по умолчанию на S2.
```
interface vlan 1
ip address 192.168.1.98 255.255.255.240
no shutdown 
exit
ip default-gateway 192.168.1.97
```
Назначаем все неиспользуемые порты S1 VLAN Parking_Lot, настраиваем их для статического режима доступа и административно деактивируем их. 
```
interface range fastEthernet 0/1-4, fastEthernet 0/7-24, gigabitEthernet 0/1-2
switchport mode access 
switchport access vlan 999
shutdown
```
На S2 административно деактивируем все неиспользуемые порты.
```
interface range fastEthernet 0/1-4, fastEthernet 0/6-17, fastEthernet 0/19-24, gigabitEthernet 0/1-2
shutdown
```
#### Шаг 8.	Назначьте сети VLAN соответствующим интерфейсам коммутатора.
a.	Назначьте используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и настройте их для режима статического доступа.    
Коммутатор S1:
```
interface fastEthernet 0/6
switchport mode access 
switchport access vlan 100
no shutdown

interface fastEthernet 0/5
switchport mode access 
switchport access vlan 200
no shutdown
```
Коммутатор S2:
```
interface fastEthernet 0/18
switchport mode access 
switchport access vlan 1
no shutdown
```
Убеждаемся, что VLAN назначены на правильные интерфейсы с помощью команды ***show vlan brief*** на коммутаторе S1.
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    
100  Clients                          active    Fa0/6
200  Management                       active    Fa0/5
999  Parking_Lot                      active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```
Интерфейс F0/5 **на коммутаторе S2** указан в VLAN 1, так как ему не задан другой VLAN. VLAN 1 используется по умолчанию.
#### Шаг 9.	Вручную настройте интерфейс S1 F0/5 в качестве транка 802.1Q.
Изменяем режим порта коммутатора, чтобы принудительно создать магистральный канал.   
В рамках конфигурации транка  устанавливаем для native VLAN значение 1000.    
В качестве другой части конфигурации магистрали указываем, что VLAN 100, 200 и 1000 могут проходить по транку.     
Сохраняем текущую конфигурацию в файл загрузочной конфигурации.     
```
interface fastEthernet 0/5
switchport mode trunk 
switchport trunk native vlan 1000
switchport trunk allowed vlan 100,200,1000
```
Проверяем состояние транка.
```
S1#show interfaces fastEthernet 0/5 switchport 
Name: Fa0/5
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: Off
Access Mode VLAN: 200 (Management)
Trunking Native Mode VLAN: 1000 (Native)
Trunking VLANs Enabled: 100,200,1000
```
***Какой IP-адрес был бы у ПК, если бы он был подключен к сети с помощью DHCP?*** У ПК был бы адрес 169.254... что означает наличие сети, но ответа от DHCP-сервера нет.
### 2. Настройка и проверка двух серверов DHCPv4 на R1
#### Шаг 1.	Настройте R1 с пулами DHCPv4 для двух поддерживаемых подсетей. Ниже приведен только пул DHCP для подсети A
Исключаем первые пять используемых адресов из каждого пула адресов.    
Создаем пул DHCP (используем уникальное имя для каждого пула).    
Указываем сеть, поддерживающую этот DHCP-сервер.     
В качестве имени домена указываем CCNA-lab.com.     
Настраиваем соответствующий шлюз по умолчанию для каждого пула DHCP.     
Настраиваем время аренды на 2 дня 12 часов и 30 минут.    
```
ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp pool R1_Client_LAN
network 192.168.1.0 255.255.255.192
domain-name CCNA-lab.com
default-router 192.168.1.1
lease 2 12 30
```
Настраиваем второй пул DHCPv4, используя имя пула R2_Client_LAN и ранее вычисленную сеть, маршрутизатор по умолчанию, используем то же имя домена и время аренды, что и предыдущий пул DHCP.
```
ip dhcp excluded-address 192.168.1.97 192.168.1.101
ip dhcp pool R2_Client_LAN
network 192.168.1.96 255.255.255.240
domain-name CCNA-lab.com
default-router 192.168.1.97
lease 2 12 30
```
#### Шаг 2.	Сохраните конфигурацию.
Сохраняем текущую конфигурацию в файл загрузочной конфигурации с помощью команды ***copy running-config startup-config***.
#### Шаг 3.	Проверка конфигурации сервера DHCPv4
Чтобы просмотреть сведения о пуле, выполняем команду ***show ip dhcp pool R1_Client_LAN***.
```
Pool R1_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 0
 Excluded addresses             : 1
 Pending event                  : none
 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.1          192.168.1.1      - 192.168.1.62      0    / 1     / 62

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
Выполняем команду ***show ip dhcp binding*** для проверки установленных назначений адресов DHCP.
```
R1#show ip dhcp binding
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      0060.4751.1A80           --                     Automatic
```
#### Шаг 4.	Попытка получить IP-адрес от DHCP на PC-A
Из командной строки компьютера PC-A выполняем команду ipconfig /all.
```
Connection-specific DNS Suffix..: CCNA-lab.com
   Physical Address................: 0060.4751.1A80
   Link-local IPv6 Address.........: FE80::260:47FF:FE51:1A80
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.6
   Subnet Mask.....................: 255.255.255.192
   Default Gateway.................: ::
                                     192.168.1.1
   DHCP Servers....................: 192.168.1.1
```
Проверяем подключение с помощью пинга IP-адреса интерфейса R2 G0/0/1.
```
C:\>ping 192.168.1.97
Pinging 192.168.1.97 with 32 bytes of data:
Request timed out.
Reply from 192.168.1.97: bytes=32 time<1ms TTL=254
Reply from 192.168.1.97: bytes=32 time<1ms TTL=254
Reply from 192.168.1.97: bytes=32 time<1ms TTL=254
Ping statistics for 192.168.1.97:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
### 3. Настройка и проверка DHCP-ретрансляции на R2
#### Шаг 1. Настройка R2 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1
настраиваем команду ip helper-address на G0/0/1, указав IP-адрес G0/0/0 R1.
```
interface GigabitEthernet0/0/1
ip helper-address 10.0.0.1
```
Сохраняем текущую конфигурацию в файл загрузочной конфигурации с помощью команды ***copy running-config startup-config***.
#### Шаг 2.	Попытка получить IP-адрес от DHCP на PC-B
Из командной строки компьютера PC-B выполняем команду ipconfig /all.
```
Connection-specific DNS Suffix..: CCNA-lab.com
   Physical Address................: 0030.F2BD.862C
   Link-local IPv6 Address.........: FE80::230:F2FF:FEBD:862C
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.102
   Subnet Mask.....................: 255.255.255.240
   Default Gateway.................: ::
                                     192.168.1.97
   DHCP Servers....................: 10.0.0.1
```
Проверяем подключение с помощью пинга IP-адреса интерфейса R1 G0/0/1
```
C:\>ping 192.168.1.1
Pinging 192.168.1.1 with 32 bytes of data:
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
Выполняем show ip dhcp binding для R1 для проверки назначений адресов в DHCP.
```
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      0060.4751.1A80           --                     Automatic
192.168.1.102    0030.F2BD.862C           --                     Automatic
```
В итоге мы получили на оба компьютера IP-адреса по технологии DHCP, а также получили сетевую связность между ними.
