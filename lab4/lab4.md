# Настройка IPv6-адресов на сетевых устройствах
### Задачи:
1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора
2. Ручная настройка IPv6-адресов
3. Проверка сквозного соединения
### Топология 
![](топология.png)
### Таблица адресации
| Устройство  | Интерфейс | IPv6-адрес   | Link local IPv6-адрес | Длина префикса | Шлюз по умолчанию | 
|-------------|-----------|--------------|-----------------------|----------------|-------------------|
| R1          | G0/0/0    |2001:db8:acad:a::1| fe80::1           |64              |—                  |
|             | G0/0/1    |2001:db8:acad:1::1| fe80::1           |64              |—                  |
| S1          | VLAN 1    |2001:db8:acad:1::b| fe80::b           |64              |—                  |
| PC-A        | NIC       |2001:db8:acad:1::3| SLACC             |64              |fe80::1            |
| PC-B        | NIC       |2001:db8:acad:a::3| SLACC             |64              |fe80::1            |

### 1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора
#### Шаг 1. Настраиваем маршрутизатор
Входим в привилегированный режим.    
Входим в режим глобальной конфигурации.   
Отключаем интерпретацию команды как DNS имя - на случай ввода команды с ошибкой.   
Задаем имя маршрутизатора.   
Включаем шифрование паролей.   
Устанавливаем пароль для доступа к маршрутизатору через консольный кабель и включаем доступ к пользовательскому режиму.   
Устанавливаем локальный пароль доступа в привилегированный режим консоли.   
Задаем баннерное сообщение при входе в систему.
```
enable
configure terminal
no ip domain-lookup
hostname R1
service password-encryption
line console 0
password cisco
login
enable secret class
banner motd @--- Unauthorized access is strictly prohibited ---@
```
#### Шаг 2. Настраиваем коммутатор
Входим в привилегированный режим.    
Входим в режим глобальной конфигурации.   
Отключаем интерпретацию команды как DNS имя - на случай ввода команды с ошибкой.   
Задаем имя коммутатора.   
Включаем шифрование паролей.   
Устанавливаем пароль для доступа к коммутатору через консольный кабель и включаем доступ к пользовательскому режиму.   
Устанавливаем локальный пароль доступа в привилегированный режим консоли.   
Задаем баннерное сообщение при входе в систему.
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
```
### 2. Ручная настройка IPv6-адресов
#### Шаг 1. Назначаем IPv6-адреса интерфейсам Ethernet на R1.
Назначаем IPv6 адреса на интерфейсы.
```
interface G0/0/0
ipv6 address 2001:db8:acad:a::1/64
ipv6 address fe80::1 link-local
no shutdown
```
Повторяем то же самое для интерфейса G0/0/1.    
Проверяем правильность настроек IPv6 адресов с помощью команды ***show ipv6 interface brief***. Отмечаем, что для каждого интерфейса установлены IPv6 адреса и Link-local адреса. 
#### Шаг 2. Активируем IPv6-маршрутизацию на R1.
В командной строке на PC-B вводим команду ***ipconfig***, чтобы получить данные IPv6-адреса, назначенного интерфейсу ПК. Отмечаем, что IPv6 адрес отсутствует.
Активируем IPv6-маршрутизацию на R1 с помощью команды ***IPv6 unicast-routing***     
Еще раз вводим команду ***ipconfig*** на на PC-B и убеждаемся, что компьютер получил IPv6-адрес и адрес шлюза по умолчанию с помощью функции SLAAC.    
#### Шаг 3. Назначаем IPv6-адрес интерфейсу управления (SVI) на S1.
Для включения IPv6 на коммутаторе необходимо запустить шаблон, поддерживающий IPv6 и перезапустить устройство.   
```
sdm prefer dual-ipv4-and-ipv6 default
reload
```
Назначаем IPv6 адреса на интерфейс VLAN1.
```
interfase vlan1
ipv6 address 2001:db8:acad:1::b/64
ipv6 address fe80::b link-local
no shutdown
```
Проверяем правильность настроек IPv6 адреса с помощью команды ***show ipv6 interface vlan1***. Отмечаем, что для интерфейса установлен Global unicast адрес и Link-local адрес.     
#### Шаг 4. Назначаем компьютерам статические IPv6-адреса.
Открываем окно Свойства Ethernet для каждого ПК и назначаем IPv6-адреса и адреса шлюзов по умолчанию согласно таблицы адресации.    
Отмечаем, что эхо-запросы проходят между двумя ПК и при статических, и при SLACC IPv6 адресах.
### 3. Проверка сквозного подключения
С PC-A отправляем эхо-запрос на FE80::1. Это локальный адрес канала, назначенный G0/1 на R1.
```
Pinging fe80::1 with 32 bytes of data:
Reply from FE80::1: bytes=32 time<1ms TTL=255
Reply from FE80::1: bytes=32 time<1ms TTL=255
Reply from FE80::1: bytes=32 time<1ms TTL=255
Reply from FE80::1: bytes=32 time<1ms TTL=255
Ping statistics for FE80::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
Отправляем эхо-запрос на интерфейс управления S1 с PC-A.
```
Pinging 2001:db8:acad:1::b with 32 bytes of data:
Reply from 2001:DB8:ACAD:1::B: bytes=32 time=2011ms TTL=255
Reply from 2001:DB8:ACAD:1::B: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:1::B: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:1::B: bytes=32 time<1ms TTL=255
Ping statistics for 2001:DB8:ACAD:1::B:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 2011ms, Average = 502ms
```
Вводим команду tracert на PC-A, чтобы проверить наличие сквозного подключения к PC-B.
```
C:\>tracert 2001:db8:acad:a::3
Tracing route to 2001:db8:acad:a::3 over a maximum of 30 hops: 
  1   0 ms      2 ms      1 ms      2001:DB8:ACAD:1::1
  2   0 ms      0 ms      0 ms      2001:DB8:ACAD:A::3
Trace complete.
```
С PC-B отправляем эхо-запрос на PC-A.
```
Pinging 2001:DB8:ACAD:1::3 with 32 bytes of data:
Reply from 2001:DB8:ACAD:1::3: bytes=32 time<1ms TTL=127
Reply from 2001:DB8:ACAD:1::3: bytes=32 time<1ms TTL=127
Reply from 2001:DB8:ACAD:1::3: bytes=32 time<1ms TTL=127
Reply from 2001:DB8:ACAD:1::3: bytes=32 time<1ms TTL=127
Ping statistics for 2001:DB8:ACAD:1::3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
С PC-B отправляем эхо-запрос на локальный адрес канала G0/0 на R1.
```
Pinging 2001:DB8:ACAD:a::1 with 32 bytes of data:
Reply from 2001:DB8:ACAD:A::1: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:A::1: bytes=32 time=3ms TTL=255
Reply from 2001:DB8:ACAD:A::1: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:A::1: bytes=32 time<1ms TTL=255
Ping statistics for 2001:DB8:ACAD:A::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 3ms, Average = 0ms
```

