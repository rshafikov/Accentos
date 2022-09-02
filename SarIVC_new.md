# 1. Cеть (доступ из сетей, порты)


1.1
- Astra Linux CE:

    \- Убедиться, что установленная ОС удовлетворяет требованиям для корректной работы **RS-client**, выполнив следующие комманды:

    ```sh
    cat /etc/astra_version # Версия ОС: CE 2.12.43 (orel)
    uname -r # Версия ядра: 5.10.0-1038.40-generic
    astra-freeipa-client -i # Обнаружен настроенный клиент в домене <domain-name>
    ```

    \- Убедиться в работе сетевого принтера и сканера

<br/>

- Windows 10:

    \- Убедиться, что машина в домене, проверив настройки ТК

    \- Убедиться в работе сетевого принтера и сканера


1.2 Разместить **ВМ** в сети VDI и получить к ним доступ с **ТК**

- Запустить **ВМ** с Windows 10 и Astra Linux CE из подготовленных образов, настроить их работу внутри сети vdi_public

- С выделеных **ТК** проверить доступность обеих ВМ

1.3 C предоставленного АРМ с Linux подключиться к размещенной **ВМ** c Astra Linux CE по ssh, проверить доступность следующих портов:

- 3240(TCP) 5005(UDP) 5007(UDP) 5008(UDP) 6566(TCP) 61000(TCP) - должны быть открыты на **ТК**. Для проверки заходим на **ВМ** по ssh и выполняем следуюие команды:

    ```sh
    for port in {3240,6566,61000}; do nmap -p $port 10.23.213.211; done
    nc -lu 5005 # запустить netcat на ТК
    nc -u $IP_TK 5005 # запустить netcat на ВМ, убедиться, что пакеты отправляются. Сделать тоже самое для остальных портов
    ```

- 5000(UDP) 5001(UDP) 5002(UDP) 5003(UDP) - должны быть открыты на ВМ

    ```sh
    nc -lu 5000 # запустить netcat на ВМ
    nc -u $IP_TK 5000 # запустить netcat на ТК убедиться, что пакеты отправляются. Сделать тоже самое для остальных портов
    ```

1.4 Получить досуп к [**СХД**](http://10.114.30.162) из сети управления

- Зайти в браузер на [**СХД**](http://10.114.30.162) используя следующие учетные данные: accentos/P@ssw0rd

- Убедиться, что во вкладке "POOLS" находятся два пула

- Убедиться, что во вкладке "VOLUMES" находятся именновые **LUN**ы:

    - nova1
    - nova2
    - gnocci
    - glance

<br/>

# 2. Cервисы сети (проверка доступности)

2.1 Проверить доступность DNS серверов из сети ПАК

- Настраиваем VLAN30

    ```sh
    # правим настройки интерфейса
    nano /etc/network/interfaces
    # добавляем в файл
    auto eno2 # указываем свой порт
    iface eno2 inet static # указываем свой порт
    address 10.116.208.1
    netmask 255.255.240.000
    gateway 10.116.208.1
    # редактируем настройки DNS
    nano /etc/resolv.conf
    # добавляем в файл
    nameserver 10.114.4.105
    nameserver 10.114.4.63
    nameserver 10.114.4.103
    nameserver 10.114.4.104
    ```

- Настраиваем VLAN 40

    ```sh
    # правим настройки интерфейса
    nano /etc/network/interfaces
    # добавляем в файл
    auto eno2
    iface eno2 inet static
    address 10.116.192.1
    netmask 255.255.240.000
    gateway 10.116.192.1
    ```
- Проверяем VLAN 10

    ```sh
    # правим настройки интерфейса
    nano /etc/network/interfaces
    # добавляем в файл
    auto eno2
    iface eno2 inet static
    address 172.21.0.1
    netmask 255.255.248.000
    gateway 172.21.0.1
    ```

    




2.3 Проверка доступности **NTP** сервиса

- С любого **ВУ** выполнить следующие команды:

    ```sh
    ping 10.114.4.128
    ping 10.114.4.129
    nslookup pvrr-ntp-01.pvrr.oao.rzd 
    nslookup pvrr-ntp-02.pvrr.oao.rzd 
    # если все пингуется и резолвится, убедиться в корректности работы следующей команды:
    sudo chronyc tracking
    ```

<br/>

# 3. Домен MS AD

3.1 Операция ввода в домен ВМ

- Проверяем доступность аккаунта администратор и самого доменного сервера **AD** для предоставленных учетных данных:

    ```sh
    ldapsearch -w <domain_admin_pass> -H ldap://<domain_server_ip> -D "CN=<domain_admin_name>,OU=VIRAZH,OU=Служебные УЗП и ЭПС,OU=ЦКСВТ-АДМИН-ПРО,OU=ЦК СВТ,OU=Services,DC=msk,DC=oao,DC=rzd" -b "OU=VIRAZH,OU=Служебные УЗП и ЭПС,OU=ЦКСВТ-АДМИН-ПРО,OU=ЦК СВТ,OU=Services,DC=msk,DC=oao,DC=rzd" "(objectclass=*)" | grep result # код успешной работы запроса - result: 0 success
    ```

- На **ВМ** с Windows 10 запускаем консоль от имени Администратора, вводим следующее:

    ```sh
    netdom join %computername% /domain:<domain_server_name> /userd:<domain_admin_name>\administrator /passwordd:<domain_admin_pass>
    shutdown /r
    ```

<br/>

# 4. Домен FreeIPA

4.1 Операция ввода в домен ВМ 

- Проверяем доступность аккаунта администратор и самого доменного сервера **FreeIPA** для предоставленных учетных данных:

    ```sh
    ldapsearch -w <domain_admin_pass> -H ldap://<domain_server_ip> -D "CN=<domain_admin_name>,OU=VIRAZH,OU=Служебные УЗП и ЭПС,OU=ЦКСВТ-АДМИН-ПРО,OU=ЦК СВТ,OU=Services,DC=msk,DC=oao,DC=rzd" -b "OU=VIRAZH,OU=Служебные УЗП и ЭПС,OU=ЦКСВТ-АДМИН-ПРО,OU=ЦК СВТ,OU=Services,DC=msk,DC=oao,DC=rzd" "(objectclass=*)" | grep result # код успешной работы запроса - result: 0 success
    ```

- На **ВМ** с Astra Linux CE запускаем консоль, вводим следующее:

    ```sh
    astra-freeipa-client -u <domain_admin_name> -p <domain_admin_pass> --par "--domain domain_server_name --server domain_server_name_1 --server domain_server_name_2" -y
    ```
    