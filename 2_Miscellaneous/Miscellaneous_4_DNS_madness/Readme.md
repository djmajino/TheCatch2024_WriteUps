# Zadanie #

> Hi, TCC-CSIRT analyst,
> 
> for infrastructure management, the Department of Operations must maintain a large number of DNS records in many zones. The DNS administrator has gone crazy from the sheer number of records, sitting in front of the monitor, swaying back and forth, and repeatedly saying "It came to me, my own, my love, my own, my precious **`infra.tcc`**". Your task is to examine the zones, resource records, and their counts so that we can hire a sufficiently resilient administrator next time.
> 
> See you in the next incident!
> 
> **Hint1:**
> On the administrator's desk lies a well-worn copy of RFC (number of RFC is unreadable due to a huge coffee stain).
> 
> **Hint2:**
> In the past weeks, the administrator has been passionately talking about a new method for provisioning secondary servers in which a list of zones to be served is stored in a DNS zone and can be propagated to slaves via AXFR/IXFR.

----------

# Riešenie #

Pozrieme sa opať na dns cez príkaz dig any infra.tcc

    dig any infra.tcc
    
    ; <<>> DiG 9.20.2-1-Debian <<>> any infra.tcc
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11546
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 5
    
    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: 57137e9c0a91cda001000000672143aa9cc21e8e5849cdb3 (good)
    ;; QUESTION SECTION:
    ;infra.tcc. IN  ANY
    
    ;; ANSWER SECTION:
    infra.tcc.  62294   IN  SOA jessiejames.infra.tcc. hostmaster.infra.tcc. 2024100701 604800 86400 2419200 86400
    infra.tcc.  50216   IN  NS  jameson.infra.tcc.
    infra.tcc.  50216   IN  NS  jamison.infra.tcc.
    
    ;; ADDITIONAL SECTION:
    jameson.infra.tcc.  31866   IN  A   10.99.24.29
    jamison.infra.tcc.  31791   IN  A   10.99.24.28
    jameson.infra.tcc.  31866   IN  AAAA2001:db8:7cc::24:29
    jamison.infra.tcc.  31791   IN  AAAA2001:db8:7cc::24:28
    
    ;; Query time: 31 msec
    ;; SERVER: 192.168.136.2#53(192.168.136.2) (TCP)
    ;; WHEN: Tue Oct 29 16:20:57 EDT 2024
    ;; MSG SIZE  rcvd: 257

Máme tu záznam SOA - jessiejames.infra.tcc

použijeme AXFR 

    dig axfr infra.tcc @jessiejames.infra.tcc
    
    ; <<>> DiG 9.20.2-1-Debian <<>> axfr infra.tcc @jessiejames.infra.tcc
    ;; global options: +cmd
    infra.tcc.  86400   IN  SOA jessiejames.infra.tcc. hostmaster.infra.tcc. 2024100701 604800 86400 2419200 86400
    infra.tcc.  86400   IN  NS  jameson.infra.tcc.
    infra.tcc.  86400   IN  NS  jamison.infra.tcc.
    jameson.infra.tcc.  86400   IN  AAAA2001:db8:7cc::24:29
    jameson.infra.tcc.  86400   IN  A   10.99.24.29
    jamison.infra.tcc.  86400   IN  AAAA2001:db8:7cc::24:28
    jamison.infra.tcc.  86400   IN  A   10.99.24.28
    jessiejames.infra.tcc.  86400   IN  AAAA2001:db8:7cc:0:6361:74:24:27
    infra.tcc.  86400   IN  SOA jessiejames.infra.tcc. hostmaster.infra.tcc. 2024100701 604800 86400 2419200 86400
    ;; Query time: 20 msec
    ;; SERVER: 2001:db8:7cc:0:6361:74:24:27#53(jessiejames.infra.tcc) (TCP)
    ;; WHEN: Tue Oct 29 16:22:42 EDT 2024
    ;; XFR size: 9 records (messages 1, bytes 321)

Máme ip adresu **`jessiejames.infra.tcc`** - **`2001:db8:7cc:0:6361:74:24:27`** , ktorá obsahuje podorivo známy rozhas printable ascii znakov 63 61 74 24 27 - po dekódovaní cat$'. Môže to evokovať mačky, prípadne nejaký príkaz na zobrazene obsahu súboru, ale v DNS žargóne to evokuje catalog.  Skúsil som pár výrazov, nakoniec výraz catalog bol správny.

    dig axfr catalog.infra.tcc @jessiejames.infra.tcc
    
    ; <<>> DiG 9.20.2-1-Debian <<>> axfr catalog.infra.tcc @jessiejames.infra.tcc
    ;; global options: +cmd
    catalog.infra.tcc.  86400   IN  SOA . . 2024100802 604800 86400 2419200 86400
    catalog.infra.tcc.  86400   IN  NS  invalid.
    version.catalog.infra.tcc. 86400 IN TXT "2"
    aldoriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR aldoria.infra.tcc.
    aldorninfratcc.zones.catalog.infra.tcc. 3600 IN PTR aldorn.infra.tcc.
    arionisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR arionis.infra.tcc.
    averoninfratcc.zones.catalog.infra.tcc. 3600 IN PTR averon.infra.tcc.
    banneroninfratcc.zones.catalog.infra.tcc. 3600 IN PTR banneron.infra.tcc.
    beloriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR beloria.infra.tcc.
    blaveninfratcc.zones.catalog.infra.tcc. 3600 IN PTR blaven.infra.tcc.
    branoxinfratcc.zones.catalog.infra.tcc. 3600 IN PTR branox.infra.tcc.
    brinleyinfratcc.zones.catalog.infra.tcc. 3600 IN PTR brinley.infra.tcc.
    brionisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR brionis.infra.tcc.
    brystoninfratcc.zones.catalog.infra.tcc. 3600 IN PTR bryston.infra.tcc.
    calianinfratcc.zones.catalog.infra.tcc. 3600 IN PTR calian.infra.tcc.
    calindrainfratcc.zones.catalog.infra.tcc. 3600 IN PTR calindra.infra.tcc.
    celoriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR celoria.infra.tcc.
    corviniainfratcc.zones.catalog.infra.tcc. 3600 IN PTR corvinia.infra.tcc.
    crystarinfratcc.zones.catalog.infra.tcc. 3600 IN PTR crystar.infra.tcc.
    delwyninfratcc.zones.catalog.infra.tcc. 3600 IN PTR delwyn.infra.tcc.
    draxorinfratcc.zones.catalog.infra.tcc. 3600 IN PTR draxor.infra.tcc.
    draymoorinfratcc.zones.catalog.infra.tcc. 3600 IN PTR draymoor.infra.tcc.
    drevininfratcc.zones.catalog.infra.tcc. 3600 IN PTR drevin.infra.tcc.
    elarisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR elaris.infra.tcc.
    eldarainfratcc.zones.catalog.infra.tcc. 3600 IN PTR eldara.infra.tcc.
    eldoriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR eldoria.infra.tcc.
    eldricinfratcc.zones.catalog.infra.tcc. 3600 IN PTR eldric.infra.tcc.
    eldrininfratcc.zones.catalog.infra.tcc. 3600 IN PTR eldrin.infra.tcc.
    elvoriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR elvoria.infra.tcc.
    falwyninfratcc.zones.catalog.infra.tcc. 3600 IN PTR falwyn.infra.tcc.
    fayleninfratcc.zones.catalog.infra.tcc. 3600 IN PTR faylen.infra.tcc.
    fendarainfratcc.zones.catalog.infra.tcc. 3600 IN PTR fendara.infra.tcc.
    finstarinfratcc.zones.catalog.infra.tcc. 3600 IN PTR finstar.infra.tcc.
    galdoriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR galdoria.infra.tcc.
    galdorinfratcc.zones.catalog.infra.tcc. 3600 IN PTR galdor.infra.tcc.
    glenwoodiainfratcc.zones.catalog.infra.tcc. 3600 IN PTR glenwoodia.infra.tcc.
    gorathinfratcc.zones.catalog.infra.tcc. 3600 IN PTR gorath.infra.tcc.
    haldorainfratcc.zones.catalog.infra.tcc. 3600 IN PTR haldora.infra.tcc.
    haldrixinfratcc.zones.catalog.infra.tcc. 3600 IN PTR haldrix.infra.tcc.
    hesperainfratcc.zones.catalog.infra.tcc. 3600 IN PTR hespera.infra.tcc.
    hexoriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR hexoria.infra.tcc.
    illyriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR illyria.infra.tcc.
    itharainfratcc.zones.catalog.infra.tcc. 3600 IN PTR ithara.infra.tcc.
    jadroninfratcc.zones.catalog.infra.tcc. 3600 IN PTR jadron.infra.tcc.
    jorathiainfratcc.zones.catalog.infra.tcc. 3600 IN PTR jorathia.infra.tcc.
    jorathinfratcc.zones.catalog.infra.tcc. 3600 IN PTR jorath.infra.tcc.
    kandorinfratcc.zones.catalog.infra.tcc. 3600 IN PTR kandor.infra.tcc.
    karnisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR karnis.infra.tcc.
    kylorainfratcc.zones.catalog.infra.tcc. 3600 IN PTR kylora.infra.tcc.
    kynerainfratcc.zones.catalog.infra.tcc. 3600 IN PTR kynera.infra.tcc.
    loriahinfratcc.zones.catalog.infra.tcc. 3600 IN PTR loriah.infra.tcc.
    lunarainfratcc.zones.catalog.infra.tcc. 3600 IN PTR lunara.infra.tcc.
    lyranthinfratcc.zones.catalog.infra.tcc. 3600 IN PTR lyranth.infra.tcc.
    lyrisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR lyris.infra.tcc.
    lytherainfratcc.zones.catalog.infra.tcc. 3600 IN PTR lythera.infra.tcc.
    mardaleinfratcc.zones.catalog.infra.tcc. 3600 IN PTR mardale.infra.tcc.
    mavrosinfratcc.zones.catalog.infra.tcc. 3600 IN PTR mavros.infra.tcc.
    mirelandinfratcc.zones.catalog.infra.tcc. 3600 IN PTR mireland.infra.tcc.
    mithrainfratcc.zones.catalog.infra.tcc. 3600 IN PTR mithra.infra.tcc.
    mythrainfratcc.zones.catalog.infra.tcc. 3600 IN PTR mythra.infra.tcc.
    nexorinfratcc.zones.catalog.infra.tcc. 3600 IN PTR nexor.infra.tcc.
    nolarainfratcc.zones.catalog.infra.tcc. 3600 IN PTR nolara.infra.tcc.
    nolviainfratcc.zones.catalog.infra.tcc. 3600 IN PTR nolvia.infra.tcc.
    nyrosinfratcc.zones.catalog.infra.tcc. 3600 IN PTR nyros.infra.tcc.
    odrisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR odris.infra.tcc.
    orinthiainfratcc.zones.catalog.infra.tcc. 3600 IN PTR orinthia.infra.tcc.
    ostarainfratcc.zones.catalog.infra.tcc. 3600 IN PTR ostara.infra.tcc.
    phaelorinfratcc.zones.catalog.infra.tcc. 3600 IN PTR phaelor.infra.tcc.
    prydiainfratcc.zones.catalog.infra.tcc. 3600 IN PTR prydia.infra.tcc.
    quintisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR quintis.infra.tcc.
    quorixinfratcc.zones.catalog.infra.tcc. 3600 IN PTR quorix.infra.tcc.
    ravenorinfratcc.zones.catalog.infra.tcc. 3600 IN PTR ravenor.infra.tcc.
    rivoriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR rivoria.infra.tcc.
    rovilleinfratcc.zones.catalog.infra.tcc. 3600 IN PTR roville.infra.tcc.
    rynisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR rynis.infra.tcc.
    serethinfratcc.zones.catalog.infra.tcc. 3600 IN PTR sereth.infra.tcc.
    silarisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR silaris.infra.tcc.
    sylviainfratcc.zones.catalog.infra.tcc. 3600 IN PTR sylvia.infra.tcc.
    talwyninfratcc.zones.catalog.infra.tcc. 3600 IN PTR talwyn.infra.tcc.
    tarnixinfratcc.zones.catalog.infra.tcc. 3600 IN PTR tarnix.infra.tcc.
    thalorinfratcc.zones.catalog.infra.tcc. 3600 IN PTR thalor.infra.tcc.
    thyrainfratcc.zones.catalog.infra.tcc. 3600 IN PTR thyra.infra.tcc.
    torvalinfratcc.zones.catalog.infra.tcc. 3600 IN PTR torval.infra.tcc.
    triloreinfratcc.zones.catalog.infra.tcc. 3600 IN PTR trilore.infra.tcc.
    uloriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR uloria.infra.tcc.
    ulyriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR ulyria.infra.tcc.
    valoriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR valoria.infra.tcc.
    vandorinfratcc.zones.catalog.infra.tcc. 3600 IN PTR vandor.infra.tcc.
    vandriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR vandria.infra.tcc.
    vardisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR vardis.infra.tcc.
    vermisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR vermis.infra.tcc.
    vesperainfratcc.zones.catalog.infra.tcc. 3600 IN PTR vespera.infra.tcc.
    vesperisinfratcc.zones.catalog.infra.tcc. 3600 IN PTR vesperis.infra.tcc.
    vionixinfratcc.zones.catalog.infra.tcc. 3600 IN PTR vionix.infra.tcc.
    wyncrestinfratcc.zones.catalog.infra.tcc. 3600 IN PTR wyncrest.infra.tcc.
    wyrmarinfratcc.zones.catalog.infra.tcc. 3600 IN PTR wyrmar.infra.tcc.
    xalithinfratcc.zones.catalog.infra.tcc. 3600 IN PTR xalith.infra.tcc.
    xarioninfratcc.zones.catalog.infra.tcc. 3600 IN PTR xarion.infra.tcc.
    yaraelinfratcc.zones.catalog.infra.tcc. 3600 IN PTR yarael.infra.tcc.
    yloriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR yloria.infra.tcc.
    zarvixinfratcc.zones.catalog.infra.tcc. 3600 IN PTR zarvix.infra.tcc.
    zeloriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR zeloria.infra.tcc.
    zephyriainfratcc.zones.catalog.infra.tcc. 3600 IN PTR zephyria.infra.tcc.
    zylariainfratcc.zones.catalog.infra.tcc. 3600 IN PTR zylaria.infra.tcc.
    catalog.infra.tcc.  86400   IN  SOA . . 2024100802 604800 86400 2419200 86400
    ;; Query time: 20 msec
    ;; SERVER: 2001:db8:7cc:0:6361:74:24:27#53(jessiejames.infra.tcc) (TCP)
    ;; WHEN: Tue Oct 29 16:26:38 EDT 2024
    ;; XFR size: 105 records (messages 1, bytes 3948)
    
tu nám zobrazilo toho mrtě, nie som blázon aby som skúšal manuálne jedno po jednom, na to je vhodné napísať si skriptík, ale koniec koncov, v jednom by mala byť vlajka, tak zopakujeme opať dix axfr

    dig @2001:db8:7cc:0:6361:74:24:27 banneron.infra.tcc AXFR 
    
    
    ; <<>> DiG 9.20.2-1-Debian <<>> @2001:db8:7cc:0:6361:74:24:27 banneron.infra.tcc AXFR
    ; (1 server found)
    ;; global options: +cmd
    banneron.infra.tcc. 86400   IN  SOA jessiejames.infra.tcc. hostmaster.infra.tcc. 2024100701 604800 86400 2419200 86400
    banneron.infra.tcc. 86400   IN  NS  jameson.infra.tcc.
    banneron.infra.tcc. 86400   IN  NS  jamison.infra.tcc.
    40ae12928dbf450106d8097a7ec875ea.banneron.infra.tcc. 86400 IN AAAA 2001:db8:7cd::24:6577
    40ae12928dbf450106d8097a7ec875ea.banneron.infra.tcc. 86400 IN TXT "RkxBR3tBdlhPLWlNazctM2JvSC1pWURwfQ=="
    banneron.infra.tcc. 86400   IN  SOA jessiejames.infra.tcc. hostmaster.infra.tcc. 2024100701 604800 86400 2419200 86400
    ;; Query time: 48 msec
    ;; SERVER: 2001:db8:7cc:0:6361:74:24:27#53(2001:db8:7cc:0:6361:74:24:27) (TCP)
    ;; WHEN: Tue Oct 29 16:38:15 EDT 2024
    ;; XFR size: 6 records (messages 1, bytes 324)
 
```
import dns.query
import dns.zone
import dns.resolver

ns_ip = '2001:db8:7cc:0:6361:74:24:27'

member_zones = [
    "aldoria", "aldorn", "arionis", "averon", "banneron", "beloria",
    "blaven", "branox", "brinley", "brionis", "bryston", "calian",
    "calindra", "celoria", "corvinia", "crystar", "delwyn", "draxor",
    "draymoor", "drevin", "elaris", "eldara", "eldoria", "eldric",
    "eldrin", "elvoria", "falwyn", "faylen", "fendara", "finstar",
    "galdoria", "galdor", "glenwoodia", "gorath", "haldora", "haldrix",
    "hespera", "hexoria", "illyria", "ithara", "jadron", "jorathia",
    "jorath", "kandor", "karnis", "kylora", "kynera", "loriah",
    "lunara", "lyranth", "lyris", "lythera", "mardale", "mavros",
    "mireland", "mithra", "mythra", "nexor", "nolara", "nolvia",
    "nyros", "odris", "orinthia", "ostara", "phaelor", "prydia",
    "quintis", "quorix", "ravenor", "rivoria", "roville", "rynis",
    "sereth", "silaris", "sylvia", "talwyn", "tarnix", "thalor",
    "thyra", "torval", "trilore", "uloria", "ulyria", "valoria",
    "vandor", "vandria", "vardis", "vermis", "vespera", "vesperis",
    "vionix", "wyncrest", "wyrmar", "xalith", "xarion", "yarael",
    "yloria", "zarvix", "zeloria", "zephyria", "zylaria"
]

for zone_name in member_zones:
    zone_name = zone_name + ".infra.tcc"
    try:
        xfr = dns.query.xfr(ns_ip, zone_name)
        zone = dns.zone.from_xfr(xfr)

        # popozeráme sa na TXT záznamy
        for name, node in zone.nodes.items():
            for rdataset in node.rdatasets:
                if rdataset.rdtype == dns.rdatatype.TXT:
                    for rdata in rdataset:
                        txt_data = rdata.strings[0].decode()
                        
                        if "FLAG{" in txt_data or "flag{" in txt_data or "RkxBR" in txt_data:
                            print(f"Vlajocka najdena na {zone_name}: {txt_data}\n")
                            
                            break
    except Exception as e:
        print(f"Zone transfer neuspesny pre {zone_name}: {e}\n")
```

Pri banneron.infra.tcc vidíme TXT záznam s base64 stringom **`RkxBR3tBdlhPLWlNazctM2JvSC1pWURwfQ==`** , po dekódovaní **`FLAG{AvXO-iMk7-3boH-iYDp}`**

----------

## Vlajka ##
    FLAG{AvXO-iMk7-3boH-iYDp}