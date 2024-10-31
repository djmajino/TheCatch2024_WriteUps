# Zadanie #

> Hi, TCC-CSIRT analyst,
> 
> rumors are spreading in the ticketing system that an old, unmaintained service is running somewhere in the network, which could be a security risk (even though it's called **`3S - Super Secure Service`**). Old-timers claim that it had the domain name **`supersecureservice.cypherfix.tcc`**. This name does not exist in the current DNS, but some information might still be available on the DNS server **`ns6-old.tcc`**, which will be shut down soon.
> 
> Explore the service and gather as much information as possible about 3S.
> 
> See you in the next incident!
> 
> **Hint1:**
> Any resource records can be useful.

----------

# Riešenie #

Tu nám pomôže príkaz dig v kali

    
    > dig supersecureservice.cypherfix.tcc @ns6-old.tcc any
    
    ; <<>> DiG 9.20.2-1-Debian <<>> supersecureservice.cypherfix.tcc @ns6-old.tcc any
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29417
    ;; flags: qr aa rd; QUERY: 1, ANSWER: 7, AUTHORITY: 0, ADDITIONAL: 1
    ;; WARNING: recursion requested but not available
    
    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1232
    ; COOKIE: 5c99cc92dddc1e7f010000006721418a8cc9d7684d9ae579 (good)
    ;; QUESTION SECTION:
    ;supersecureservice.cypherfix.tcc. IN   ANY
    
    ;; ANSWER SECTION:
    supersecureservice.cypherfix.tcc. 86400 IN TXT  "Super secure service in testing mode, any records are hipsters friendly!"
    supersecureservice.cypherfix.tcc. 86400 IN HINFO "TCC 686" "TCC-OS 20.20"
    supersecureservice.cypherfix.tcc. 86400 IN SVCB 1 web3s-746865636174636832303234.cypherfix.tcc. alpn="h2,h3,mandatory=alpn" port=8020
    supersecureservice.cypherfix.tcc. 86400 IN SVCB 4 web3s-7468656361746368323032343.cypherfix.tcc. alpn="h2,h3,mandatory=alpn" port=8020
    supersecureservice.cypherfix.tcc. 86400 IN SVCB 2 web3s-7468656361746368323032342.cypherfix.tcc. alpn="h2,h3,mandatory=alpn" port=8020
    supersecureservice.cypherfix.tcc. 86400 IN A10.99.24.21
    supersecureservice.cypherfix.tcc. 86400 IN AAAA 2001:db8:7cc::24:21
    
    ;; Query time: 31 msec
    ;; SERVER: 2001:db8:7cc::24:20#53(ns6-old.tcc) (TCP)
    ;; WHEN: Tue Oct 29 16:11:54 EDT 2024
    ;; MSG SIZE  rcvd: 526

Tu vidíme nejakú službu web3s-746865636174636832303234.cypherfix.tcc bežiacu na porte 8020, pozrieme na ňu 
**http://web3s-746865636174636832303234.cypherfix.tcc:8020**

![](http://majino.sk/thecatch2024writeup/oldservice001.png)


----------

## Vlajka ##
    FLAG{yNx6-tH9y-hKtB-20k6}