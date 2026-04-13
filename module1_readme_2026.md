# <div align="center"><strong>⚙️</strong></div>
# <div align="center"><strong>МОДУЛЬ 1</strong></div>
# <div align="center"><strong>КОД 09.02.06-1-2026</strong></div>

<br/>

> [!WARNING]
> Данный файл собран под задание **Модуля 1** из PDF **«КОД 09.02.06-1-2026 Том 1»**.  
> В отличие от старого шаблона, здесь изменены адресация, SSH-порт, UID пользователя и DNS-записи.

---

## ✔️ Общая логика настройки

Настройка выполняется в такой последовательности:

1. Преднастройка всех устройств  
2. Настройка имени и IP-адресов на каждом устройстве  
3. Обеспечение доступа в Интернет для всех устройств  
4. Создание пользователей  
5. Настройка коммутации и VLAN  
6. Настройка SSH  
7. Настройка туннеля между офисами  
8. Настройка динамической маршрутизации  
9. Настройка DHCP  
10. Настройка DNS  
11. Настройка часового пояса  

---

## ✔️ Принятая схема адресации

### Подсети

| Сеть | Подсеть | Назначение |
|---|---|---|
| ISP-HQ | 172.16.1.0/28 | Канал между ISP и HQ-RTR |
| ISP-BR | 172.16.2.0/28 | Канал между ISP и BR-RTR |
| TUN | 172.16.0.0/30 | GRE-туннель между HQ-RTR и BR-RTR |
| VLAN 100 | 192.168.100.0/27 | Сегмент HQ-SRV |
| VLAN 200 | 192.168.200.0/27 | Сегмент HQ-CLI |
| VLAN 999 | 192.168.99.0/29 | Сеть управления |
| BR-Net | 192.168.0.0/28 | Сегмент BR-SRV |

### Таблица адресации

| Устройство | Интерфейс | IP-адрес | Маска | Шлюз |
|---|---|---:|---|---|
| ISP | ens192 | DHCP | DHCP | DHCP |
| ISP | ens224 | 172.16.1.1 | /28 | — |
| ISP | ens256 | 172.16.2.1 | /28 | — |
| HQ-RTR | ens192 | 172.16.1.2 | /28 | 172.16.1.1 |
| HQ-RTR | ens224.100 | 192.168.100.1 | /27 | — |
| HQ-RTR | ens224.200 | 192.168.200.1 | /27 | — |
| HQ-RTR | ens224.999 | 192.168.99.1 | /29 | — |
| HQ-RTR | gre1 | 172.16.0.1 | /30 | — |
| BR-RTR | ens192 | 172.16.2.2 | /28 | 172.16.2.1 |
| BR-RTR | ens224 | 192.168.0.1 | /28 | — |
| BR-RTR | gre1 | 172.16.0.2 | /30 | — |
| HQ-SRV | ens192 | 192.168.100.2 | /27 | 192.168.100.1 |
| BR-SRV | ens192 | 192.168.0.2 | /28 | 192.168.0.1 |
| HQ-CLI | ens192 | DHCP | /27 | 192.168.200.1 |

---

# ✔️ Задание 0 [Преднастройка]

## На всех устройствах, кроме ISP, задаём DNS для временного выхода в интернет

```bash
nano /etc/resolv.conf
```

```bash
nameserver 77.88.8.8
nameserver 1.1.1.1
```

## На маршрутизаторах ISP, HQ-RTR и BR-RTR включаем маршрутизацию пакетов

```bash
echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
sysctl -p
```

## На всех устройствах проверяем список репозиториев

```bash
nano /etc/apt/sources.list
```

Если есть строка `deb cdrom`, её нужно закомментировать, затем выполнить:

```bash
apt update
```

## Если NetworkManager мешает ручной настройке сети

```bash
systemctl stop NetworkManager
systemctl disable NetworkManager
systemctl mask NetworkManager
```

## Проверка преднастройки

### Проверка DNS на сервере или маршрутизаторе
```bash
ping -c 4 77.88.8.8
```

### Проверка включённой маршрутизации на роутерах
```bash
cat /proc/sys/net/ipv4/ip_forward
```

Ожидаемый результат: `1`

---

# ✔️ Задание 1 [Имя устройства + полная IP-настройка]

> Здесь сразу задаются имена устройств, интерфейсы, VLAN и GRE.  
> Это удобнее, чем разносить часть настройки в 4 и 6 задание.

---

## ISP

### Имя устройства
```bash
hostnamectl set-hostname isp.au-team.irpo
bash
```

### Настройка сети
```bash
nano /etc/network/interfaces
```

```bash
auto ens192
iface ens192 inet dhcp

auto ens224
iface ens224 inet static
    address 172.16.1.1/28

auto ens256
iface ens256 inet static
    address 172.16.2.1/28
```

```bash
systemctl restart networking
```

### Проверка
```bash
ip a
ip route
ping -c 4 172.16.1.2
ping -c 4 172.16.2.2
```

---

## HQ-RTR

### Имя устройства
```bash
hostnamectl set-hostname hq-rtr.au-team.irpo
bash
```

### Установка VLAN
```bash
apt install vlan -y
echo 8021q >> /etc/modules
modprobe 8021q
```

### Настройка сети
```bash
nano /etc/network/interfaces
```

```bash
auto ens192
iface ens192 inet static
    address 172.16.1.2/28
    gateway 172.16.1.1

auto ens224
iface ens224 inet manual

auto ens224.100
iface ens224.100 inet static
    address 192.168.100.1/27
    vlan-raw-device ens224

auto ens224.200
iface ens224.200 inet static
    address 192.168.200.1/27
    vlan-raw-device ens224

auto ens224.999
iface ens224.999 inet static
    address 192.168.99.1/29
    vlan-raw-device ens224

auto gre1
iface gre1 inet tunnel
    address 172.16.0.1
    netmask 255.255.255.252
    mode gre
    local 172.16.1.2
    endpoint 172.16.2.2
    ttl 64
```

### Для GRE
```bash
echo ip_gre >> /etc/modules
modprobe ip_gre
systemctl restart networking
```

### Проверка
```bash
ip a
ip route
ping -c 4 172.16.1.1
ping -c 4 172.16.0.2
```

---

## BR-RTR

### Имя устройства
```bash
hostnamectl set-hostname br-rtr.au-team.irpo
bash
```

### Настройка сети
```bash
nano /etc/network/interfaces
```

```bash
auto ens192
iface ens192 inet static
    address 172.16.2.2/28
    gateway 172.16.2.1

auto ens224
iface ens224 inet static
    address 192.168.0.1/28

auto gre1
iface gre1 inet tunnel
    address 172.16.0.2
    netmask 255.255.255.252
    mode gre
    local 172.16.2.2
    endpoint 172.16.1.2
    ttl 64
```

```bash
echo ip_gre >> /etc/modules
modprobe ip_gre
systemctl restart networking
```

### Проверка
```bash
ip a
ip route
ping -c 4 172.16.2.1
ping -c 4 172.16.0.1
```

---

## HQ-SRV

### Имя устройства
```bash
hostnamectl set-hostname hq-srv.au-team.irpo
bash
```

### Настройка сети
```bash
nano /etc/network/interfaces
```

```bash
auto ens192
iface ens192 inet static
    address 192.168.100.2/27
    gateway 192.168.100.1
```

```bash
systemctl restart networking
```

### Проверка
```bash
ip a
ip route
ping -c 4 192.168.100.1
```

---

## BR-SRV

### Имя устройства
```bash
hostnamectl set-hostname br-srv.au-team.irpo
bash
```

### Настройка сети
```bash
nano /etc/network/interfaces
```

```bash
auto ens192
iface ens192 inet static
    address 192.168.0.2/28
    gateway 192.168.0.1
    dns-nameservers 77.88.8.8 1.1.1.1
```

```bash
systemctl restart networking
```

### Проверка
```bash
ip a
ip route
ping -c 4 192.168.0.1
```

---

## HQ-CLI

### Имя устройства
```bash
hostnamectl set-hostname hq-cli.au-team.irpo
bash
```

На данном этапе адрес вручную не задаётся, потому что по заданию машина должна получать его по DHCP.

### Проверка
```bash
hostnamectl
```

---

# ✔️ Задание 2 [Доступ в Интернет на все устройства]

---

## 2.1 NAT на ISP

```bash
apt install iptables iptables-persistent -y

iptables -t nat -A POSTROUTING -s 172.16.1.0/28 -o ens192 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 172.16.2.0/28 -o ens192 -j MASQUERADE

netfilter-persistent save
systemctl restart netfilter-persistent
```

### Проверка
```bash
iptables -t nat -L -n -v
ping -c 4 8.8.8.8
```

---

## 2.2 NAT на HQ-RTR

```bash
apt install iptables iptables-persistent -y

iptables -t nat -A POSTROUTING -s 192.168.100.0/27 -o ens192 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.200.0/27 -o ens192 -j MASQUERADE

netfilter-persistent save
systemctl restart netfilter-persistent
```

### Проверка
```bash
iptables -t nat -L -n -v
ping -c 4 8.8.8.8
```

---

## 2.3 NAT на BR-RTR

```bash
apt install iptables iptables-persistent -y

iptables -t nat -A POSTROUTING -s 192.168.0.0/28 -o ens192 -j MASQUERADE

netfilter-persistent save
systemctl restart netfilter-persistent
```

### Проверка
```bash
iptables -t nat -L -n -v
ping -c 4 8.8.8.8
```

---

## 2.4 Проверка интернета на конечных устройствах

### HQ-SRV
```bash
ping -c 4 8.8.8.8
```

### BR-SRV
```bash
ping -c 4 8.8.8.8
```

После настройки DHCP аналогично проверить на HQ-CLI.

---

# ✔️ Задание 3 [Создание учётных записей]

## На HQ-SRV и BR-SRV создаём sshuser

```bash
useradd -m -u 2026 -s /bin/bash sshuser
passwd sshuser
usermod -aG sudo sshuser
echo 'sshuser ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

## На HQ-RTR и BR-RTR создаём net_admin

```bash
useradd -m -s /bin/bash net_admin
passwd net_admin
usermod -aG sudo net_admin
echo 'net_admin ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

## Проверка

### Проверка UID у sshuser
```bash
id sshuser
```

Ожидаемый UID: `2026`

### Проверка sudo без пароля
```bash
su - sshuser
sudo whoami
```

```bash
su - net_admin
sudo whoami
```

Ожидаемый результат: `root`

---

# ✔️ Задание 4 [Коммутация в сегменте HQ]

> Маршрутизация VLAN уже реализована в настройке `HQ-RTR` через один физический интерфейс `ens224`.

## Логика VLAN

- `VLAN 100` — HQ-SRV  
- `VLAN 200` — HQ-CLI  
- `VLAN 999` — управление  

Если используется отдельный виртуальный коммутатор HQ-SW, то:
- порт к HQ-SRV — access VLAN 100  
- порт к HQ-CLI — access VLAN 200  
- порт к HQ-RTR — trunk VLAN 100, 200, 999  

## Проверка

### На HQ-RTR проверяем наличие подинтерфейсов
```bash
ip a
```

### Проверяем пинг до HQ-SRV
```bash
ping -c 4 192.168.100.2
```

После настройки DHCP дополнительно:
```bash
ping -c 4 192.168.200.2
```

---

# ✔️ Задание 5 [SSH]

## На HQ-SRV и BR-SRV устанавливаем SSH

```bash
apt install openssh-server -y
nano /etc/ssh/sshd_config
```

Добавляем или меняем строки:

```bash
Port 2026
PermitRootLogin no
MaxAuthTries 2
AllowUsers sshuser
Banner /etc/ssh/banner
PasswordAuthentication yes
```

Создаём баннер:

```bash
echo 'Authorized access only' > /etc/ssh/banner
systemctl restart ssh
```

## Проверка

### Проверка, что SSH слушает нужный порт
```bash
ss -tulnp | grep 2026
```

### Проверка конфигурации
```bash
ssh -p 2026 sshuser@192.168.100.2
```

или

```bash
ssh -p 2026 sshuser@192.168.0.2
```

---

# ✔️ Задание 6 [GRE-туннель]

> Туннель уже добавлен в конфигурацию маршрутизаторов, здесь выполняется проверка его работоспособности.

## Проверка на HQ-RTR
```bash
ip a show gre1
ping -c 4 172.16.0.2
```

## Проверка на BR-RTR
```bash
ip a show gre1
ping -c 4 172.16.0.1
```

---

# ✔️ Задание 7 [Динамическая маршрутизация]

## Устанавливаем FRR на HQ-RTR и BR-RTR

```bash
apt install frr -y
nano /etc/frr/daemons
```

В файле включаем OSPF:

```bash
ospfd=yes
```

Запускаем сервис:

```bash
systemctl enable --now frr
systemctl restart frr
```

---

## Настройка HQ-RTR

```bash
vtysh
```

```bash
conf t
router ospf
 router-id 1.1.1.1
 passive-interface default
 network 172.16.0.0/30 area 0
 network 192.168.100.0/27 area 0
 network 192.168.200.0/27 area 0
 network 192.168.99.0/29 area 0
 area 0 authentication
exit
interface gre1
 no ip ospf passive
 ip ospf authentication
 ip ospf authentication-key P@ssw0rd
exit
write
```

---

## Настройка BR-RTR

```bash
vtysh
```

```bash
conf t
router ospf
 router-id 2.2.2.2
 passive-interface default
 network 172.16.0.0/30 area 0
 network 192.168.0.0/28 area 0
 area 0 authentication
exit
interface gre1
 no ip ospf passive
 ip ospf authentication
 ip ospf authentication-key P@ssw0rd
exit
write
```

## Проверка

### На обоих маршрутизаторах
```bash
vtysh -c "show ip ospf neighbor"
vtysh -c "show ip route ospf"
```

### Проверка связи между офисами
С BR-SRV:
```bash
ping -c 4 192.168.100.2
```

С HQ-SRV:
```bash
ping -c 4 192.168.0.2
```

---

# ✔️ Задание 8 [DHCP]

## На HQ-RTR устанавливаем DHCP-сервер

```bash
apt install isc-dhcp-server -y
nano /etc/dhcp/dhcpd.conf
```

Добавляем:

```bash
subnet 192.168.200.0 netmask 255.255.255.224 {
  range 192.168.200.2 192.168.200.30;
  option routers 192.168.200.1;
  option domain-name-servers 192.168.100.2;
  option domain-name "au-team.irpo";
  default-lease-time 600;
  max-lease-time 7200;
}
```

Указываем интерфейс:

```bash
nano /etc/default/isc-dhcp-server
```

```bash
INTERFACESv4="ens224.200"
```

Запускаем DHCP:

```bash
systemctl enable --now isc-dhcp-server
```

## Проверка

### На HQ-RTR
```bash
systemctl status isc-dhcp-server
cat /var/lib/dhcp/dhcpd.leases
```

### На HQ-CLI
```bash
ip a
ip route
cat /etc/resolv.conf
ping -c 4 192.168.200.1
ping -c 4 192.168.100.2
```

---

# ✔️ Задание 9 [DNS]

## На HQ-SRV устанавливаем bind9

```bash
apt install bind9 bind9-utils -y
```

## Файл параметров

```bash
nano /etc/bind/named.conf.options
```

```bash
options {
    directory "/var/cache/bind";

    forwarders {
        77.88.8.8;
        77.88.8.3;
    };

    dnssec-validation auto;
    listen-on { any; };
    allow-query { any; };
    recursion yes;
};
```

## Файл локальных зон

```bash
nano /etc/bind/named.conf.local
```

```bash
zone "au-team.irpo" {
    type master;
    file "/etc/bind/db.au-team.irpo";
};

zone "100.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.100";
};

zone "200.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.200";
};
```

> Для BR-сети по заданию PTR не требуется, поэтому отдельную обратную зону для `192.168.0.0/28` можно не создавать.

## Прямая зона

```bash
nano /etc/bind/db.au-team.irpo
```

```bash
$TTL 1D
@   IN SOA hq-srv.au-team.irpo. root.au-team.irpo. (
        2026010101
        12H
        1H
        1W
        1H )

@       IN NS      hq-srv.au-team.irpo.
hq-rtr  IN A       192.168.100.1
br-rtr  IN A       192.168.0.1
hq-srv  IN A       192.168.100.2
hq-cli  IN A       192.168.200.2
br-srv  IN A       192.168.0.2
docker  IN A       172.16.1.1
web     IN A       172.16.2.1
```

## Обратная зона для 192.168.100.0/27

```bash
nano /etc/bind/db.192.168.100
```

```bash
$TTL 1D
@   IN SOA hq-srv.au-team.irpo. root.au-team.irpo. (
        2026010101
        12H
        1H
        1W
        1H )

@       IN NS      hq-srv.au-team.irpo.
1       IN PTR     hq-rtr.au-team.irpo.
2       IN PTR     hq-srv.au-team.irpo.
```

## Обратная зона для 192.168.200.0/27

```bash
nano /etc/bind/db.192.168.200
```

```bash
$TTL 1D
@   IN SOA hq-srv.au-team.irpo. root.au-team.irpo. (
        2026010101
        12H
        1H
        1W
        1H )

@       IN NS      hq-srv.au-team.irpo.
2       IN PTR     hq-cli.au-team.irpo.
```

## Перезапуск

```bash
named-checkconf
named-checkzone au-team.irpo /etc/bind/db.au-team.irpo
systemctl restart bind9
systemctl enable bind9
```

## На всех машинах после настройки DNS

```bash
nano /etc/resolv.conf
```

```bash
search au-team.irpo
nameserver 192.168.100.2
```

## Проверка

### На HQ-SRV
```bash
named-checkconf
named-checkzone au-team.irpo /etc/bind/db.au-team.irpo
```

### На HQ-CLI, HQ-RTR, BR-SRV
```bash
nslookup hq-srv.au-team.irpo
nslookup hq-rtr.au-team.irpo
nslookup docker.au-team.irpo
nslookup web.au-team.irpo
nslookup 192.168.100.2
nslookup 192.168.200.2
```

---

# ✔️ Задание 10 [Часовой пояс]

На всех устройствах:

```bash
timedatectl set-timezone Asia/Tomsk
```

## Проверка

```bash
timedatectl
```

Ожидаемо должен отображаться часовой пояс `Asia/Tomsk`.

---

# ✔️ Итоговая финальная проверка модуля

## Проверка адресации
На всех устройствах:
```bash
ip a
ip route
```

## Проверка интернета
На HQ-SRV, BR-SRV, HQ-RTR, BR-RTR:
```bash
ping -c 4 8.8.8.8
```

## Проверка туннеля
На HQ-RTR:
```bash
ping -c 4 172.16.0.2
```

На BR-RTR:
```bash
ping -c 4 172.16.0.1
```

## Проверка маршрутизации между офисами
На BR-SRV:
```bash
ping -c 4 192.168.100.2
```

На HQ-SRV:
```bash
ping -c 4 192.168.0.2
```

## Проверка DHCP
На HQ-CLI:
```bash
ip a
ip route
cat /etc/resolv.conf
```

## Проверка DNS
На HQ-CLI или HQ-SRV:
```bash
nslookup hq-srv.au-team.irpo
nslookup hq-cli.au-team.irpo
nslookup docker.au-team.irpo
nslookup web.au-team.irpo
```

## Проверка SSH
```bash
ssh -p 2026 sshuser@192.168.100.2
ssh -p 2026 sshuser@192.168.0.2
```

## Проверка времени
```bash
timedatectl
```

---

# ✔️ Краткий список того, что важно не перепутать

- `sshuser` создаётся с **UID 2026**
- пароль `sshuser` — **P@ssw0rd**
- пароль `net_admin` — **P@ssw0rd**
- SSH работает на **порту 2026**
- DNS-записи `docker.au-team.irpo` и `web.au-team.irpo` — это **A-записи**
- туннель между офисами должен быть настроен **между HQ-RTR и BR-RTR**
- HQ-CLI получает адрес **по DHCP**
