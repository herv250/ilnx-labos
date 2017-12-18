# Labo 4: Webserver/LAMP stack

In dit labo zullen we een webserver opzetten in de VM die je in het vorige labo gemaakt hebt.
Een van de populairste toepassingen van Linux als server is de zgn. LAMP-stack. Deze afkorting staat voor Linux + Apache + MySQL + PHP. De combinatie vormt een platform voor het ontwikkelen van webapplicaties waar vele bekende websites gebruik van maken (bv. Facebook).

Als je gebruik maakt van andere bronnen (bv. blog-artikel of HOWTO die je op het Internet vond), voeg die dan toe aan je logboek. Zo kan je die later makkelijk terug vinden.

## De Apache webserver installeren

Het is belangrijk dat je controleert voordat je aan dit labo begint, dat je twee netwerkinterfaces hebt op je virtuele machine. De ene moet van het type *NAT* zijn. Deze heeft verbinding met het internet en heeft typisch als IP-adres 10.0.2.15. De andere netwerkinterface moet van het type *Host-only* zijn. Via deze kan je communiceren met de host-machine en je webserver testen. Als je niets hebt veranderd de standaardinstellingen van VirtualBox, is het IP-adres hoogstwaarschijnlijk 192.168.56.101.

1. Installeer Apache op je virtuele machine en verifieer dat hij draait en vanop je host-machine bereikbaar is.
2. Installeer ondersteuning voor PHP en verifieer dat dit werkt, bijvoorbeeld met een eenvoudige PHP-pagina

### Procedure

Beschrijf hier de exacte procedure hoe je dit uitgevoerd hebt. Zorg er voor dat je aan de hand van je beschrijving deze taken later heel vlot kan herhalen als dat nodig is. Test ook telkens na elke stap dat die correct verlopen is.

1. sudo dnf install httpd
2. sudo systemctl start httpd
3. systemctl status httpd
4. ifconfig | grep inet
5. systemctl enable httpd # to start on boot
6. sudo dnf install php php-mysqlnd php-gd php-mcrypt php-mbstring
7. echo "<?php phpinfo(); ?>" > /var/www/html/info.php

## MariaDB (MySQL)

MariaDB is de naam van een variant (fork) van de bekende database MySQL. Onder Fedora is MySQL zelf niet meer beschikbaar, maar MariaDB is volledig compatibel. Installeer MariaDB op je virtuele machine. Voer daarna het script `mysql_secure_installation` uit om het root-wachtwoord voor MariaDB in te stellen. Installeer ook PHPMyAdmin, dit is een webinterface voor het beheer van MySQL/MariaDB.


### Procedure

Beschrijf hier de exacte procedure hoe je dit uitgevoerd hebt. Zorg er voor dat je aan de hand van je beschrijving deze taken later heel vlot kan herhalen als dat nodig is. Test ook telkens na elke stap dat die correct verlopen is.

1. sudo dnf install mariadb-server
2. sudo systemctl start mariadb
3. systemctl enable mariadb
4. sudo mysql_secure_installation
5. sudo dnf install phpMyAdmin

## Webapplicatie

Kies een webapplicatie gebaseerd op PHP en installeer op je webserver. Enkele voorbeelden die je kan gebruiken: Drupal, Wordpress, Joomla, MediaWiki, enz.,


### Procedure

Beschrijf hier de exacte procedure hoe je dit uitgevoerd hebt. Zorg er voor dat je aan de hand van je beschrijving deze taken later heel vlot kan herhalen als dat nodig is. Test ook telkens na elke stap dat die correct verlopen is.

1. sudo dnf install @"Web Server" drupal8 drupal8-httpd php-opcache php-mysqlnd mariadb-server
2. sudo systemctl enable httpd.service mariadb.service #maakt dat het runt bij het opstarten van OS
3. sudo systemctl start httpd.service mariadb.service
4. sudo mysqladmin create myd8site -u root -p
5. sudo mysql -D mysql -u root -p
6. GRANT ALL PRIVILEGES ON myd8site.* TO 'sqluser'@'localhost' IDENTIFIED BY 'password'; # vervang password tussen aanhalingstekens door een wachtwoord
7. FLUSH PRIVILEGES;
8. QUIT;
9. sudo setsebool -P httpd_can_network_connect_db=1
10. sudo setsebool -P httpd_can_sendmail=1
11. sudo sed -i 's/Require local/Require all granted/' /etc/httpd/conf.d/drupal8.conf
12. sudo firewall-cmd --add-service=http --permanent
13. sudo firewall-cmd --reload
14. sudo cp /etc/drupal8/sites/default/default.settings.php /etc/drupal8/sites/default/settings.php
15. sudo chmod 666 /etc/drupal8/sites/default/settings.php
16. systemctl restart httpd

## Netwerkconfiguratie en troubleshooting

1. Welk(e) IP-adres(sen) heeft je VM? Vul onderstaande tabel aan (voeg rijen toe zoveel als nodig). Welk commando gebruikte je?

    ```
    $ COMMANDO ip address
    UITVOER
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b7:54:4c brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 80610sec preferred_lft 80610sec
    inet6 fe80::5c1c:7015:8db6:a61c/64 scope link
       valid_lft forever preferred_lft forever
    3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e4:f0:d4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.101/24 brd 192.168.56.255 scope global dynamic enp0s8
       valid_lft 1023sec preferred_lft 1023sec
    inet6 fe80::b980:865e:3238:6eda/64 scope link
       valid_lft forever preferred_lft forever
    4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:77:46:89 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
    5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:77:46:89 brd ff:ff:ff:ff:ff:ff

    ```

    | Interface | IP-adres | Netwerkmasker |
    | :---      | :---     | :---          |
    | enp0s3    | 10.0.2.15 | 10.0.2.255   |
    | enp0s8 | 192.168.56.101 | 192.168.56.255 |


2. Via welke router/default gateway kan je VM naar het Internet? Welk commando gebruikte je?

    ```
    $ COMMANDO ip r
    UITVOER
    default via 10.0.2.2 dev enp0s3 proto static metric 100
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
    192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.101 metric 100
    192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown

    ```

3. Wat is het IP-adres van `www.hogent.be`?

    ```
    $ COMMANDO nslookup www.hogent.be
    UITVOER
    Server:		195.130.131.2
    Address:	195.130.131.2#53

    Non-authoritative answer:
    Name:	www.hogent.be
    Address: 178.62.144.90
    ```

4. Om het IP-adres van websites en dergelijke op te kunnen vragen, moet je VM kunnen contact maken met een DNS-server. Hoe weet Linux welke DNS-server(s) beschikbaar is/zijn? Geef het configuratiebestand en de huidige inhoud ervan.

    ```
    $ COMMANDO cat /etc/resolv.conf
    UITVOER
    # Generated by NetworkManager
    search home
    nameserver 195.130.131.2
    nameserver 195.130.130.2
    ```

5. In welk(e) configuratiebestanden worden de instellingen van je netwerkinterfaces bijgehouden? Geef hieronder telkens ook de inhoud van deze bestanden:

    ```
    $ COMMANDO cat /etc/sysconfig/network-scripts/ifcfg-lo
    UITVOER
    DEVICE=lo
    IPADDR=127.0.0.1
    NETMASK=255.0.0.0
    NETWORK=127.0.0.0
    # If you're having problems with gated making 127.0.0.0/8 a martian,
    # you can change this to something else (255.255.255.255, for example)
    BROADCAST=127.255.255.255
    ONBOOT=yes
    NAME=loopback

    $ COMMANDO cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
    UITVOER
    HWADDR=08:00:27:B7:54:4C
    TYPE=Ethernet
    BOOTPROTO=dhcp
    DEFROUTE=yes
    PEERDNS=yes
    PEERROUTES=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=yes
    IPV6_AUTOCONF=yes
    IPV6_DEFROUTE=yes
    IPV6_PEERDNS=yes
    IPV6_PEERROUTES=yes
    IPV6_FAILURE_FATAL=no
    IPV6_ADDR_GEN_MODE=stable-privacy
    NAME=enp0s3
    UUID=5ef27c22-2601-3cbf-b3c0-113ee3592f49
    ONBOOT=yes
    AUTOCONNECT_PRIORITY=-999
    ```

6. Met welk commando test je of een host op het netwerk op dit moment online is? Probeer dit uit  vanop je VM met het IP-adres van je host-systeem en voeg de uitvoer hieronder in. Welk protocol uit de TCP/IP familie wordt door deze tool gebruikt?

    ```
    $ COMMANDO ping 10.0.2.15
    UITVOER
    PING 10.0.2.15 (10.0.2.15) 56(84) bytes of data.
    64 bytes from 10.0.2.15: icmp_seq=1 ttl=64 time=0.067 ms
    64 bytes from 10.0.2.15: icmp_seq=2 ttl=64 time=0.042 ms
    64 bytes from 10.0.2.15: icmp_seq=3 ttl=64 time=0.050 ms
    ^C
    --- 10.0.2.15 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2041ms
    rtt min/avg/max/mdev = 0.042/0.053/0.067/0.010 ms
    ```
    Maakt gebruik van de ICMP protocol.

7. Met het commando `ss -tln` kan je opvragen welke services er draaien op je systeem, ahv. de open netwerkpoorten. Leg uit wat de opties (`-tln`) betekenen. Probeer het commando uit op je VM wanneer Apache en MariaDB draaien en voeg de uitvoer hieronder in. Geef voor elke open poort beneden de 10.000 welke netwerkservice er mee geassocieerd is.

    ```
    $ COMMANDO
    UITVOER
    ```
    Vlag 't' staat voor het tonen TCP sockets.
    Vlag 'l' staat voor het tonen van luisterende sockets.
    Vlag 'n' staat voor het tonen van sockets nummer en niet hun service namen.

## Systeemlogbestanden

Om oorzaken te vinden van problemen op een Linux-systeem, maken systeembeheerders intensief gebruik van zgn. logbestanden. De meeste netwerkservices houden hun eigen logbestand(en), of maken gebruike van het algemene logsysteem dat op recente versies van Fedora `journald` heet.

1. In welke directory horen alle logbestanden volgens de Linux Filesystem Hierarcy Standard thuis?

  **Antwoord:**

2. Met welk commando kan je de algemene systeemlogs bekijken op een recente versie van Fedora?

    ```
    $ COMMANDO
    UITVOER
    ```

3. Met welk commando (incl. opties) kan je de logs blijven bekijken terwijl er nieuwe boodschappen toegevoegd worden? (Ter info: dit afsluiten kan met `<Ctrl-C>`

    ```
    $ COMMANDO
    UITVOER
    ```

4. Met welk commando kan je de logs voor een bepaalde netwerkservice bekijken?

    ```
    $ COMMANDO
    UITVOER
    ```

5. Open in één console het algemene logbestand en hou het open voor nieuwe boodschappen (zoals hierboven gevraagd). Open nu een andere console waar je de host-only netwerkinterface opnieuw opstart. Wat gebeurt er in de logs? Bekijk in het bijzonder de lijnen met `dhclient`.

    ```
    $ COMMANDO
    UITVOER
    ```

6. Wat is de naam van het logbestand waar je kan opvolgen welke webpagina's er opgevraagd worden aan je webserver? **Antwoord:** `BESTAND`
7. Open dit bestand met `tail -f` en laad een webpagina via een webbrowser. Wat gebeurt er in het logbestand?

    ```
    $ COMMANDO
    UITVOER
    ```

## Gebruikte bronnen

- https://www.tecmint.com/install-lamp-linux-apache-mysql-php-on-fedora-22/
- https://developer.fedoraproject.org/tech/database/mariadb/about.html
- https://fedoramagazine.org/howto-install-drupal-8-fedora/
- https://www.digitalocean.com/community/tutorials/how-to-reset-your-mysql-or-mariadb-root-password
- https://www.lifewire.com/uses-of-command-ping-2201076
