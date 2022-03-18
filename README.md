# Ебля с песнями, начинается!

## Видео №1. Базовая адресация и настройка

#### WEB-L, WEB-R, ISP 
```
Заходим в репозиторий дисков [images] --> debian (там будет 4 диска, можно сразу все добавить)
ВНИМАНИЕ! ВКЛЮЧИТЕ ПЕРВЫЙ ДИСК, А ТО ОН МОЖЕТ НЕ РАБОТАТЬ


apt-cdrom add
apt -y install mc network-manager
```


#### WEB-L
```
nmtui

Address: 192.168.100.100/24
Gateway: 192.168.100.254

Перезапускаем соединение, переименуем компьютер
```


#### WEB-R
```
nmtui

Address: 172.16.100.100/24
Gateway: 172.16.100.254

Перезапускаем соединение, переименуем компьютер
```


#### ISP
```
nmtui

Тут у нас есть 3 соединения, начинаем с 1 до 3 (снизу вверх)

1: Address: 3.3.3.1/24
2: Address: 4.4.4.1/24
3: Address: 5.5.5.1/24

Перезапускаем соединения, переименуем компьютер
```


#### SRV (тут винда, разжимаем булки)
```
Windows + R: scnofig

Переходим в 8 пункт, выбираем наш интерфейс, статика (S)
IP: 192.168.100.200
Mask: (пропускаем)
Gateway: 192.168.100.254

2 пункт, переименуем компьютер и перезагружаем его.
```


#### RTR-R (тут циско, сжимаем булки. Пишем внимательно все.)
```
en
conf t
hostname RTR-R
interface gi1
  ip address 5.5.5.100 255.255.255.0
  no sh
  exit
interface gi2
  ip address 172.16.100.254 255.255.255.0
  no sh
  exit
do wr
```



#### RTR-L (думали все? Не разжимаем булки, пишем дальше)
```
en
conf t
hostname RTR-L
interface gi1
  ip address 4.4.4.100 255.255.255.0
  no sh
  exit
interface gi2
  ip address 192.168.100.254 255.255.255.0
  no sh
  exit
do wr
``` 


#### SRV (тут винда, разжимаем булки)
```
Windows + R: ncpa.cpl

Идем в свойства подключения, настройка IPV4!

IP Address: 3.3.3.10
Mask: 255.255.255.0
Gateway: 3.3.3.1

Нажимаем Yes, когда выскачит справа предупреждение
```


## Видео №2. Сетевая связанность
Если вы уже тут, то поздравляю. Ваш IQ больше 133, однако дальше будет моральный террор и убийства. Поехали!


#### ISP
```
mcedit /etc/sysctl.conf
net.ipv4.ip_forward=1 — НАДО УБРАТЬ #
F2
F10

sysctl -p
```


#### RTR-R
```
ip route 0.0.0.0 0.0.0.0 gi1
do wr
```

#### RTR-L
```
ip route 0.0.0.0 0.0.0.0 gi1
do wr

int tun1
ip address 172.16.1.1 255.255.255.0
tunnel mode gre ip
tunnel source 4.4.4.100
tunnel destination 5.5.5.100
no sh
exit
do wr
```


#### RTR-R
```
int tun1
ip address 172.16.1.2 255.255.255.0
tunnel mode gre ip
tunnel source 5.5.5.100
tunnel destination 4.4.4 .100
no sh
exit
do wr


router eigrp 6500
network 172.16.100.0 0.0.0.255
network 172.16.1.0 0.0.0.255
do wr
exit
```


#### RTR-L
```
router eigrp 6500
network 192.168.100.0 0.0.0.255
network 172.16.1.0 0.0.0.255 
do wr
exit
```


#### RTR-R
```
crypto isakmp policy 1
encryption aes
authentication pre-share
group 15
hash sha 
exit
crypto isakmp key Cisco address 4.4.4.100
crypto ipsec transform-set CTI esp-aes esp-sha-hmac
mode tunnel
exit
crypto ipsec profile CTP
set transfrom-set CTI
exit
do wr
```




#### RTR-L
```
crypto isakmp policy 1
encryption aes
authentication pre-share
group 15
hash sha 
exit
crypto isakmp key Cisco address 5.5.5.100
crypto ipsec transform-set CTI esp-aes esp-sha-hmac
mode tunnel
exit
crypto ipsec profile CTP
set transfrom-set CTI
exit
do wr
int tunnel 1
tunnel mode ipsec ipv4 
tunnel protection ipsec profile CTP 
do wr
exit
```


#### RTR-R
```
int tunnel 1
tunnel mode ipsec ipv4 
tunnel protection ipsec profile CTP 
do wr
exit
```


#### RTR-R
```
access-list 1 permit 172.16.100.0 0.0.0.255
ip nat inside source list 1 interface gi 1
do wr
```


#### RTR-L
```
access-list 1 permit 192.168.100.0 0.0.0.255
ip nat inside source list 1 interface gi 1
do wr
```
