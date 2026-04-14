<img width="561" height="671" alt="изображение" src="https://github.com/user-attachments/assets/c2ae75fa-bb8c-4d8b-97fc-5d024a828db3" />
<br/>

# <div align="center"><strong>⚙️</strong></div> <div align="center"><strong>МОДУЛЬ 1</strong></div>

<br/>

<br/>

> [!WARNING]
> ## ПРЕДНАСТРОЙКА
>
> ```
> systemctl stop NetworkManager
> systemctl disable NetworkManager
> systemctl mask NetworkManager
> ```
></br>
>
> На всех устройствах, кроме **ISP**:
>```
>nano /etc/resolv.conf
>nameserver 77.88.8.8
>nameserver 1.1.1.1
>```
></br>
>
> На **РОУТЕРАХ** sysctl -p:
>```
> echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
>```
></br> 
>
> **Сурс** листы везде:
> ```
>nano /etc/apt/sources.list
>```
>```
> # deb cdrom:.......
>  ↑
>  Ставим комментарий
>```
>```
>apt update
>```
</br>

<table>
  <thead>
    <tr>
      <th>Маска подсети</th>
      <th>Полная маска</th>
      <th>Сколько адресов вмещает</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/26</td>
      <td>255.255.255.192</td>
      <td align="center">62</td>
    </tr>
    <tr>
      <td>/27</td>
      <td>255.255.255.224</td>
      <td align="center">30</td>
    </tr>
    <tr>
      <td>/28</td>
      <td>255.255.255.240</td>
      <td align="center">14</td>
    </tr>
    <tr>
      <td>/29</td>
      <td>255.255.255.248</td>
      <td align="center">6</td>
    </tr>
  </tbody>
</table>

</br>

## ✔️ Задание 1 <code>[ Адрессация + Имя устройства]</code>

> [!WARNING]
> В инструкции по сетевой настройке используется редактирование: <strong>`etc/network/interfaces`</strong>

> [!NOTE]
> **Используй редактор файлов: `nano`**

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Произведите `базовую` настройку устройств

- Настройте имена устройств согласно топологии. Используйте полное доменное имя
- На всех устройствах необходимо сконфигурировать IPv4
- IP-адрес должен быть из приватного диапазона, в случае, если сеть локальная, согласно RFC1918
- Локальная сеть в сторону HQ-SRV(VLAN100) должна вмещать не более 32 адресов
- Локальная сеть в сторону HQ-CLI(VLAN200) должна вмещать не более 16 адресов
- Локальная сеть для управления(VLAN999) должна вмещать не более 8 адресов
- Локальная сеть в сторону BR-SRV должна вмещать не более 16 адресов-
 Сведения об адресах занесите в отчёт, в качестве примера используйте Таблицу 2

</details>

<br/>

<details>
<summary><strong>Настройка имени</strong></summary>
  
<br/>

  - Для **Linux** используется команда: <strong>`hostnamectl set-hostname (имя устройства.au-team.irpo)`</strong>
  
  - Обновить имя можно введя команду: **`newgrp`**
### ISP
```
hostnamectl set-hostname isp.au-team.irpo
```
```
newgrp
```
<br/>

### HQ-RTR
```
hostnamectl set-hostname hq-rtr.au-team.irpo
```
```
newgrp
```
<br/>

### BR-RTR
```
hostnamectl set-hostname br-rtr.au-team.irpo
```
```
newgrp
```
<br/>

### HQ-SRV
```
hostnamectl set-hostname hq-srv.au-team.irpo
```
```
newgrp
```
<br/>

### HQ-CLI
```
hostnamectl set-hostname hq-cli.au-team.irpo
```
```
newgrp
```
<br/>

### BR-SRV
```
hostnamectl set-hostname br-srv.au-team.irpo
```
```
newgrp
```
<br/>

</details>

<details>
<summary><strong>Настройка адрессации в файле <code>etc/network/interfaces</code></strong></summary>
<br/>

### Настройка адресации (кроме HQ-CLI(он позже))
##
### ISP:
```
auto ens224
iface ens224 inet static
address 172.16.1.1/28
auto ens256
iface ens256 inet static
address 172.16.2.1/28
```

### HQ-RTR: (в 4 и 6 задании продолжение)
```
allow-hotplug ens192
iface ens192 inet static
address 172.16.1.2/28
gateway 172.16.1.1

auto gre1
iface gre1 inet tunnel
address 172.16.0.1
netmask 255.255.255.240
mode gre
local 172.16.1.2
endpoint 172.16.2.2
ttl 64
```

### BR-RTR: (в 6 задании продолжение)
```
allow-hotplug ens192
iface ens192 inet static
address 172.16.2.2/28
gateway 172.16.2.1

auto ens224
iface ens224 inet static
address 192.168.0.1/28

auto gre1
iface gre1 inet tunnel
address 172.16.0.2
netmask 255.255.255.240
mode gre
local 172.16.2.2
endpoint 172.16.1.2
ttl 64
```

### BR-SRV:

```
allow-hotplug ens192
iface ens192 inet static
address 192.168.0.2/28
gateway 192.168.0.1
dns-nameservers 192.168.100.2 192.168.0.2
dns-search au-team.irpo
```

### HQ-SRV:

```
allow-hotplug ens192
iface ens192 inet static
address 192.168.100.2/27
gateway 192.168.100.1
```

</details>

<details>
<summary><strong>Таблицы сетей</strong></summary>

</br>

<table align="center">
  <tr>
    <td align="center"><strong>Сеть</strong></td>
    <td align="center"><strong>Адрес подсети</strong></td>
    <td align="center"><strong>Пул-адресов</strong></td>
  </tr>
  <tr>
    <td align="center">SRV-Net (VLAN 100)</td>
    <td align="center">192.168.100.0/27</td>
    <td align="center">192.168.100.1-62</td>
  </tr>
  <tr>
    <td align="center">CLI-Net (VLAN 200)</td>
    <td align="center">192.168.200.0/28</td>
    <td align="center">192.168.200.1-14</td>
  </tr>
  <tr>
    <td align="center">MGMT (VLAN 999)</td>
    <td align="center">192.168.99.0/29</td>
    <td align="center">192.168.99.1-6</td>
  </tr>
  <tr>
    <td align="center">BR-Net</td>
    <td align="center">192.168.0.0/27</td>
    <td align="center">192.168.0.1-30</td>
  </tr>
  <tr>
    <td align="center">ISP-HQ</td>
    <td align="center">172.16.1.0/28</td>
    <td align="center">172.16.1.1 - 14</td>
  </tr>
  <tr>
    <td align="center">ISP-BR</td>
    <td align="center">172.16.2.0/28</td>
    <td align="center">172.16.2.1 - 14</td>
  </tr>
</table>
<p align="center"><strong>Таблица подсетей</strong></p>

#

<table align="center">
  <tr>
    <td align="center"><strong>Устройство</strong></td>
    <td align="center"><strong>Интерфейс</strong></td>
    <td align="center"><strong>IPv4/IPv6</strong></td>
    <td align="center"><strong>Маска/Префикс</strong></td>
    <td align="center"><strong>Шлюз</strong></td>
    <td align="center"><strong>Сеть</strong></td>
  </tr>
  <tr>
    <td align="center" rowspan="3">ISP</td>
    <td align="center">ens192</td>
    <td align="center">(DHCP)</td>
    <td align="center">(DHCP)</td>
    <td align="center">(DHCP)</td>
    <td align="center">INTERNET</td>
  </tr>
  <tr>
    <td align="center">ens224</td>
    <td align="center">172.16.1.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
    <td align="center">ISP-HQ-RTR</td>
  </tr>
  <tr>
    <td align="center">ens256</td>
    <td align="center">172.16.2.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
    <td align="center">ISP-BR-RTR</td>
  </tr>
  <tr>
    <td align="center" rowspan="3">HQ-RTR</td>
    <td align="center">ens192</td>
    <td align="center">172.16.1.2</td>
    <td align="center">/28</td>
    <td align="center">172.16.1.1</td>
    <td align="center">ISP-HQ-RTR</td>
  </tr>
  <tr>
    <td align="center">ens224.200</td>
    <td align="center">192.168.200.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
    <td align="center">HQ-RTR-CLI</td>
  </tr>
  <tr>
    <td align="center">ens224.100</td>
    <td align="center">192.168.100.1</td>
    <td align="center">/26</td>
    <td align="center"></td>
    <td align="center">HQ-RTR-SRV</td>
  </tr>
  <tr>
    <td align="center" rowspan="2">BR-RTR</td>
    <td align="center">ens192</td>
    <td align="center">172.16.5.2</td>
    <td align="center">/28</td>
    <td align="center">172.16.5.1</td>
    <td align="center">ISP-BR-RTR</td>
  </tr>
  <tr>
    <td align="center">ens224</td>
    <td align="center">192.168.0.1</td>
    <td align="center">/28</td>
    <td align="center"></td>
    <td align="center">BR-RTR-SRV</td>
  </tr>
  <tr>
    <td align="center">HQ-SRV</td>
    <td align="center">ens192</td>
    <td align="center">192.168.100.62</td>
    <td align="center">/26</td>
    <td align="center">192.168.100.1</td>
    <td align="center">HQ-RTR-SRV</td>
  </tr>
  <tr>
    <td align="center">BR-SRV</td>
    <td align="center">ens192</td>
    <td align="center">192.168.0.2</td>
    <td align="center">/27</td>
    <td align="center">192.168.0.1</td>
    <td align="center">BR-RTR-SRV</td>
  </tr>
  <tr>
    <td align="center">HQ-CLI</td>
    <td align="center">ens192</td>
    <td align="center">192.168.200.##(DHCP)</td>
    <td align="center">/28</td>
    <td align="center">192.168.200.1</td>
    <td align="center">HQ-RTR-CLI</td>
  </tr>
</table>
<p align="center"><strong>Таблица адресации</strong></p>

</details>
</br>

## ✔️ Задание 2 <code>[NAT на ISP]</code>

<br/>

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройка `ISP`

- Настройте адресацию на интерфейсах:

  - Интерфейс, подключенный к магистральному провайдеру, получает адрес по DHCP **[Выполнено в задании 1]**

  - Настройте маршруты по умолчанию там, где это необходимо 

  - Интерфейс, к которому подключен HQ-RTR, подключен к сети 172.16.1.0/28 **[Выполнено в задании 1]**

  - Интерфейс, к которому подключен BR-RTR, подключен к сети 172.16.2.0/28 **[Выполнено в задании 1]**

  - На ISP настройте динамическую сетевую трансляцию в сторону HQ-RTR и BR-RTR для доступа к сети Интернет

</details>

<br/>

<details>
<summary><strong>Настройка динамической трансляции через iptables <code>NAT</code></strong></summary>

### Настройка динамической сетевой трансляции на _`ISP`_

```
echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
```
```
apt-get install iptables iptables-persistent –y
```
```
iptables –t nat –A POSTROUTING –s 172.16.1.0/28 –o ens192 –j MASQUERADE  
iptables –t nat –A POSTROUTING –s 172.16.2.0/28 –o ens192 –j MASQUERADE
netfilter-persistent save
systemctl restart netfilter-persistent  
```
</br>

<details>
<summary><strong><code>Либо другая настройка</code></strong></summary>  

```  
apt install iptables  
apt install iptables iptables-persistent  
iptables –t nat –A POSTROUTING –s 172.16.1.0/28 –o ens192 –j MASQUERADE   
iptables –t nat –A POSTROUTING –s 172.16.2.0/28 –o ens192 –j MASQUERADE   
iptables-save > /etc/iptables/rules.v4  
```
</br>

<summary><strong><code>Либо настройка через nftables </code></strong></summary> 

  ```
nano /etc/nftables.conf
   ```
   ```
table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100; policy accept;
        oifname "ens192" masquerade
    }
}
 ```

###Если не работает, презагрузите машину, на которую пытаетесь раздать интернет, либо, возвращайтесь к другому варианту.

> **`ens192`** - интерфейс с которого приходит **интернет**
> 
> Для проверки можно использовать команду: **`iptables –L –t nat`** - должны высветится в Chain POSTROUTING две настроенные подсети  

> Для того, чтобы сбросить настройку *nat*, можно использовать команду **`iptables -t nat -F`**

#
</details>

</details>

<br/>

## ✔️ Задание 3 `[Создание учетных записей]` 

> [!WARNING]
> Обратите внимание, что группа <code><strong>WHEEL</strong></code> не используется!
<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Создание локальных учетных записей

- Создайте пользователя sshuser на серверах HQ-SRV и BR-SRV

  - Пароль пользователя sshuser с паролем P@ssw0rd

  - Идентификатор пользователя 2026

  - Пользователь sshuser должен иметь возможность запускать sudo без ввода пароля.

- Создайте пользователя net_admin на маршрутизаторах HQ-RTR и BR-RTR

  - Пароль пользователя net_admin с паролем P@ssw0rd

  - При настройке на OC на базе Linux пользователь net_admin должен обладать максимальными привилегиями

  - При настройке ОС на базе Linux, запускать sudo без дополнительной аутентификации
 
</details>

<br/>

<details>
<summary><strong>Как создать <code>sshuser</code></strong></summary>
<br/>

- ## Создание учёток _`sshuser`_ производится на _`HQ-SRV`_ и _`BR-SRV`_

**1.** Создаём sshuser следующими командами:
```
useradd sshuser -u 2026
```
```
passwd sshuser
```
```
P@ssw0rd
```
<br/>

**2.** После чего даем пользователю <strong><code>root</strong></code> права:
```yml
usermod -aG sudo sshuser
```

<br/>

**3.** Добавляем следующую строку в файле `/sudoers`:
```
nano /etc/sudoers
```

```yml
sshuser ALL=(ALL) NOPASSWD:ALL
```
> Позволяет запускать **`sudo`** без аутентификации

<br/>

**4.** Создаем и задаем необходимые права на домашнюю папку
```
mkdir /home/sshuser
```
```
chown sshuser:sshuser /home/sshuser
```
```
chmod 700 /home/sshuser
```
<br/>
</details>

<details>
<summary><strong>Как создать <code>net_admin</code></strong></summary>
<br/>

- ## Пользователь _`net_admin`_ на _`HQ-RTR`_ и _`BR-RTR`_

**1.** Создаём **`net_admin`**, следующими командами, но уже без `-u 1010` и с новым паролем:
```
useradd net_admin
```
```
passwd net_admin
```
```
P@ssw0rd
```
<br/>

**2.** После чего даем пользователю <strong><code>root</strong></code> права:
```yml
usermod -aG sudo net_admin
```
<br/>

**3.** Добавляем следующую строку в `/sudoers`:
```
nano /etc/sudoers
```
```yml
net_admin ALL=(ALL) NOPASSWD:ALL
```
> Позволяет запускать **`sudo`** без аутентификации
<br/>

**4.** Создаем и задаем необходимые права на домашнюю папку
```
mkdir /home/net_admin
```
```
chown net_admin:net_admin /home/net_admin
```
```
chmod 700 /home/net_admin
```
<br/>

</details>

<br/>

## ✔️ Задание 4 `[VLAN]`

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройте на интерфейсе `HQ-RTR` в сторону офиса `HQ` виртуальный коммутатор

- Сервер HQ-SRV должен находиться в ID VLAN 100
- Клиент HQ-CLI в ID VLAN 200
- Создайте подсеть управления с ID VLAN 999
- Основные сведения о настройке коммутатора и выбора реализации разделения на VLAN занесите в отчёт

</details>
<br/>

<details>
<summary><strong>Настройка VLAN на <code>HQ-RTR</code></strong></summary>

## Настройка VLAN на **`HQ-RTR`**

- Для начала установи пакет _**`vlan`**_:

```
apt install vlan
```

- После чего добавляем модуль _**`modprobe 8021q`**_ командой:
```
echo 8021q >> /etc/modules
```
- Далее переходим к конфигурации файла _**`/etc/network/interfaces`**_ и приводим её к виду:
```
nano /etc/network/interfaces
```

```
# The primary network interface
auto ens192  
iface ens192 inet static  
address 172.16.1.2/28
gateway 172.16.1.1

auto gre1
iface gre1 inet tunnel
address 172.16.0.1
netmask 255.255.255.240
mode gre
local 172.16.1.2
endpoint 172.16.2.2
ttl 64
  
auto ens224  
iface ens224 inet static  
address 192.168.100.1/27 
  
auto ens224:1  
iface ens224:1 inet static  
address 192.168.200.1/28

auto ens224:2  
iface ens224:2 inet static  
address 192.168.99.9/29

auto ens224.100  
iface ens224.100 inet manual   
Vlan-raw-device ens224  
  
auto ens224.200  
iface ens224.200 inet manual   
Vlan-raw-device ens224:1

auto ens224.999  
iface ens224.999 inet manual   
Vlan-raw-device ens224:2
```

</details>

<br/>

## ✔️ Задание 5 `[SSH]`

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройка безопасного удаленного доступа на серверах `HQ-SRV` и `BR-SRV`

- Для подключения используйте порт 2026
- Разрешите подключения только пользователю sshuser
- Ограничьте количество попыток входа до двух
- Настройте баннер «Authorized access only»

</details>
<br/>

<details>
<summary><strong>Настройка <code>SSH</code></strong></summary>
<br/>

### SSH настраиваем на `HQ-SRV` и `BR-SRV`

**1.** Для настройки **SSH** необходимо его установить коммандой:
```
apt-get install openssh-server -y
```

</br>

**2.** После чего необходимо добавить строчки в файл **`sshd_config`**:
```
nano /etc/ssh/sshd_config
```

```
Port 2026
MaxAuthTries 2
PasswordAuthentication yes
Banner /etc/ssh/banner
AllowUsers  sshuser
           ^ - это TAB
```
<br/>

**3.** После чего требуется создать файл **`/etc/ssh/banner`** и привести его в следующую форму:
```
----------------------
Authorized access only
----------------------
```
</br>

**4.** Далее необходимо перезапустить **`SSH`** коммандой:
  
```
systemctl restart sshd
```


</details>

<br/>

## ✔️ Задание 6 `[GRE-TUNNEL]`

> [!WARNING]
> **Не используйте** устройства с именем **`tun0`, `gre0` и `sit0`**, т.к они являются зарезервированными в iproute2 («base devices») и имеют особое поведение.

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Между офисами `HQ` и `BR` необходимо сконфигурировать _`IP-туннель`_

- Сведения о туннеле занесите в отчёт  
- На выбор технологии GRE или IP in IP
</details>
<br/>

<details>
<summary><strong>Настройка GRE в файле <code>etc/network/interfaces</code>на <code>HQ-RTR</code></strong></summary>
<br/>

Для настройки _**`GRE`**_ туннеля на **`HQ-RTR`** переходим в файл конфигурации

```
nano /etc/network/interfaces
```

И убеждаемся в наличии этих строк:
```
auto gre1
iface gre1 inet tunnel
address 172.16.0.1
netmask 255.255.255.240
mode gre
local 172.16.1.2
endpoint 172.16.2.2
ttl 64
```

Для работы туннеля необходимо добавить строчку в файл `/etc/modules`
```
echo gre_ip >> /etc/modules
```
```
systemctl restart networking
```
</details>

<details>
<summary><strong>Настройка GRE в файле <code>etc/network/interfaces</code> на <code>BR-RTR</code></strong></summary>
<br/>
Для настройки *GRE* туннеля на *BR-RTR* переходим в файл конфигурации
  
```
nano /etc/network/interfaces
```

И убеждаемся в наличии этих строк:
```
auto gre1
iface gre1 inet tunnel
address 172.16.0.2
netmask 255.255.255.240
mode gre
local 172.16.2.2
endpoint 172.16.1.2
ttl 64
```

Для работы туннеля необходимо добавить строчку в файл `/etc/modules`
```
echo gre_ip >> /etc/modules
```
```
systemctl restart networking
```
</details>
<br/>


## ✔️ Задание 7 `[OSPF]`

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Обеспечьте динамическую маршрутизацию: ресурсы одного офиса должны быть доступны из другого офиса. Для обеспечения динамической маршрутизации используйте `link state` протокол на ваше усмотрение

- Разрешите выбранный протокол только на интерфейсах в ip туннеле  
- Маршрутизаторы должны делиться маршрутами только друг с другом  
- Обеспечьте защиту выбранного протокола посредством парольной защиты  
- Сведения о настройке и защите протокола занесите в отчёт

</details>
<br/>

<details>
<summary><strong>Настройка <code>OSPF</code></strong></summary>
<br/>


## Настройка `OSPF` производится на `HQ-RTR` и `BR-RTR`:

### HQ-RTR

**1.** Устанавливаем пакет `FRR`

```
sudo apt install -y frr
```

**2.** В конфигурационном файле `/etc/frr/daemons` необходимо активировать выбранный протокол `OSPF` для дальнейшей реализации его настройки:

```
nano /etc/frr/daemons

!!! Ищем следующую строку и меняем с (no) на (yes)
ospfd = yes

```

**3.** Далее перезаргужаем и добавляем в автозагрузку службу **`FRR`**

```
systemctl restart frr
systemctl enable --now frr
```

**4.** Переходим в интерфейс управления симуляцией **`FRR`** командой:
```
vtysh
```

**5.** Пишем команды для настройки **маршрутизации:**
 
```
conf t
router ospf
  passive-interface default
  router-id 1.1.1.1
  network 172.16.0.0/28 area 0
  network 192.168.100.0/27 area 1
  network 192.168.200.0/28 area 2
  area 0 authentication
exit

int gre1
  no ip ospf network broadcast
  no ip ospf passive
  ip ospf authentication
  ip ospf authentication-key password
(config-if)exit
(config)exit
#write
```
<br>

### BR-RTR

**1-4.** Пункты такие же как и в HQ-RTR

**5.** Пишем команды для настройки **маршрутизации:**

**Меняется:** 

- `id-router: 2.2.2.2`

- `network 192.168.0.0/27 area 3`

- `network 172.16.0.0/28 area 0`

```
conf t
router ospf
  passive-interface default
  router-id 2.2.2.2
  network 192.168.0.0/27 area 3
  network 172.16.0.0/28 area 0
  area 0 authentication
exit

int gre1
  no ip ospf network broadcast
  no ip ospf passive
  ip ospf authentication
  ip ospf authentication-key password
(config-if)exit
(config)exit
#write
```

### ПРОВЕРКА

Пингуем: **`BR-SRV - > HQ-SRV`** и **`BR-SRV - > HQ-CLI`**

Проверка в **FRR**:

```
vtysh
  show ip ospf neighbor
  show ip route ospf
```

</details>

<br/>


<br/>
Настройка маршрутов, если не работает <code><strong>OSPF</strong></code>:

<br/>
<details>
<summary><strong>Настройка <code>маршрутов</code></strong></summary>
<br/>

После создания влана настраиваем маршрут для подсетей (чтобы они видели друг друга)  
На роутере *HQ-RTR* настройка выглядит так:
```
sudo nano /etc/systemd/system/iproute.service
```

<br/>

После чего добавляем текст:
```
[Unit]
Description=iproute
After=network.target

[Service]
Type=oneshot
ExecStart=/sbin/ip route add 192.168.0.0/27 via 172.16.0.2 dev gre1
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

<br/>

На роутере *BR-RTR* создаем тот же файли настраивеим:
```
sudo nano /etc/systemd/system/iproute.service
Для HQ-RTR:
[Unit]
Description=iproute
After=network.target

[Service]
Type=oneshot
ExecStart=/sbin/ip route add 192.168.100.0/27 via 172.16.0.1 dev gre1
ExecStart=/sbin/ip route add 192.168.200.0/28 via 172.16.0.1 dev gre1
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

<br/>

После на обоих устройствах прописываем:
```
systemctl daemon-reload
systemctl enable iproute.service
systemctl start iproute.service
```

</details>

<br/>

## ✔️ Задание 8 `[NAT на HQ-rtr и BR-rtr]`

> [!NOTE]
> Пул адрессов можно посмотреть в таблице <strong>[Задания 1](https://github.com/Flicks1383/Demo2025_debian/tree/main/Module1#задание-1)</strong> или же отталкиваться от вашей адрессации.

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройка динамической трансляции адресов

- Настройте динамическую трансляцию адресов для обоих офисов.  
- Все устройства в офисах должны иметь доступ к сети Интернет

</details>
<br/>

<details>
<summary><strong>Настройка NAT на <code>BR</code> и <code>HQ</code></strong></summary>
<br/>

## > Настройка динамической трансляции адресов <

> ### Настройка на `ISP` выполнена в [Задании 2](https://github.com/Flicks1383/Demo09.02.06_2025/tree/main/module1#задание-2)

### Настройка динамической сетевой трансляции на `HQ-RTR`
```
apt-get install iptables iptables-persistent –y
iptables –t nat –A POSTROUTING –s 192.168.100.0/27 –o ens192 –j MASQUERADE
iptables –t nat –A POSTROUTING –s 192.168.200.0/28 –o ens192 –j MASQUERADE
netfilter-persistent save
systemctl restart netfilter-persistent  
```
### Настройка динамической сетевой трансляции на `BR-RTR`

```
apt-get install iptables iptables-persistent –y
iptables –t nat –A POSTROUTING –s 192.168.0.0/27 –o ens192 –j MASQUERADE
netfilter-persistent save
systemctl restart netfilter-persistent  
```
> Для того, чтобы **сбросить** настройку *nat*, можно использовать команду **`iptables -t nat -F`**



</br>

</details>

<details>
<summary><strong>Альтарнативная настройка сетевой трансляции через <code> nftables </code> </strong></summary>

### ВНИМАНИЕ! Если у вас не вышло настроить <code> NAT </code> через <code> nftables </code>, возвращайтесь к первому варианту!

```
table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100; policy accept;
        oifname "ens192" masquerade
    }
}
```
</details>

</br>

## ✔️ Задание 9 `[DHCP]`

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройка протокола динамической конфигурации хостов

- Настройте нужную подсеть  
- Для офиса HQ в качестве сервера DHCP выступает маршрутизатор HQ-RTR.  
- Клиентом является машина HQ-CLI.  
- Исключите из выдачи адрес маршрутизатора  
- Адрес шлюза по умолчанию – адрес маршрутизатора HQ-RTR.  
- Адрес DNS-сервера для машины HQ-CLI – адрес сервера HQ-SRV.  
- DNS-суффикс для офисов HQ – au-team.irpo  
- Сведения о настройке протокола занесите в отчёт

</details>
<br/>

<details>
<summary><strong>Настройка DHCP на <code>HQ-RTR</code></strong></summary>
<br/>

## > Настройка _`DHCP`_ на _`HQ-RTR`_ для `CLI` <

<br/>

**1.** Устанавливаем сам **DHCP-сервер**:  
```
apt install isc-dhcp-server
```
<br/>

**2.** После чего переходим в конфигурацию файла `dhcpd.conf` и добавляем следующие строчки:
```
nano /etc/dhcp/dhcpd.conf
```

```
subnet 192.168.200.0 netmask 255.255.255.240 {
  range 192.168.200.2 192.168.200.14;
  option domain-name-servers 192.168.100.62;
  option domain-name "au-team.irpo";
  option routers 192.168.200.1;
  default-lease-time 600;
  max-lease-time 7200;
}
```
<br/>

**3.** После чего переходим в конфигурацию файла `isc-dhcp-server` и меняем ее добалвяя данный текст:
```
nano /etc/default/isc-dhcp-server

INTERFACESv4="ens224:1" - порт смотрящий в сторону CLI
```

<br/>

**4.** Включаем сервиc **`DHCP`** и добавляем в автозагрузку на **`HQ-RTR`**:
```
systemctl start isc-dhcp-server
systemctl enable isc-dhcp-server
```

**5.** Далее на клиенсткой машине необходимо в настройках сет.адаптера выбрать режим **DHCP** и проверить работоспособность
<br/>

</details>

</br>

## ✔️ Задание 10 `[DNS]`

<details>
<summary><strong>[ОПИСАНИЕ ЗАДАНИЙ]</strong></summary>

### Настройка DNS для офисов HQ и BR  
- Основной DNS-сервер реализован на HQ-SRV.  
- Сервер должен обеспечивать разрешение имён в сетевые адреса устройств и обратно в соответствии с таблицей 3  
- В качестве DNS сервера пересылки используйте любой общедоступный DNS сервер  

</details>
<br/>

<table align="center">
  <tr>
    <td align="center">Устройство</td>
    <td align="center">Запись</td>
    <td align="center">Тип</td>
  </tr>
  <tr>
    <td align="center">HQ-RTR</td>
    <td align="center">hq-rtr.au-team.irpo</td>
    <td align="center">A,PTR</td>
  </tr>
  <tr>
    <td align="center">BR-RTR</td>
    <td align="center">br-rtr.au-team.irpo</td>
    <td align="center">A</td>
  </tr>
  <tr>
    <td align="center">HQ-SRV</td>
    <td align="center">hq-srv.au-team.irpo</td>
    <td align="center">A,PTR</td>
  </tr>
  <tr>
    <td align="center">HQ-CLI</td>
    <td align="center">hq-cli.au-team.irpo</td>
    <td align="center">A,PTR</td>
  </tr>
  <tr>
    <td align="center">BR-SRV</td>
    <td align="center">br-srv.au-team.irpo</td>
    <td align="center">A</td>
  </tr>
  <tr>
    <td align="center">HQ-RTR</td>
    <td align="center">moodle.au-team.irpo</td>
    <td align="center">CNAME</td>
  </tr>
  <tr>
    <td align="center">BR-RTR</td>
    <td align="center">wiki.au-team.irpo</td>
    <td align="center">CNAME</td>
  </tr>
</table>

<p align="center"><strong>Таблица 2</strong></p>

<br/>

<details>
<summary><strong>Настройка при помощи <code>bind9</code></strong>[Дольше]</summary>

<br/>

## > Настройка BIND9 на `HQ-SRV`<

### HQ-SRV

<br/>

**1.** Для работы с **DNS** требуется установить **`bind`** и доп. пакет командой:

```
apt-get install bind9 bind9-utils
```
<br/>

**2.** Далее необходимо сконфигурировать файл **`named.conf.options`** таким образом:

```
nano /etc/bind/named.conf.options
```

```
  listen-on { 127.0.0.1; 192.168.100.0/27; 192.168.200.0/28; 192.168.0.0/27; 172.16.0.0/30; };
  forwarders { 127.0.0.1; 8.8.8.8; 192.168.100.62; 8.8.4.4; };
  recursion yes;
  allow-query { 127.0.0.1; 192.168.100.0/27; 192.168.200.0/28; 192.168.0.0/27; 172.16.0.0/30; };
  allow-query-cache { 127.0.0.1; 192.168.100.0/27; 192.168.200.0/28; 192.168.0.0/27; 172.16.0.0/28; };
  allow-recursion { 127.0.0.1; 192.168.100.0/27; 192.168.200.0/28; 192.168.0.0/27; 172.16.0.0/28; };
  dnssec-validation auto;
```
<br/>

**3.** Конфигурация ключей **`rndc`**:
```
rndc-confgen > /etc/rndc.key 
```
</br>

**❗ Для проверки, можно использовать комманду:** 

```
named-checkconf
```
</br>

**4.** Далее необходимо запустить **утилиту** коммандой:

```
systemctl enable --now named
```
</br>

**5.** Далее требуется изменить конфигурацию файла на **`HQ-SRV`** **`resolv.conf`**:

```
nano /etc/resolv.conf
```

```
nameserver 192.168.100.62
search au-team.irpo
```
</br>

**6.** После чего требуется прописать в **`/etc/bind/named.conf.local`**:

```
zone "au-team.irpo" {
  type master;
  file "/etc/bind/au-team.irpo";
};

zone "100.168.192.in-addr.arpa" {
  type master;
  file "/etc/bind/100.168.192.in-addr.arpa";
};

zone "200.168.192.in-addr.arpa" {
  type master;
  file "/etc/bind/200.168.192.in-addr.arpa";
};

zone "0.168.192.in-addr.arpa" {
  type master;
  file "/etc/bind/0.168.192.in-addr.arpa";
};
```
</br>

**7.** Далее следующими командами **создаём копию** файла и присваиваем права:

```
cp /etc/bind/db.local /etc/bind/au-team.irpo
```

</br>

**8.** После чего приводим **файл `au-team.irpo`** к следующему виду:
```
nano /etc/bind/au-team.irpo
```

```
$TTL    1D
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                                2024102200      ; serial
                                12H             ; refresh
                                1H              ; retry
                                1W              ; expire
                                1H              ; ncache
                        )
@       IN      NS      hq-srv.au-team.irpo.
hq-rtr  IN      A       192.168.100.1
br-rtr  IN      A       192.168.0.1
hq-srv  IN      A       192.168.100.62
hq-cli  IN      A       192.168.200.3
br-srv  IN      A       192.168.0.2
moodle  IN      CNAME   hq-rtr
wiki    IN      CNAME   br-rtr
_ldap._tcp.au-team.irpo. IN SRV 0 5 389 br-srv.au-team.irpo.
_kerberos._tcp.au-team.irpo.        IN      SRV     0 100 88  br-srv.au-team.irpo.
_kdc._tcp.au-team.irpo.             IN      SRV     0 100 88  br-srv.au-team.irpo.
_kpasswd._tcp.au-team.irpo.         IN      SRV     0 100 464 br-srv.au-team.irpo.
```
</br>

</br>

**9.** После чего **создаем файлы** командами:
```
cp /etc/bind/db.127 /etc/bind/100.168.192.in-addr.arpa
cp /etc/bind/db.127 /etc/bind/200.168.192.in-addr.arpa
cp /etc/bind/db.127 /etc/bind/0.168.192.in-addr.arpa
```
</br>

**10.** После изменений файл **`100.168.192.in-addr.arpa`** выглядит так:
```
nano /etc/bind/100.168.192.in-addr.arpa
```

```
$TTL    1D
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                                2024102200      ; Serial
                                12H             ; Refresh
                                1H              ; Retry
                                1W              ; Expire
                                1H              ; Ncache
                        )
@       IN      NS      localhost.
1       IN      PTR     hq-rtr.au-team.irpo.
62      IN      PTR     hq-srv.au-team.irpo.
```

</br>

**11.** После изменений файл **`200.168.192.in-addr.arpa`** выглядит так:
```
nano /etc/bind/200.168.192.in-addr.arpa
```

```
$TTL    1D
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                                2024102200      ; Serial
                                12H             ; Refresh
                                1H              ; Retry
                                1W              ; Expire
                                1H              ; Ncache
                        )
@       IN      NS      localhost.
2       IN      PTR     hq-cli.au-team.irpo.
```

</br>

**12.** После изменений файл **`0.168.192.in-addr.arpa`** выглядит так:
```
nano /etc/bind/0.168.192.in-addr.arpa
```

```
$TTL    1D
@       IN      SOA     hq-srv.au-team.irpo. root.au-team.irpo. (
                                2024102200      ; Serial
                                12H             ; Refresh
                                1H              ; Retry
                                1W              ; Expire
                                1H              ; Ncache
                        )
@       IN      NS      localhost.
1       IN      PTR     br-rtr.au-team.irpo.
2       IN      PTR     br-srv.au-team.irpo.
```

### ❗ ❗ ❗ Все пробелы выше ^ ставятся TAB'ом

</br>

**13.** После чего можно проверить **ошибки** командой:
```
named-checkconf -z
```
</br>

**14.** А также перезапускаем **`bind`** командой:

```
systemctl restart named bind9
```
</br>

**15.** На всех устройствах локальной сети необходимо указать в конфигурационном файле `resolv.conf`:
```
nano /etc/resolv.conf
```

```
nameserver 192.168.100.62
search au-team.irpo
```
</br>

**16.** Проверить работоспособность можно **командой**:
```
nslookup **IP-адрес/DNS-имя**
```
</br>

</details>

<br/>

<details>
<summary><strong>Настройка при помощи <code>dnsmasq</code></strong>[Быстрее]</summary>
<br/>

Для начала устанавливаются необходимые пакеты:
```
apt install iptables iptables-persistent dnsmasq -y
```

После чего необходимо открыть порт 53
```
iptables -I INPUT -p udp --dport 53 -j ACCEPT
netfilter-persistent save
```
Дальше конфигурируем файл `/etc/dnsmasq.conf`
```
no-resolv
interface=ens192
read-ethers
listen-address=192.168.100.62
server=8.8.8.8
server=8.8.4.4
address=/hq-rtr.au-team.irpo/192.168.100.1
address=/hq-srv.au-team.irpo/192.168.100.62
address=/br-rtr.au-team.irpo/192.168.0.1
address=/br-srv.au-team.irpo/192.168.0.2
address=/hq-cli.au-team.irpo/192.168.200.3
srv-host=_ldap._tcp.au-team.irpo,br-srv.au-team.irpo,389
srv-host=_kerberos._tcp.au-team.irpo,br-srv.au-team.irpo,88
srv-host=_kdc._tcp.au-team.irpo,br-srv.au-team.irpo,88
srv-host=_kpasswd._tcp.au-team.irpo,br-srv.au-team.irpo,464
cname=moodle.au-team.irpo,hq-rtr.au-team.irpo
cname=wiki.au-team.irpo,br-rtr.au-team.irpo
```
Далее в файле hosts на HQ-SRV правим:
```
nano /etc/hosts
```
```
127.0.0.1  localhost
127.0.1.1  server.localdomain  server
192.168.100.1  hq-rtr.au-team.irpo  hq-rtr
192.168.100.62  hq-srv.au-team.irpo  hq-srv
192.168.0.1  br-rtr.au-team.irpo  br-rtr
192.168.0.2  br-srv.au-team.irpo  br-srv
192.168.200.3  hq-cli.au-team.irpo  hq-cli
```

Далее по окончанию настройки:
```
systemctl restart dnsmasq
```

После чего, на ВСЕХ машинах, в конфигурационном файле `/etc/resolv.conf` добавляем строку:
```
search au-team.irpo
nameserver 192.168.100.62
```

</details>
  
<br/>

## ✔️ Задание 11 `[ВРЕМЯ/ДАТА]`

### Настройте часовой пояс на всех устройствах, согласно месту проведения экзамена

<br/>

<details>
<summary><strong>Настройка <code>часового</code> пояса</strong></summary>
<br/>

## > Настройте часовой пояс на всех устройствах <
- На Linux настраивается часовой пояс командой:
```
timedatectl set-timezone Asia/Tomsk
```  

- Если на `CLI` возникли проблемы, делаем через настройки в графическом интерфейсе и настраиваем там временную зону

</details>

# <div align="center"><strong>⚙️</strong></div> <div align="center"><strong>МОДУЛЬ 2</strong></div>

<br/>

ДОРАБОТАТЬ
### Задание 1

<details>
<summary><strong> ## Описание задания: </strong></summary>

Настройте контроллер домена Samba DC на сервере BR-SRV: 

•	Имя домена au-team.irpo 

•	Введите в созданный домен машину HQ-CLI 

•	Создайте 5 пользователей для офиса HQ: имена пользователей формата hquser№ (например hquser1, hquser2 и т.д.) 

•	Создайте группу hq, введите в группу созданных пользователей 

•	Убедитесь, 	что 	пользователи 	группы 	hq 	имеют 	право аутентифицироваться на HQ-CLI  

• Пользователи группы hq должны иметь возможность повышать привилегии для выполнения ограниченного набора команд: cat, grep, id. Запускать другие команды с повышенными привилегиями пользователи группы права не имеют. 

</details>

<details>
<summary><strong>Создание контрлера домена </strong></summary>

<br/>

Прописать в файле /etc/hosts ip, доменное имя (полностью) и имя сервера. Как это должно выглядеть:
```
192.168.0.2 br-srv.au-team.irpo br-srv
```

Удаляем старую папку /etc/resolv.conf и создаём её заново:

```
unlink /etc/resolv.conf
```

```
nano /etc/resolv.conf
```
После, прописываем следующие занчениея:

```
nameserver 8.8.8.8

nameserver 1.1.1.1

search br-srv.au-team.irpo
```

</br>

```
apt install samba samba-dsdb-modules samba-vfs-modules acl attr winbind libpam-winbind libnss-winbind krb5-config krb5-user libpam-krb5 smbclient dnsutils net-tools
```

Указываем следующие значения, в следующих трёх появившихся полях, по порядку:

```
AU-TEAM.IRPO
```

```
br-srv.au-team.irpo
```
```
br-srv.au-team.irpo
```

После установки пакетов нам нужно остановить и запретить для автозапуска следующие службы:

```
systemctl disable --now smbd nmbd winbind
```

Все эти функции на себя возьмет служба samba-ad-dc. Если мы не выключим перечисленные сервисы, будет конфликт и ошибка в работе. По идее, они будут заблокированы автоматически (masked0, но лишняя осторожность не повредит.

```
samba-tool domain provision 
```
Удаление/перенос класического конфигурационного фала smb.conf, иначе так же может возникнуть ошибка:

```
mv /etc/samba/smb.conf /etc/samba/smb.conf.backup
```

👉 создаёт контроллер домена

<br/>

<details>
<summary><strong>Если не создался домен, прописываем следующее </strong></summary>

</br>

```
systemctl stop samba-ad-dc smbd nmbd winbind 2>/dev/null
```
```
apt purge samba samba-common-bin samba-ad-dc winbind -y
```
```
apt autoremove -y
```
```
rm -rf /etc/samba
```
```
rm -rf /var/lib/samba
```
```
rm -rf /var/cache/samba
```
Проверяем имя машины на соответствие нужному доменному имени

Проверяем nano /etc/hosts на наличи ip машины своему домену, если нет, прописываем слудующее:

(BR-SRV-IP либо 192.168.xxx.xxx)  br-srv.au-team.irpo br-srv

Проверяем nano /etc/resolv.conf на наличие локального ip, если нет, прописываем:

```
nameserver 192.168.0.2
```
Сохроням и пробуем установить снова

</details>

</br>

Убираем оригинальный конфигурационный файл krb5.conf и заменяем его тем, что сформировала утилита samba-tool:

```
mv /etc/krb5.conf /etc/krb5.conf.backup
```

```
cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

Открываем конфигурационный файл resolv.conf:

```
nano /etc/resolv.conf
```

Удаляем старую запись DNS сервера и заменяем на запись локального сервера:

```
nameserver 192.168.0.2
```
Перезапускаем сервис и включаем его для автозапуска:

```
systemctl restart samba
```

```
systemctl enable --now samba-ad-dc
```

Проверьте статус, если всё работет продолжайте дальше. (Позже посмотрю в техе на ошибки и если найду таковые постараюсь исправить и внести изменения).


<details>
<summary><strong>Проверка работы домена и его обнаружение (не обязательно если уверены в работе)</strong></summary>

Убедимся, что сервер возвращает правильный IP при запросе адреса по домену:

```
host -t A dmosk.local
```

Должен вернуть - au-team.irpo has address 192.168.0.2 
Пользователи и группа

Проверяем dns запись для имени контроллера домена:

```
host -t A au-team.irpo
```
Должен вернуть - au-team.irpo has address 192.168.0.2

Также DNS должен правильно вернуть SRV запись для kerberos:

```
host -t SRV _kerberos._udp.au-team.irpo
```
Должен вернуть - _ldap._tcp.au-team.irpo has SRV record 0 100 389 br-rtr.au-team.irpo

Нужно проверить появление тикета в системе

```
klist
```
Должен вывести: 

<br/>

Ticket cache: FILE:/tmp/krb5cc_0
Default principal: administrator@AU-TEAM.IRPO

Valid starting     Expires            Service principal
12/17/25 16:14:12  12/18/25 02:14:12  krbtgt/AU-TEAM.IRPO@AU-TEAM.IRPO
        renew until 12/18/25 16:14:05

</br>

Так же попытаться аунтифицироваться через администратора

```
kinit administrator
```

Вывод - Warning: Your password will expire in 41 days on Wed Jan 28 15:08:31 2026

</details>

</details>

<details>
<summary><strong>Создание полльзователей</strong></summary>

Создание пользователей от 1 до 5:

```
for i in {1..5}; do samba-tool user create hquser$i P@ssw0rd; done
```

Создание группы hq

```
samba-tool group add hq
```

Добавление пользователей в группу hq

```
for i in {1..5}; do samba-tool group addmembers hq hquser$i; done
```

Sudo ограничения:

<details>
<summary><strong>visudo</strong></summary>



```
apt install sudo -y
```

```
visudo
```

Добавить:

```
%hq ALL=(ALL) NOPASSWD: /bin/cat, /bin/grep, /usr/bin/id
```

</details>

<details>
<summary><strong>Конфиги <code>/etc/sudoers </code>/</strong></summary>

Переходим в конфиги:

```
nano /etc/sudoers
```

Прописываем правило:

```
%hq ALL=(ALL) NOPASSWD: /bin/cat, /bin/grep, /usr/bin/id
```

</details>

</details>

<details>
<summary><strong>Настройка HQ-CLI на домен</strong></summary>

Первым делом, необходимо ввести HQ-CLI в домен, это делается достаточно просто. Заходим в /etc/resolv.conf и прописываем следующие занчения:

```
nameserver 192.168.0.2

search au-team.irpo
```

После устанавливаем  <code>realm </code>, он понадобится нам в большинстве случаев.

```
apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit -y
```

Проверяем нахождение домена на HQ-CLI

```
realm discover au-team.irpo
```

Добавляем дополнительные права пользоватлею HQ-CLI для возможности пользоваться командами <code>realm</code> без пользования <code>root</code> прав. Для этого переходим в каталог ```nano /home/usr1/.bashrc``` и прописываем следующие значения:

```
export PATH=$PATH:/usr/sbin
```

Применяем изминения:

```
source ~/.bashrc
```

После этого, нужно настройить права доступа группы hq. Для этого, разрешаем группе hq вход:

```
realm permit -g hq
```

После, прописываем права доступа на hq-cli в каталоге ```nano /etc/sssd/sssd.conf``` :

```
[sssd]
services = nss, pam
domains = au-team.irpo

[domain/au-team.irpo]
id_provider = ad
auth_provider = ad
access_provider = simple
simple_allow_groups = hq
```

Задаём права "только для чтения" на этот файл:

```
chmod 600 /etc/sssd/sssd.conf
```

Рестартим:

```
systemctl restart sssd
```

Включаем вход через GUI (PAM):

```
pam-auth-update
```

Перезапускаем вход:

```
systemctl restart sssd
```

```
systemctl restart display-manager
```

После, прописывем:

```
hquser1@au-team.irpo
```

Вводим пароль в появившемся окне:

```
P@ssw0rd
```

<details>
<summary><strong>Если не получается зайти на пользователя </strong></summary>

## Если не получается зайти на пользователя из начального экрана, необходимо проверить, что это за ошибка. Для этого заходим в уже имеющегося, прописывам: 

```
su - hquser1@au-team.irpo
```

Если вывод - <code> System eror </code>, то делаем следующее:

```
systemctl stop sssd
```

```
rm -f /etc/krb5.keytab
```

```
rm -rf /var/lib/sss/db/*
```

```
realm leave au-team.irpo --remove
```

И переподключаемся заново:

```
realm join au-team.irpo -U Administrator
```

```
realm permit -g hq
```

```
systemctl restart sssd
```

После, проверяем через: 
```
su - hquser1@au-team.irpo 
```

## Должно появиться следующее сообщение:

<img width="733" height="95" alt="изображение" src="https://github.com/user-attachments/assets/3403a5ff-d63b-4ee6-b1d8-f9e940a01bc2" />

</details>

</details>

Ошибка - нет возможности активировать sudo с пользователя hq. Исправть

## Задание 2 RAID (HQ-SRV)

<br/>

apt install mdadm -y

mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc

👉 RAID0

mkfs.ext4 /dev/md0

mkdir /raid

mount /dev/md0 /raid

echo "/dev/md0 /raid ext4 defaults 0 0" >> /etc/fstab

<br/>

## ЗАДНИЕ 3 NFS

<br/>
HQ-SRV

apt install nfs-kernel-server -y

mkdir -p /raid/nfs

echo "/raid/nfs 192.168.200.0/28(rw,sync)" >> /etc/exports

exportfs -a

systemctl restart nfs-kernel-server

HQ-CLI

apt install autofs -y

echo "/mnt /etc/auto.nfs" >> /etc/auto.master

echo "nfs -rw 192.168.100.62:/raid/nfs" >> /etc/auto.nfs

systemctl restart autofs
<br/>

## Задание 4. NTP (ISP)

<br/>

apt install chrony -y

nano /etc/chrony/chrony.conf

Добавить:

server pool.ntp.org iburst

local stratum 5

allow 0.0.0.0/0

systemctl restart chrony

👉 клиенты: HQ-SRV, HQ-CLI, BR-RTR, BR-SRV — указать IP ISP

<br/>

## Задание 5 . ANSIBLE (BR-SRV)

<br/>

apt install ansible -y

mkdir -p /etc/ansible

nano /etc/ansible/hosts

[hq]

192.168.100.62

192.168.200.2

172.16.4.2

[br]

172.16.5.2

ansible all -m ping

<br/>

## Задание 6. DOCKER (BR-SRV)

<br/>

apt install docker.io docker-compose -y

docker load < site_latest.tar

docker load < mariadb_latest.tar

nano docker-compose.yml

version: '3'

services:
  db:
    image: mariadb_latest
    container_name: db
    environment:
      MYSQL_DATABASE: testdb
      MYSQL_USER: test
      MYSQL_PASSWORD: P@ssw0rd
      MYSQL_ROOT_PASSWORD: root

  testapp:
    image: site_latest
    container_name: testapp
    ports:
      - "8080:80"

docker-compose up -d

<br/>

## Задание 7. WEB (HQ-SRV)

<br/>

apt install apache2 mariadb-server php php-mysql -y

mysql

CREATE DATABASE webdb;

CREATE USER 'web'@'%' IDENTIFIED BY 'P@ssw0rd';

GRANT ALL ON webdb.* TO 'web'@'%';

mysql webdb < dump.sql

cp index.php /var/www/html/

cp -r images /var/www/html/

<br/>

## Задание 8. NAT (ПОРТЫ)

<br/>

HQ-RTR

iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to 192.168.100.62:80

iptables -t nat -A PREROUTING -p tcp --dport 2026 -j DNAT --to 192.168.100.62:22

BR-RTR

iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to 192.168.0.2:8080

iptables -t nat -A PREROUTING -p tcp --dport 2026 -j DNAT --to 192.168.0.2:22

<br/>

## Задаие 9. NGINX PROXY (ISP)

<br/>

apt install nginx apache2-utils -y

htpasswd -c /etc/nginx/.htpasswd WEB

nano /etc/nginx/sites-available/default

server {
  server_name web.au-team.irpo;

  auth_basic "Restricted";
  auth_basic_user_file /etc/nginx/.htpasswd;

  location / {
    proxy_pass http://172.16.4.2:8080;
  }
}

server {
  server_name docker.au-team.irpo;

  location / {
    proxy_pass http://172.16.5.2:8080;
  }
}

systemctl restart nginx

<br/>

## Задание 10. ЯНДЕКС БРАУЗЕР (HQ-CLI)

<br/>

wget https://repo.yandex.ru/yandex-browser/YANDEX-BROWSER-KEY.GPG

apt install ./yandex-browser*.deb

<br/>
