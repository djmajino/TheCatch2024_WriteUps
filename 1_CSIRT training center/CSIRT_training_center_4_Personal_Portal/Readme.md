# Zadanie #

> Hi, CSIRT trainee,
> 
> check if there is a leak of non-public data on the personal portal.
> 
- > The portal is accesible at [http://perso.cypherfix.tcc](http://perso.cypherfix.tcc "http://perso.cypherfix.tcc").
>
> See you in the next incident!
> 
> **Hint1:**
> Don't waste time trying to log in.
>  
> **Hint2:**
> Base64 is most popular encoding.


----------

# Riešenie #

Navštívime ponúkanú stránku, kde vidíme nejaký formulár, ale hint jasne hovorí, že nemáme márniť čas pokusom o prihlásenie. Máme tam Odkaz na Help, po kliknutí ktorého vidím v url 

	http://perso.cypherfix.tcc/files?file=help.html

a obsah stránky


**Directory listing for /**
- [info.txt](http://perso.cypherfix.tcc/info.txt "info.txt")
- [meeting2024-09-05.md](http://perso.cypherfix.tcc/meeting2024-09-05.md "meeting2024-09-05.md")
- [index.html](http://perso.cypherfix.tcc/index.html "index.html")
- [help.html](http://perso.cypherfix.tcc/help.html "help.html")

Keď poklikám na odkazy, zakaždým dostanem Not found chybu, ale keď názvy súborov zadám za file= do url address baru, tak pri meeting2024-09-05.md 

	http://perso.cypherfix.tcc/files?file=meeting2024-09-05.md

vidíme na konci súboru


	Signature code: 

	RkxBR3tZemZOLVo5UlAtb2MxUC1hdURvfQo=

Z hintu dva vieme, že ide o base64 string, ktorý dekódujeme do ascii a výsledok je **FLAG{YzfN-Z9RP-oc1P-auDo}**



----------

## Vlajka ##
    FLAG{YzfN-Z9RP-oc1P-auDo}