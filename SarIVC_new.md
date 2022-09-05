# Необходимое оборудование для проведение проверок

- Веб-камера

- USB-Токен

- Аудио-гарнитура с микрофоном

- Образ виртуальной машины с Astra Linux 

- Образ виртуальной машины с Windows 10

- Сканер сетевой

- Принтер сетевой

# 1. Cеть (доступ из сетей, порты)

1.1 Проверить работу выделенных **АРМ**ов

- Astra Linux CE:

    - Убедиться, что установленная ОС удовлетворяет требованиям, а **АРМ** заведен в домен, выполнив следующие комманды:

        ```sh
        cat /etc/astra_version # Версия ОС: CE 2.12.43 (orel)
        uname -r # Версия ядра: 5.10.0-1038.40-generic
        astra-freeipa-client -i # Обнаружен настроенный клиент в домене <domain-name>
        ```

    - Убедиться в работе сетевого принтера и сканера

    - Убедиться в работе всех USB-устройств из списка "Необходимое оборудование"

- Windows 10:

    - Убедиться, что машина в домене, проверив настройки **АРМ**

    - Убедиться в работе сетевого принтера и сканера

    - Убедиться в работе всех USB-устройств из списка "Необходимое оборудование"

<br/>

1.2 Разместить **ВМ** в сети VDI и получить к ним доступ с обоих **АРМ**ов

- Запустить **ВМ** с Windows 10 и Astra Linux CE из подготовленных образов, настроить их работу внутри сети **vdi_public**, используя следующие настройки:

    ```sh
    nano /etc/network/interfaces
    # добавляем в файл
    auto eno2 # указываем свой порт
    iface eno2 inet static # указываем свой порт
    address 10.116.208.15
    netmask 255.255.240.000
    gateway 10.116.208.1
    ```

- Проверить доступность обеих **ВМ** с выделенных **АРМ**ов

<br/>

1.3 C предоставленного **АРМ** с Linux подключиться к размещенной **ВМ** c Astra Linux CE по `ssh`, проверить доступность следующих портов:

- 3240(TCP) 5005(UDP) 5007(UDP) 5008(UDP) 6566(TCP) 61000(TCP) - должны быть открыты на **ТК**. Для проверки выполняем следующие команды:

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

2.1 Проверить доступность DNS серверов из сети ПАК с запущенных ВМ

- Доступность из сети **vdi_public**

    ```sh
    # редактируем настройки DNS
    nano /etc/resolv.conf
    # добавляем в файл
    nameserver 10.114.4.105
    nameserver 10.114.4.63
    nameserver 10.114.4.103
    nameserver 10.114.4.104
    # проверяем резолвинг
    nslookup 10.114.4.105 pvr-dc-01.pvrr.oao.rzd
    nslookup 10.114.4.63 pvr-dc-02.pvrr.oao.rzd
    nslookup 10.114.4.103 pvr-dc-03.pvrr.oao.rzd
    nslookup 10.114.4.104 pvr-dc-04.pvrr.oao.rzd
    ```

- Настраиваем VLAN 40

    ```sh
    # правим настройки интерфейса
    nano /etc/network/interfaces
    # добавляем в файл
    auto eno2
    iface eno2 inet static
    address 10.116.192.22
    netmask 255.255.240.000
    gateway 10.116.192.1
    # проверяем резолвинг
    nslookup 10.114.4.105 pvr-dc-01.pvrr.oao.rzd
    nslookup 10.114.4.63 pvr-dc-02.pvrr.oao.rzd
    nslookup 10.114.4.103 pvr-dc-03.pvrr.oao.rzd
    nslookup 10.114.4.104 pvr-dc-04.pvrr.oao.rzd
    ```

- Проверяем доступность необходимых ресурсов

    ```sh
    nmap -sn 10.246.101.234 10.246.102.165 10.114.3.20 — 10.114.3.29 10.248.69.24 — 10.248.69.28 10.48.25.99 10.247.129.3 — 10.247.129.9 10.248.69.29
    ```

2.3 Проверка доступности **NTP** сервиса

- С настроенной **ВМ** выполнить следующие команды:

    ```sh
    ping 10.114.4.128
    ping 10.114.4.129
    nslookup pvrr-ntp-01.pvrr.oao.rzd 
    nslookup pvrr-ntp-02.pvrr.oao.rzd 
    ```

# 3. Домен MS AD

3.1 Операция ввода в домен ВМ

- Проверяем доступность аккаунта администратор и самого доменного сервера **AD** для предоставленных учетных данных:

    ```sh
    ldapsearch -H ldap://<domain_server_ip> -D "CN=<domain_admin_name>,OU=VIRAZH,OU=Service Accounts,DC=pvrr,DC=oao,DC=rzd" -b "ou=VIRAZH,ou=Service Accounts,dc=pvrr,dc=oao,dc=rzd" "(objectclass=person)" sAMAccountName -w <domain_admin_pass> | grep result # код успешной работы запроса - result: 0 success
    ```

- На **ВМ** с Windows 10 запускаем консоль от имени Администратора, вводим следующее:

    ```sh
    netdom join %computername% /domain:<domain_server_name> /userd:<domain_admin_name>\administrator /passwordd:<domain_admin_pass>
    shutdown /r
    ```

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
