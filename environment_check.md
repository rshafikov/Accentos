# Необходимое оборудование для проведение проверок

- Веб-камера

- USB-Токен

- Аудио-гарнитура с микрофоном

- Сканер сетевой

- Принтер сетевой


# 1. Проверка пользователей, АРМ, доступ из сетей, порты



1.1 Проверить работу выделенных АРМов и ВМ АРМ Администратора ВИРАЖ:

- ВМ АРМ Администратора ВИРАЖ:

    Убедиться в доступе с данной ВМ ко всем подсетям ВИРАЖа и оборудованию в стойке.
            
    ```sh
    kvm_internal, ipmi_net, vdi_public
    nmap -sn <ipmi_address_net>/<mask>_
    # после пунктов 1.2 - 1.3
    nmap -sn <kvm_internal_net>.51
    nmap -sn <vdi_public_net>.100
    nslookup controller.<domain>.loc
    ```

- Проверяем доступность аккаунтов пользователей и самого доменного сервера **MSAD** для предоставленных учетных данных:

    ```sh
    ldapsearch -H ldap://<domain_server_ip> -D "CN=<domain_user_name>,OU=VIRAZH,OU=Service Accounts,DC=<domain>,DC=oao,DC=rzd" -b "ou=VIRAZH,ou=Service Accounts,dc=<domain>,dc=oao,dc=rzd" "(objectclass=person)" sAMAccountName -w '<domain_user_password>' | grep result 
    # код успешной работы запроса - result: 0 success
    ```


- Проверяем доступность аккаунтов пользователей и самого доменного сервера **FreeIPA** для предоставленных учетных данных:

    ```sh
	ldapsearch -w <domain_user_password> -H ldap://<domain_server_ip> -D "uid=<domain_user_name>,cn=users,cn=accounts,dc=crp,dc=rzd" -b "cn=group,cn=accounts,dc=crp,dc=rzd" "(objectclass=person)" | grep result 
    # код успешной работы запроса - result: 0 success
    ```

- АРМ Astra Linux CE:

    - Убедиться, что установленная ОС удовлетворяет требованиям, а **АРМ** заведен в домен, выполнив следующие комманды:

        ```sh
        cat /etc/astra_version # Версия ОС: CE 2.12.44 (orel)
        uname -r # Версия ядра: 5.10.0-1038.40-generic
        astra-freeipa-client -i # Обнаружен настроенный клиент в домене <domain-name>
        # ввести в домен под учетными записями выданными РЖД
        astra-freeipa-client -u <domain_admin_name> -p <domain_admin_pass> --par "--domain <domain_server_name> --server <domain_server_name_1> --server <domain_server_name_2>" -y
        ```

    - Убедиться в работе всех устройств из списка "Необходимое оборудование"
	Rutoken - https://wiki.astralinux.ru/pages/viewpage.action?pageId=32834416
	https://forum.rutoken.ru/topic/1644/

- АРМ Windows 10:

    - Убедиться, что машина в домене, проверив настройки АРМ, ввести в домен под учетными записями выданными РЖД

    - Убедиться в работе всех устройств из списка "Необходимое оборудование"


1.2 Настроить первый сервер в стойке ВИРАЖ на сеть **kvm_internal** и получить к нему доступ по ssh с **ВМ АРМ Администратора ВИРАЖ**

- Настроить сеть используя следующие настройки:

    ```sh
    # /etc/network/interfaces
    auto bond0
    iface bond0 inet static
        bond-slaves eth0 eth1
        bond-mode 802.3ad   
        address <kvm_internal_net>.51/<mask>
        gateway <kvm_internal_net>.1
        dns-nameservers <dc1___.oao.rzd>  <dc1___.crp.rzd>
    ```

- добавить временный репозиторий:

    ```sh
    sudo echo "deb [trusted=yes] http://repo-<domain>.crp.rzd/orel stable main contrib non-free" >> /etc/apt/sources.list
    ```

- установить дополнительные пакеты:

    ```sh
    sudo apt update && sudo apt install ldap-utils nmap telnet -y
    # перезагрузить сервер
    sudo reboot
    ```

1.3 Настроить последний сервер в стойке ВИРАЖ на сеть VDI и получить к нему доступ с обоих **АРМ**ов

- Настроить сеть используя следующие настройки:

    ```sh
    # /etc/network/interfaces
    auto bond0
    iface bond0 inet manual
        bond-slaves eth0 eth1
        bond-mode 802.3ad
        address <kvm_internal_net>.11/<mask>
    # добавляем VLAN30
    auto bond0.30
    iface bond0.30 inet static
        vlan-raw-device bond0
        address <vdi_public_net>.100/<mask>
        gateway <vdi_public_net>.1
        dns-nameservers <dc1___.oao.rzd>  <dc1___.crp.rzd>

- добавить временный репозиторий:

    ```sh
	sudo echo "deb [trusted=yes] http://repo-<domain>.crp.rzd/orel stable main contrib non-free" >> /etc/apt/sources.list
    ```

- установить дополнительные пакеты:

    ```sh
	sudo apt update && sudo apt install ldap-utils nmap telnet -y
    # перезагрузить сервер
    sudo reboot
    ```

- Выполнить проверки из пункта 1.1


1.4 C предоставленного АРМ с AstraLinux подключиться к серверу настроенному в сети **vdi_public** по `ssh`, проверить доступность следующих портов:

- 3240(TCP) 5005(UDP) 5007(UDP) 5008(UDP) 6566(TCP) 61000(TCP) - должны быть открыты на **АРМ** с AstraLinux. Для проверки выполняем следующие команды:
	- Проверка доступа с сервера на **АРМ** с AstraLinux
	- для UDP портов
    nc -lu <PORT> # запустить netcat на **АРМ** с AstraLinux
    nc -u <IP_TK> <PORT> # запустить netcat на сервере, ввести любой надор символов, убедиться, что пакеты отправляются. Сделать тоже самое для остальных портов

	- для TCP портов
    nc -l <PORT> # запустить netcat на **АРМ** с AstraLinux
    nc <IP_TK> <PORT> # запустить netcat на сервере, ввести любой набор символов, убедиться, что пакеты отправляются. Сделать тоже самое для остальных портов

	Провести проверку в обратную сторону с **АРМ** с AstraLinux на сервер.

1.5 Проверить досуп к [**СХД**]

- Проверить досуп к [**СХД**] с первого сервера из сети управления kvm_internal

    nmap <IP_СХД>

- На **ВМ-Администратора** зайти в браузер на [**СХД**] используя следующие учетные данные: accentos/P@ssw0rd

- Убедиться, что во вкладке "POOLS" находятся два пула

- Убедиться, что во вкладке "VOLUMES" находятся именновые **LUN**ы:

    - nova1
    - nova2
    - gnocci
    - glance

# 2. Cервисы сети (проверка доступности)

2.1 Проверить доступность DNS и NTP серверов c настроеныз серверов из сети ПАК 

- Доступность из сетей **kvm_internal** и **vdi_public**:

    ```sh
    # проверям резолвинг
    nslookup <msad-domain-1>.oao.rzd
    nslookup <msad-domain-2>.oao.rzd
    nslookup <msad-domain-3>.oao.rzd
    nslookup <msad-domain-4>.oao.rzd
    nslookup <ipa-domain-1>.crp.rzd
    nslookup <ipa-domain-2>.crp.rzd
    # проверяем доступность NTP
    # добавляем настройки NTP сервера коммандами:
    echo "server <IP_NTP1> iburst" >> /etc/chrony/chrony.conf
    echo "server <IP_NTP2> iburst" >> /etc/chrony/chrony.conf
    # проверяем доступность NTP серверов командой
    chronyc sources

- Проверяем доступность необходимых ресурсов из kvm_internal (Zabbix, NetBackup, Tivoli):

    ```sh
	telnet <IP_Zabbix> 10051
	nmap -Pn <IP_NetBackup> 
	nmap -sn <IP_Tivoly> 
    ```

- Проверяем доступность портов для Самбы 149 и 445:

Необходимо добавить!

# 3. Проверка доступа к доменам MSAD и Freeipa из сетей **kvm_internal** и **vdi_public** с настроеных серверов внутри ПАК:

- Проверяем доступность аккаунта администратор и самого доменного сервера **AD** для предоставленных учетных данных:

    ```sh
	ldapsearch -H ldap://<domain_server_ip> -D "CN=<domain_admin_name>,OU=VIRAZH,OU=Service Accounts,DC=<domain>,DC=oao,DC=rzd" -b "ou=VIRAZH,ou=Service Accounts,dc=<domain>,dc=oao,dc=rzd" "(objectclass=person)" sAMAccountName -w '<domain_admin_password>' | grep result # код успешной работы запроса - result: 0 success
    ```

- Проверяем доступность аккаунта администратор и самого доменного сервера **FreeIPA** для предоставленных учетных данных:

    ```sh
	ldapsearch -w <domain_admin_password> -H ldap://<domain_server_ip> -D "uid=<domain_admin_name>,cn=users,cn=accounts,dc=crp,dc=rzd" -b "cn=group,cn=accounts,dc=crp,dc=rzd" "(objectclass=person)" | grep result # код успешной работы запроса - result: 0 success
    ```
