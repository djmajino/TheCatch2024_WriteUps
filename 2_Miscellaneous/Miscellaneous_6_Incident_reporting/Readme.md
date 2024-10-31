# Zadanie #

> Hi, TCC-CSIRT analyst,
> 
> our automatic incident recording system has captured about an hour of traffic originating from the IP range within the AI-CSIRT constituency. Analyze whether there are any incidents present and report all of them through the AI-CSIRT web interface.
> 
- > [Download the pcap file (cca 53 MB)](https://owncloud.cesnet.cz/index.php/s/OuJqS655dVCr2aw/download "Download the pcap file (cca 53 MB)")
> (sha256 checksum: **`e38550ba2cc32931d7c75856589a26e652f2f64e5be8cafaa34d5c191fc0fd1c`**)
- > The web interface is at http://incident-report.csirt.ai.tcc.
>
> See you in the next incident!
> 
> **Hint1:**
> The IP ranges in constituency of AI-CSIRT are **`10.99.224.0/24 and 2001:db8:7cc::a1:0/112.`**
> 
> **Hint2:**
> Based on our previous experience, the AI handling reports is very rigid and refuses to acknowledge incompletely described incidents. Make sure that you have described the incident accurately, with nothing missing or excessive.

----------

# Riešenie #

Pre vyriešenie úlohy je potrebné stiahnutý pcap súbor otvoriť vo Wiresharku, kde uvidíme všetky zachytené packety za určené obdobie a nájsť v ňom všetky výskyty nekalých úmyslov (incidentov) a zadať informácie incidentu na stránke uvedenej v úlohe. Na výber z typov incidentov máme
1. Brute force attack
1. Cryptomining
1. (D)DoS
1. Scanning
1. Sending phishing/spam
1. Web service enumeration
1. Other

Toto nám čiastočne zužuje okruh, na čo sa máme zhruba zamerať a rovnako vieme aj subnet ip adries "dobrákov".

Skúsime zadať filter 

**`ipv6.dst == 2001:db8:7cc::a1:0/112`**

ktorý nám zobrazí len packety prichádzajúce do siete a lepšie sa nám zorientuje v menšom množstve. Po krátkom skrolovaní som si všimol, že mnoho podozrivých requestov prichádza aj priamo z danej podsiete.

Nemusel som dlho skrolovať a už som videl obrovský výskyt HTTP requestov z odpoveďou 404, čo nám hneď napovedá, že ide o **`Web service enumeration`**. Skúsime sa zamerať nateraz len na HTTP packety s kódom odpovede 404 a do filtra zadáme **http**, kde krásne vidíme všetky pokusy o enumeráciu (okrem iného, ale teraz nás zaujíme len tento incident). Vidím, že pochádzaju z ip **`2001:db8:7cc::a1:210`**, tak upresním filter 

**`http &&  ipv6.src==2001:db8:7cc::a1:210`**

a všimnem si, že enumerácia trvala od **`08:48:17`** do **`08:49:44`**, bol tam ešte ďalší packet trochu neskôr, ale to už bol request na endpoint s odpoveďou 200/OK. Počet packetov tohto incidentu bolo **46141**, do stránky teda zadáme nasledovné  

    Incident type: Web service enumeration
    Offending IP address: 2001:db8:7cc::a1:210
    Target IP address: 2001:db8:7cc::acdc:24:a160
    Datetime of the first attempt (UTC): 2024-09-26 08:48:17
    Datetime of the last attempt (UTC): 2024-09-26 08:49:44
    Number of enumerated URL: 10000-49999 (bolo ich 46141)

Dostali sme odpoveď:

    AI response:
    {"message":"It's probably a real incident, I'll consult the natural intelligence of a member of the CSIRT."}
    
    Your incident ID is NC80OiBkNWtNfQ==, please keep it.

Incident ID v Base64 je po dekódovaní **`4/4: d5kM}`**, máme teda časť vlajky 4/4.


Ostaneme pri tomto filtri, pretože vidíme, že nasledne tam bolo mnoho požiadaviek typu **`POST`** na endpoint **`http://[2001:db8:7cc:0:0:acdc:24:beef]/login`**, čo nám naznačuje, že pôjde o ďalší incident typu **`Brute force attack`**. Cieľová IP sa líši, tak odfiltrujeme packety trochu viac filtrom 

**`http.request.method == POST &&  ipv6.src==2001:db8:7cc::a1:210 && ipv6.dst == 2001:db8:7cc:0:0:acdc:24:beef`** 

kde vidíme už len 1225 pokusov, z toho packet číslo 225910 naznačuje, že brute force útok bol úspešný. Zadáme do stránky nasledovné

    Incident type: Brute force attack
    Offending IP address: 2001:db8:7cc::a1:210
    Target IP address: 2001:db8:7cc:0:0:acdc:24:beef
    Datetime of the first attempt (UTC): 2024-09-26 08:55:19
    Datetime of the last attempt (UTC): 2024-09-26 08:55:42
	Number of attempts: 1000-4999
    Result: Success

Dostali sme odpoveď:

    AI response:
    {"message":"It's probably a real incident, I'll consult the natural intelligence of a member of the CSIRT."}
    
    Your incident ID is MS80OiBGTEFHe2xFOA==, please keep it.

Incident ID v Base64 je po dekódovaní **`1/4: FLAG{lE8`**, máme teda časť vlajky 1/4.

Pri http ešte ostaneme, v prvom filtri, kde aplikujem aj filter http 

**`http && ipv6.dst == 2001:db8:7cc::a1:0/112`**

vidíme aj veľký počet requestov veľmi rýchlo po sebe nasledujúcich na endpoint **`http://subsidies.tcc:80/check_status.php`**, na ip **`2001:db8:7cc::acdc:24:911`** kde endpoint zjavne musí niečo prežuť a následne pošle výstup, čo výrazne zahltí daný endpoint. Tu je teda zjavné, že ide o (D)DoS útok. Zadáme teda

    Incident type: (D)DoS
    Offending IP address: 2001:db8:7cc::a1:d055
    Target IP address: 2001:db8:7cc::acdc:24:911
    Datetime of the first attempt (UTC): 2024-09-26 08:58:48
    Datetime of the last attempt (UTC): 2024-09-26 09:46:44
	Service affected: HTTP

Dostali sme

    AI response:
    {"message":"It's probably a real incident, I'll consult the natural intelligence of a member of the CSIRT."}
    
    Your incident ID is Mi80OiBzLVVrb3g=, please keep it.

Incident ID v Base64 je po dekódovaní **`2/4: s-Ukox`**, máme teda časť vlajky 2/4.

Pre hľadanie posledného incidentu bolo potrebné pozorné oko, všimol som si však, že ip `2001:db8:7cc::a1:42` sa snaží skenovať adresy na sieti `ff02::1:ff24:0/116`, a dostáva odpoveď 

	Neighbor Solicitation for 2001:db8:7cc::acdc:24:fd61 from 02:42:ac:11:02:20

alebo

    Neighbor Advertisement 2001:db8:7cc::acdc:24:2848 (sol, ovr) is at 02:42:ac:f2:db:74

ďalej je vidieť, že sa okrem iného snaží na sieti `2001:db8:7cc::acdc:24:0/112` skenovať porty `21,22,53,80,443,8080` a celkové trvanie skenov podľa filtra **`ipv6.addr == 2001:db8:7cc::a1:42`** trvalo  od `2024-09-26 08:44:28` do `2024-09-26 09:17:08` . Keď sa sa zameráme na Neighbor Advertisement packety, čo če je icmpv6.type == 136, pomôžeme si teda filtrom 

**`ipv6.dst  == 2001:db8:7cc::a1:42 && icmpv6.type == 136`**

vidíme, že útočník objavil 262 odpovedajúcich endpointov, následne pri skenovaní port objavil 25 endpointov.

Zadáme

	Incident type: Scanning
    Offending IP address: 2001:db8:7cc::a1:42
    Target IP address: 2001:db8:7cc::acdc:24:0/112
    Datetime of the first attempt (UTC): 2024-09-26 08:44:28
    Datetime of the last attempt (UTC): 2024-09-26 09:17:08
	Target ports: 21, 22, 53, 80, 443, 8080
	Number of found and scanned targets: 20-99

a dostaneme odpoveď
    
    AI response:
    {"message":"It's probably a real incident, I'll consult the natural intelligence of a member of the CSIRT."}
    
    Your incident ID is My80OiAtYTBRZi0=, please keep it.

Incident ID v Base64 je po dekódovaní **`3/4: -a0Qf-`**, máme teda časť vlajky 3/4, čím po poskladaní dostaneme **FLAG{lE8s-Ukox-a0Qf-d5kM}**



----------

## Vlajka ##
    FLAG{lE8s-Ukox-a0Qf-d5kM}