## Linux network.
Отчет о конфигурации сети в системе Linux.
<!-- <images src="./img/ubuntu_logo.png" alt="drawing" width="40"/>
![Ubuntu_logo](./images/ubuntu_logo.png) -->
## Содержание
1. [Part 1. Инструмент ipcalc](#part-1-installation-of-the-os)
2. [Part 2. Статическая маршрутизация между двумя машинами](#part-2-creating-a-user)
3. [Part 3. Утилита iperf3](#part-3-setting-up-the-os-network)
4. [Part 4. Сетевой брандмауэр](#part-4-os-update)
5. [Part 5. Статическая маршрутизация сети](#part-5-using-the-sudo-command)
6. [Part 6. Динамическая настройка IP с использованием DHCP](#part-6-installing-and-configuring-the-time-service)
7. [Part 7. NAT](#part-7-nat)
8. [Part 8. Бонус. Введение в SSH-туннели](#part-8-installing-and-basic-setup-of-the-sshd-service)

## Part 1. Инструмент ipcalc

* Если у вас не установлен ipcalc, запустите `sudo apt install ipcalc`.

### 1.1 Netwroks and Masks

1. 192.167.36.54/13 сетевой адрес:\
![1](./img/1.png)

2. Преобразование маски 255.255.255.0 в префиксную и двоичную, /15 в обычную и двоичную, 11111111.11111111.11111111.11110000 в обычную и префиксную:

* 255.255.255.0 в префикс: /24 255.255.255.0 в двоичный файл: 11111111.11111111.11111111.00000000\
![2](./img/2.png)

* /15 в обычный: 255.254.0.0 /15 в двоичный: 11111111.11111110.00000000.00000000\
![3](./img/3.png)

* 1111111111.11111111.1111.11110000 до нормального состояния: 255.255.255.240 11111111.11111111.1111.1111.11110000 до префикса: /28
![4](./img/4.png)

3. Минимальный и максимальный хост в сети 12.167.38.4 с масками: /8, 11111111.11111111.00000000.00000000, 255.255.254.0 и /4:\
![5](./img/5.png)

### 1.2 localhost

- 194.34.23.100 - нет
- 127.0.0.2 - да
- 127.1.0.1 - да
- 128.0.0.1 - нет

### 1.3 Диапазоны и сегменты сетей

1. Какие из перечисленных IP-адресов могут использоваться как общедоступные, а какие только как частные:

- 10.0.0.45 - private
- 134.43.0.2 - public
- 192.168.4.2 - private
- 172.20.250.4 - private
- 172.0.2.1 - public
- 192.172.0.1 - public
- 172.68.0.2 - public
- 172.16.255.255 - private
- 10.10.10.10 - private
- 192.169.168.1 - public

2. Какие из перечисленных IP - адресов шлюза возможны для сети 10.10.0.0/18:

- 10.0.0.1 - нет
- 10.10.0.2 - да
- 10.10.10.10 - да
- 10.10.100.1 - нет
- 10.10.1.255 - да

## Part 2. Статическая маршрутизация между двумя машинами

* Для этой задачи важно клонировать виртуальную машину и установить внутреннюю сеть на обоих:\
![settings](./img/2.settings.png)

* Вывод `ip a`:\
![init](./img/2.init.png)

* Чтобы проложить сеть между машинами, нам нужно настроить netplan с помощью running `mcedit /etc/netplan/00-installer-config.yaml`:
![yaml](./img/2.yaml.png)

* Запустите `sudo netplan apply`, чтобы применить настройки на обеих машинах:\
![apply](./img/2.netplan_apply.png)

### 2.1. Добавление статического маршрута вручную

* Для ручной настройки статического маршрута выполните `ip r add [ip of another machine] dev enp0s8`:
![ping](./img/2.ping.png)

Как показано на скриншоте, компьютеры успешно обмениваются данными друг с другом.

### 2.2. Добавление статического маршрута с сохранением

* Запустите `mcedit /etc/netplan/00-installer-config.yaml` и добавьте маршруты, подобные этому: 
![yaml_upd](./img/2.yaml_upd.png) 

* Повторное применение настроек и проверка ping:\
![ping_upd](./img/2.upg_ping.png)

## Part 3. Утилита iperf3
### 3.1 Скорость соединения
``` brew
8Mbps == 1MB/s;
100MB/s == 800000 Kbps;
1Gbps == 1000 Mbps.
``` 

### 3.2 Утилита iperf3

* Чтобы проверить скорость соединения через iperf3, запустите::
``` brew
sudo iperf3 -s
``` 
на ws1;
``` brew
sudo iperf3 -c 192.168.100.10
``` 
на ws2.
![speed](./img/3.speed.png)

## Part 4. Сетевой брандмауэр
### 4.1 Утилита iptables

* Если еще не установлен, запустите `sudo apt install iptables`.

* Создайте брандмауэр /etc/.файл sh, имитирующий брандмауэр на ws1 и ws2:
![firewall](./img/4.firewall.png)

* Запустите файлы на обеих машинах с помощью chmod +x /etc/firewall.sh и /etc/firewall.команды sh:
![chmod](./img/4.chmod.png)
    Разница между стратегиями, используемыми в первом и втором файлах, заключается в том, что мы сначала настроили drop на ws1, поэтому этот компьютер не может быть пропингован.

### 4.2 Утилита nmap
![nmap](./img/4.nmap.png)

Как видно на скриншоте, ws1 не отвечает на запросы, хотя его хост запущен, как мы проверили с помощью nmap.

## Part 5. Статическая маршрутизация сети

* Запустите пять виртуальных машин (3 рабочие станции (ws11, ws21, ws22) и 2 маршрутизатора (r1, r2))

* В настройках виртуальной машины настройте сеть должным образом:

    - Для r1: `Adapter 1 (Internal Network, r1-ws11), Adapter 2 (Internal Network, r1-r2), Adapter 3 (NAT)`
    - Для r2: `Adapter 1 (Internal Network, r1-r2), Adapter 2 (Internal Network, r2-ws21-ws22), Adapter 3 (NAT)`
    - Для ws11: `Adapter 1 (Internal Network, r1-ws11), Adapter 2 (NAT)`
    - Для ws21, ws22: `Adapter 1 (Internal Network, r2-ws21-ws22), Adapter 2 (NAT)`

### 5.1. Настройка адресов компьютеров

* Настройте конфигурации компьютера в etc/netplan/00-installer-config.yaml в соответствии со скриншотами:

*r1*
![r1](./img/r1.png)

*r2*
![r2](./img/r2.png)

*ws11*
![ws11](./img/ws11.png)

*ws21*
![ws21](./img/ws21.png)

*ws22*
![ws22](./img/ws22.png)

* Перезапустите сетевую службу через `sudo netplan apply`.

* Проверьте правильность адреса компьютера с помощью команды `ip -4 a`.

*r1*
![ipar1](./img/ipar1.png)

*r2*
![ipar1](./img/ipar2.png)

*ws22*
![ipar1](./img/ipaws22.png)

* Ping ws22 из ws21:\
![ws21ping](./img/ws21ping.png)

* Ping r1 из ws11:\
![ws21ping](./img/ws11ping.png)

### 5.2. Включение переадресации IP

* Чтобы включить переадресацию IP, выполните следующую команду на маршрутизаторах:
`sysctl -w net.ipv4.ip_forward=1`.
![forward](./img/routersforward.png)
    *При таком подходе переадресация не будет работать после перезагрузки системы.*

Откройте файл /etc/sysctl.conf и раскомментируйте следующую строку:
`net.ipv4.ip_forward = 1`.
![uncommented](./img/uncommented.png)
    *При таком подходе переадресация IP-адресов включена постоянно.*

### 5.3. Конфигурация маршрута по умолчанию

* Настройте маршрут (шлюз) по умолчанию для рабочих станций. Для этого добавьте routes перед IP маршрутизатора в файле конфигурации:

    -*ws11*
    ![ws11](./img/ws11routes.png)

    -*ws21*
    ![ws21](./img/ws21routes.png)

    -*ws22*
    ![ws22](./img/ws22routes.png)

* Примените с помощью `sudo netplan apply` и проверьте, сработало ли это с `ip r`:
    ![ipr3](./img/ipr3.png)

* Ping ws11 -> r2:
    ![ping](./img/5_ping.png)

### 5.4. Добавление статических маршрутов

* Добавить статические маршруты в r1 и r2 в файле конфигурации:

    -*r1*
    ![r1](./img/5_4r1.png)

    -*r2*
    ![r2](./img/5_4r2.png)

* Вызовите ip r и покажите таблицы маршрутов на обоих маршрутизаторах:
![ipr](./img/5_4ipr.png)

* Запустите команды ip r list 10.10.0.0/[netmask] и ip r list 0.0.0.0/ 0 на ws11:

![iprlist](./img/5_4iprlist.png)

* Объясните в отчете, почему для 10.10.0.0/[netmask] был выбран другой маршрут, отличный от 0.0.0.0/0, хотя это мог быть маршрут по умолчанию.
    ```
    Адрес 0.0.0.0/0 обычно означает "любой адрес", следовательно, команда ipr list 0.0.0.0/0 выведет все маршруты, доступные на этом устройстве. Результат команды ip r list 10.10.0.0/18 будет отличаться, поскольку этот адрес уже есть в сети устройства, и нет необходимости обращаться к маршрутизатору для дальнейшей передачи пакета.
    ```

### 5.5. Составление списка маршрутизаторов

* Запустите команду дампа tcpdump -tnv -i enp0s3 на r1

![tcpdump](./img/5_5r1.png)

* Используйте утилиту traceroute для составления списка маршрутизаторов по пути ws11 -> ws21
    ```
    traceroute 10.20.0.10
    ```
![traceroute](./img/5_5traceroute.png)

### 5.6. Использование протокола ICMP при маршрутизации

* Запустите на r1 сбор сетевого трафика, проходящего через eth0, с помощью  `tcpdump -n -i enp0s3` команды icmp.

    ```
    tcpdump -tn -i enp0s3
    ```
![tcpdumb](./img/5_6tcpdump.png)

* Пропингуйте несуществующий IP (например, 10.30.0.111) из ws11 с помощью`ping -c 1 10.30.0.111` команды.

    ```
    ping -c 1 10.30.0.111
    ```
![ping](./img/5_6ping.png)

## Part 6. Динамическая настройка IP с использованием DHCP

* Укажите адрес маршрутизатора по умолчанию, DNS-сервер и адрес внутренней сети. Вот пример файла для r2:

    ```
    sudo mcedit /etc/dhcp/dhcpd.conf
    ```

    ![dhcp](./img/6dhcpd.png)


* Определите сервер имен как `8.8.8.8` в resolv.conf:

    ```
    sudo mcedit /etc/resolv.conf
    ```

    ![resolv](./img/6resolv.png)


* Перезапустите службу DHCP:

    ```
    systemctl restart isc-dhcp-server
    ```

    ![server](./img/6systemctl.png)

    - Ping ws21->ws22:

    ![ping](./img/6ws21ping.png)


* Настройте MAC-адрес ws11 в настройках виртуальной машины и в netplan: 

    ![mac](./img/6mac.png)

    ![w11yaml](./img/6w11yaml.png)


* Настройте r1 так же, как r2, но сделайте назначение адресов строго привязанным к MAC-адресу (ws11). Запустите те же тесты:

    ```
    sudo mcedit /etc/dhcp/dhcpd.conf
    ```

    ![dhcp](./img/6r1dhcpd.png)

    ```
    sudo mcedit /etc/resolv.conf
    ```

    ![resolv](./img/6resolv.png)

    ```
    systemctl restart isc-dhcp-server
    ```

    ![server](./img/6r1serv.png)

    Новый IP-адрес ws11:

    ![ws11newip](./img/6ws11after.png)


* Запросить обновление ip -адреса у ws21:

    - Запустите `sudo dhclient -r enp0s3`,  чтобы удалить старый ip.

    - Запустите `sudo dhclient -v enp0s3`, чтобы запросить новый ip 

    Перед обновлением:
    
    ![ws21before](./img/6ws21before.png)

    После обновления:

    ![ws21after](./img/6ws21after.png)


## Part 7. NAT

* В файле /etc/apache2/ports.conf измените строку Listen 80 на Listen 0.0.0.0:80 на ws22 и r1, т.е. сделайте сервер Apache2 общедоступным:

    ```
    sudo mcedit /etc/apache2/ports.conf
    ```
    ![ports](./img/7_ws2280.png)


* Запустите веб - сервер Apache с помощью service apache2 start команды на ws22 и r1:

    ![apache2start](./img/7_apache2start.png)


* Добавьте следующие правила в брандмауэр, созданный аналогично брандмауэру из части 4, на r2:

    1) удалить правила в таблице фильтров - iptables -F

    2) удалить правила в таблице "NAT" - iptables -F -t nat

    3) удалить все маршрутизируемые пакеты - iptables --политика ПРЯМОГО УДАЛЕНИЯ

    ![firewall](./img/7_r2firewall.png)

* Запустите файл:

    ![chmod](./img/7_r2chmod.png)

    *При запуске файла с этими правилами ws22 не должен выполнять пинг с r1*

* Ping w22->r1:

    ![ping](./img/7_ping.png)

    *Как видно на скриншоте, до обновления брандмауэра ping работал просто отлично. Теперь брандмауэр блокирует его..*

* Добавьте в файл еще одно правило:

    разрешить маршрутизацию всех пакетов протокола ICMP

    ![filewall2](./img/7_r2filewall4rule.png)

* Запустите файл:

    ![runagain](./img/7_runagain.png)

* Проверьте соединение между ws22 и r1 с помощью команды `ping`:

    ![ping2](./img/7_ping2.png)

* Добавьте в файл еще два правила:

   включить SNAT, который маскирует все локальные ip-адреса из локальной сети за r2 (как определено в части 5 - сеть 10.20.0.0)
    ```
    Tip: стоит подумать о маршрутизации внутренних пакетов, а также внешних пакетов с установленным соединением
    ```

    включите DNAT на порту 8080 компьютера r2 и добавьте внешний сетевой доступ к веб -серверу Apache, работающему на ws22
    ```
    Tip: имейте в виду, что при попытке подключения появится новое tcp-соединение для ws22 и порта 80
    ```

    ![firewall3](./img/7_r2firewall3.png)

* Проверьте TCP-соединение на наличие SNAT, подключившись из ws22 к серверу Apache на r1 с помощью команды telnet [address] [port]:

    ![ws22telnet](./img/7_ws22telnet.png)

* Проверьте TCP-соединение на наличие DNAT, подключившись с r1 к серверу Apache на ws22 с помощью команды telnet (адрес r2 и порт 8080):

    ![r1](./img/7_r1telnet.png)


## Part 8. Бонус. Введение в SSH-туннели

* Запустите брандмауэр на r2 с правилами из части 7:

    ![firewall](./img/8_firewall.png)

* Запустите веб-сервер Apapche на ws22 только на localhost (т.е. В файле /etc/apache2/ports.conf измените строку Listen 80 на Listen localhost:80)

    ![apache](./img/8_apache.png)

* Используйте локальную переадресацию TCP с ws21 на ws22 для доступа к веб-серверу на ws22 из ws21:

    ![localssh](./img/8_localssh.png)

* Используйте удаленную переадресацию TCP с ws11 на ws22 для доступа к веб-серверу на ws22 из ws11:

    ![remotessh](./img/8_remotessh.png)

* Чтобы проверить, работало ли соединение на обоих предыдущих этапах, перейдите ко второму терминалу (например, с помощью Alt + F2) и запустите `telnet 127.0.0.1 [local port]` команду:

    ![check](./img/8_final.png)