# Zadanie #

> Hi, CSIRT trainee,
> 
> nobody can work secure from home without VPN. You have to install and configure OpenVPN properly. Configuration file can be downloaded from CTFd's link [VPN](https://www.thecatch.cz/vpn "VPN"). Your task is to activate VPN, visit the testing pages, and get your code proving your IPv4 and IPv6 readiness.
> 
> 
> - IPv4 testing page is available at [http://bellatrix.cypherfix.tcc](http://bellatrix.cypherfix.tcc "http://bellatrix.cypherfix.tcc").
> 
> - IPv6 testing page is available at [http://gerry.cypherfix.tcc](http://gerry.cypherfix.tcc "http://gerry.cypherfix.tcc").
> 
> See you in the next incident!
> 
> **Hint1:**
> https://openvpn.net/community-resources/reference-manual-for-openvpn-2-4/
> 
> **Hint2:**
> Do not run more different VPNs at once.

----------

# Riešenie #

Stačí stiahnuť VPPN profil a buď ho vo windowse naimportovať do OpenVpn klienta alebo v Kali pripojiť pomocou príkazu v konzole 

    sudo ovpn --config ctfd_ovpn.ovpv --daemon


Následne otvoriť oba URL odkazy, skopírovať časti vlajky, spojiť dokopy a hotovo!

### Bellatrix (IPv4 cat) ###
Meow there, my half of the verification code is **FLAG{Hgku-58OA**, purrhaps you send me a yummy tuna can, meow?

### Gerry (IPv6 cat) ###
Purr, my part of the code is **-Hsrn-03Zr}**, could you paw-lease send me a scrumptious chicken treat, meow?

----------

## Vlajka ##
    FLAG{Hgku-58OA-Hsrn-03Zr}