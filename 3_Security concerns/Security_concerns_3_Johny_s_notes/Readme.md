# Zadanie #

> Hi, TCC-CSIRT analyst,
> 
> admin Johny is testing a new notebook to take notes, as any good administrator would. He thinks he has correctly secured the application so that only he can access it and no one else. Your task is to check if he made any security lapses.
> 
- > Admin Johny uses workstation `johny-station.cypherfix.tcc`.
- > Application for notes runs on `notes.cypherfix.tcc`.
>
> See you in the next incident!
> 
> **Hint1:**
> 
> 
> **Hint2:**
> `/usr/sbin/nologin` is a good servant but a bad master.

----------

# Riešenie #

Tak ako v iných úlohách, urobíme fuzzing (enumeráciu) a preskenujeme porty.

Na `johny-station.cypherfix.tcc` má otvorené porty

    22/tcp open  ssh	OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
    80/tcp open  http	Apache httpd 2.4.62 ((Debian))

Na `notes.cypherfix.tcc`

	80/tcp   open  http   Apache httpd 2.4.62 ((Debian))
	8080/tcp open  tcpwrapped
	8081/tcp open  tcpwrapped

Po pozretí stránok je na `johny-station.cypherfix.tcc` iba default Apache stránka a na `notes.cypherfix.tcc` stránka, ktorá ma automaticky presmeruje na port 8080

```
┌──(kali㉿kali)-[~]
└─$ curl notes.cypherfix.tcc          
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Redirect Example</title>
    <script type="text/javascript">
        // JavaScript function to redirect to another URL with a different port
        function redirectToNewPort() {
            // Extract current location details
            var currentHost = window.location.hostname; // Get current host
            var newPort = 8080; // Define new port

            // Construct new URL with the new port
            var newUrl = window.location.protocol + "//" + currentHost + ":" + newPort + window.location.pathname;

            // Redirect to the new URL
            window.location.href = newUrl;
        }

        // Call the function to perform the redirection
        window.onload = redirectToNewPort;
    </script>
</head>
<body>
    <h1>Redirecting...</h1>
    <p>If you are not redirected automatically, <a href="javascript:redirectToNewPort()">click here</a>.</p>
</body>
</html>
                                                                                                                                                                                         
┌──(kali㉿kali)-[~]
└─$ curl notes.cypherfix.tcc:8080
curl: (52) Empty reply from server
                                                                                                                                                                                         
┌──(kali㉿kali)-[~]
└─$ curl notes.cypherfix.tcc:8081
curl: (52) Empty reply from server
```

Heslo na johnyho ssh síce nemáme, ale podľa hintu je zrejmé, že sa budeme motkať niekde aj týmto smerom.

Trochu fuzzingu s pridaním aj prefixu ~ za lomítkom sa mi objavil index priečinka `http://johny-station.cypherfix.tcc/~johny/`, kde vidíme, že sa nachádza priečinok flatnotes. Trochu sa tu pošprtáme a vidíme tu aj napríklad Dockerfile, kde vidíme, že flatnotes nám bežia na porte 8080.

Trochu pokusov a omylov a objavit som git repozitár na 

	http://johny-station.cypherfix.tcc/~johny/flatnotes/.git/

stiahol som obsah, tentokrát som bol na windowse, tak som použil github desktop appku, načítal repozitár a hneď vidím nepushnutý commit s heslom na johnyho, **`gojohnygo`**

        -e "PGID=1000" \
        -e "FLATNOTES_AUTH_TYPE=password" \
        -e "FLATNOTES_USERNAME=user" \
      - -e "FLATNOTES_PASSWORD=gojohnygo" \
      + -e "FLATNOTES_PASSWORD=changeMe!" \
        -e "FLATNOTES_SECRET_KEY=aLongRandomSeriesOfCharacters" \
        -v "$(pwd)/data:/data" \
        -p "8080:8080" \

keď sa pokuúsim prihlásiť na ssh, tak neúspešne. Teda heslo zoberie, ale tu sme narazili na spomínaný /usr/sbin/nologin..

Znovu som si prečítal zadanie a všimol si časť 

**He thinks he has correctly secured the application so that only he can access it and no one else.** 

Takže zrejme sa dostaneme na notes iba ak by sme tam šli z johny-station a to defacto môžeme. Máme predsa login a password na ssh a príkaz zadať žiadny nemusíme, stačí, že si vytvoríme tunel.

Opäť som na kali, v jednom terminál okne zadám príkaz nižšie, zadám heslo a tunel je vytvorený

    ssh -N -D 8888 johny@johny-station.cypherfix.tcc
    
Otvoríme Firefox, nastavíme proxy, do SOCKS host dáme localhost, port dáme 8888, zvolíme SOCKS v5 a otvoríme notes stránku
Tá nás pekne presmeruje na port 8080 a zároveň vidím login stránku

    http://notes.cypherfix.tcc:8080/login?redirect=/

Z git repo vieme že user je user a password je gojohnygo, prihlásenie funguje, vidím poznámky, medzi nimi jedna s názvom flag, otvorím a tam  FLAG{VfCK-Hlp4-cQl8-p0UM}


----------

## Vlajka ##
    FLAG{VfCK-Hlp4-cQl8-p0UM}