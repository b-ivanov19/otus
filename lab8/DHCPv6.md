# Реализация DHCPv6
### Топология
![](топология1.png)
### Таблица адресации
| Устройство | Интерфейс | IPv6‑адрес |
| --- | --- | --- |
| R1 | G0/0/0 | 2001:db8:acad:2::1/64 |
| R1 | G0/0/0 | fe80::1 |
| R1 | G0/0/1 | 2001:db8:acad:1::1/64 |
| R1 | G0/0/1 | fe80::1 |
| R2 | G0/0/0 | 2001:db8:acad:2::2/64 |
| R2 | G0/0/0 | fe80::2 |
| R2 | G0/0/1 | 2001:db8:acad:3::1/64 |
| R2 | G0/0/1 | fe80::1 |
| PC‑A | NIC | DHCP |
| PC‑B | NIC | DHCP |
### Задачи
1. Создание сети и настройка основных параметров устройства
2. Проверка назначения адреса SLAAC от R1
3. Настройка и проверка сервера DHCPv6 на R1
4. Настройка и проверка состояния DHCPv6 сервера на R1
5. Настройка и проверка DHCPv6 Relay на R2
### 1. Создание сети и настройка основных параметров устройства
#### Шаг 1. Создайте сеть согласно топологии.
Подключаем устройства согласно топологии, и подсоединяем необходимые кабели.
#### Шаг 2. Настройте базовые параметры каждого коммутатора
Входим в привилегированный режим.    
Входим в режим глобальной конфигурации.   
Отключаем интерпретацию команды как DNS имя - на случай ввода команды с ошибкой.   
Задаем имя коммутатора.   
Включаем шифрование паролей.   
Устанавливаем пароль для доступа к коммутатору через консольный кабель и включаем доступ к пользовательскому режиму.   
Устанавливаем локальный пароль доступа в привилегированный режим консоли.   
Задаем баннерное сообщение при входе в систему.    
Настраиваем на коммутаторе время.    
Отключаем все неиспользуемые порты.    
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
clock set 14:15:00 18 feb 2026
interface range fastEthernet 0/1-4, fastEthernet 0/7-24, gigabitEthernet 0/1-2
shutdown
exit
copy running-config startup-config
```
**Повторяем процедуру для второго коммутатора.**
#### Шаг 3. Произведите базовую настройку маршрутизаторов.
Подключаемся к маршрутизатору с помощью консоли.    
Входим в привилегированный режим.    
Входим в режим глобальной конфигурации.   
Задаем имя маршрутизатору.   
Отключаем интерпретацию команды как DNS имя - на случай ввода команды с ошибкой.    
Включаем шифрование паролей.   
Устанавливаем пароль для доступа к маршрутизатору через консольный кабель и включаем доступ к пользовательскому режиму.   
Устанавливаем локальный пароль доступа в привилегированный режим консоли.   
Устанавливаем пароль VTY и включаем вход в систему по паролю.    
Задаем баннерное сообщение при входе в систему.     
Активируем IPv6-маршрутизацию.     
Сохраняем текущую конфигурацию в файл загрузочной конфигурации.    
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
ipv6 unicast-routing
exit
copy running-config startup-config
```
**Повторяем процедуру для второго маршрутизатора.**
#### Шаг 4. Настройка интерфейсов и маршрутизации для обоих маршрутизаторов.
Настраиваем интерфейсы G0/0/0 и G0/0/1 на R1 и R2 с адресами IPv6, указанными в таблице выше.     
Настраиваем маршрут по умолчанию на каждом маршрутизаторе, который указывает на IP-адрес G0/0/0 на другом маршрутизаторе.
**Для R1:**
```
interface GigabitEthernet 0/0/0
ipv6 address 2001:db8:acad:2::1/64
ipv6 address fe80::1 link-local
no shutdown
exit
interface GigabitEthernet 0/0/1
ipv6 address 2001:db8:acad:1::1/64
ipv6 address fe80::1 link-local
no shutdown
exit
ipv6 route 2001:db8:acad:3::/64 2001:db8:acad:2::2
```
**Для R2:**
```
interface GigabitEthernet 0/0/0
ipv6 address 2001:db8:acad:2::2/64
ipv6 address fe80::2 link-local
no shutdown
exit
interface GigabitEthernet 0/0/1
ipv6 address 2001:db8:acad:3::1/64
ipv6 address fe80::1 link-local
no shutdown
exit
ipv6 route 2001:db8:acad:1::/64 2001:db8:acad:2::1
```
Убедились, что маршрутизация работает с помощью пинга адреса G0/0/1 R2 из R1
```
Sending 5, 100-byte ICMP Echos to 2001:db8:acad:3::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```
Сохраняем текущую конфигурацию в файл загрузочной конфигурации с помощью команды ***copy running-config startup-config***.
### 2. Проверка назначения адреса SLAAC от R2
Включаем PC-B и убеждаемся, что сетевой адаптер настроен для автоматической настройки IPv6.   
Смотрим сетевые параметры PC-B с помощью команды ***ipconfig***. Компьютер присвоил себе IPv6 адрес. 
```
Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::260:47FF:FE98:45EE
   IPv6 Address....................: 2001:DB8:ACAD:3:260:47FF:FE98:45EE
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
```
Часть адреса с идентификатором хоста была рассчитана с использованием MAC-адреса сетевой карты хоста.
### 3. Настройка и проверка сервера DHCPv6 на R2
#### Шаг 1. Более подробно изучите конфигурацию PC-B
Выполняем команду ***ipconfig /all*** на PC-B и смотрим на результат:
```
FastEthernet0 Connection:(default port)
   Connection-specific DNS Suffix..: 
   Physical Address................: 0060.4798.45EE
   Link-local IPv6 Address.........: FE80::260:47FF:FE98:45EE
   IPv6 Address....................: 2001:DB8:ACAD:3:260:47FF:FE98:45EE
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-EE-9E-D1-47-00-60-47-98-45-EE
   DNS Servers.....................: ::
                                     0.0.0.0
```
Отмечаем отсутствие DNS-суффикса, также отсутствуют DNS серверы.
#### Шаг 2. Настройте R2 для предоставления DHCPv6 без состояния для PC-B
Создаём пул DHCP IPv6 на R2 с именем R2-STATELESS. В составе этого пула назначаем адрес DNS-сервера как 2001:db8:acad: :1, а имя домена — как stateless.com.
```
ipv6 dhcp pool R2-STATELESS
dns-server 2001:db8:acad::254
domain-name STATELESS.com
```
Настраиваем интерфейс G0/0/1 на R2, чтобы предоставить флаг конфигурации OTHER для локальной сети R2 и указываем только что созданный пул DHCP в качестве ресурса DHCP для этого интерфейса.
```
interface g0/0/1
ipv6 nd other-config-flag 
ipv6 dhcp server R1-STATELESS
```
Сохраняем текущую конфигурацию в файл загрузочной конфигурации с помощью команды ***copy running-config startup-config***.
Запускаем PC-B, проверяем вывод ***ipconfig /all***.
```
FastEthernet0 Connection:(default port)
   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 0060.4798.45EE
   Link-local IPv6 Address.........: FE80::260:47FF:FE98:45EE
   IPv6 Address....................: 2001:DB8:ACAD:3:260:47FF:FE98:45EE
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 1520237486
   DHCPv6 Client DUID..............: 00-01-00-01-EE-9E-D1-47-00-60-47-98-45-EE
   DNS Servers.....................: 2001:DB8:ACAD::254
                                     0.0.0.0
```
Видим, что появился как DNS-суффикс, так и DNS сервер.
Тестируем подключение с помощью пинга IP-адреса интерфейса G0/0/1 R1 c PC-B.
```
C:\>ping 2001:db8:acad:1::1
Pinging 2001:db8:acad:1::1 with 32 bytes of data:
  Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
  Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
  Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
  Reply from 2001:DB8:ACAD:1::1: bytes=32 time<1ms TTL=254
Ping statistics for 2001:DB8:ACAD:1::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
```
### 4. Настройка сервера DHCPv6 с сохранением состояния на R2
Создаем пул DHCPv6 на R2 для сети 2001:db8:acad:3:aaa::/80. Это предоставит адреса локальной сети, подключенной к интерфейсу G0/0/1 на R1. В составе пула задаем DNS-сервер 2001:db8:acad: :254 и задаем доменное имя STATEFUL.com.
```
ipv6 dhcp pool R2-STATEFUL
address prefix 2001:db8:acad:3:aaa::/80
dns-server 2001:db8:acad::254
domain-name STATEFUL.com
```
Назначаем только что созданный пул DHCPv6 интерфейсу G0/0/0 на R2.
```
interface g0/0/0
ipv6 dhcp server R1-STATEFUL
```
### 5. Настройка и проверка ретрансляции DHCPv6 на R1.
#### Шаг 1. Включите PC-A и проверьте адрес SLAAC, который он генерирует
Используем команду ***ipconfig /all***
```
FastEthernet0 Connection:(default port)
   Connection-specific DNS Suffix..: 
   Physical Address................: 0010.11EE.70E3
   Link-local IPv6 Address.........: FE80::210:11FF:FEEE:70E3
   IPv6 Address....................: 2001:DB8:ACAD:1:700A:4733:1D7D:FEB
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-DC-48-57-22-00-0D-BD-E4-49-74
   DNS Servers.....................: ::
                                     0.0.0.0
```
Обращаем внимание на вывод, что используется префикс 2001:db8:acad:3::
#### Шаг 2. Настройте R1 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1
В условиях невозможности настроить DHCP relay в Cisco Packet Tracer, развернем на R1 собственный DHCP-сервер для PC-A.
```
ipv6 dhcp pool R1-LAN-POOL
dns-server 2001:db8:acad::254
domain-name STATELESS.com
exit
int g0/0/1
ipv6 nd managed-config-flag 
ipv6 nd other-config-flag 
ipv6 dhcp server R1-lan
```
#### Шаг 3. Получаем IPv6 адрес из DHCPv6 на PC-A
Открываем командную строку на PC-A и выполняем команду ***ipconfif /all***, проверяем выходные данные.
```
FastEthernet0 Connection:(default port)
   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 0010.11EE.70E3
   Link-local IPv6 Address.........: FE80::210:11FF:FEEE:70E3
   IPv6 Address....................: 2001:DB8:ACAD:1:700A:4733:1D7D:FEB
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 1176899032
   DHCPv6 Client DUID..............: 00-01-00-01-27-4D-2B-59-00-10-11-EE-70-E3
   DNS Servers.....................: 2001:DB8:ACAD::254
                                     0.0.0.0
```
Проверяем подключение с помощью пинга IP-адреса интерфейса R2 G0/0/1
```
C:\>ping 2001:db8:acad:3::1
Pinging 2001:db8:acad:3::1 with 32 bytes of data:
  Reply from 2001:DB8:ACAD:3::1: bytes=32 time=1ms TTL=254
  Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
  Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
  Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Ping statistics for 2001:DB8:ACAD:3::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```
Также мы отмечаем, что проходят эхо-запросы между PC-A и PC-B
```
C:\>ping 2001:DB8:ACAD:3:260:47FF:FE98:45EE
Pinging 2001:DB8:ACAD:3:260:47FF:FE98:45EE with 32 bytes of data:
  Reply from 2001:DB8:ACAD:3:260:47FF:FE98:45EE: bytes=32 time<1ms TTL=126
  Reply from 2001:DB8:ACAD:3:260:47FF:FE98:45EE: bytes=32 time<1ms TTL=126
  Reply from 2001:DB8:ACAD:3:260:47FF:FE98:45EE: bytes=32 time<1ms TTL=126
  Reply from 2001:DB8:ACAD:3:260:47FF:FE98:45EE: bytes=32 time<1ms TTL=126
Ping statistics for 2001:DB8:ACAD:3:260:47FF:FE98:45EE:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
