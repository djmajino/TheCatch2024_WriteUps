# Zadanie #

> Hi, CSIRT trainee,
> 
> we have received a request for cooperation issued by law enforcement authorities to locate a device on our network **`2001:db8:7cc::/64`**. Please try to accommodate the requester as much as possible.
> 
- > [Download the request of cooperation](https://owncloud.cesnet.cz/index.php/s/I7qNEzYdU8yUzTt/download "Download the request of cooperation")
> (sha256 checksum: 
> `67bd096145a74feb4f1ffad849721547ac4c29de52f525282a5e01a978109878`)
>
> See you in the next incident!
> 
> **Hint1:**
> Find out if there is a way to create an IPv6 address from a MAC address.


----------

# Riešenie #

Stiahnuť, rozbaliť archív, otvoriť pdf.

> **Subject: 20/2024/TCC - Request for Cooperation in Locating a Device**
> Dear TCC-CSIRT,
> we are writing to formally request your cooperation in a matter involving the identification and
> location of a specific device on your network.
> **Details of the Request:**
> - **Network Range:** `2001:db8:7cc::/64`
> - **Device media access control address:** `00ca.7ad0.ea71`
> - **Purpose:** To locate a device of interest which is believed to be operating within the
> aforementioned network range.
> 
> This request is issued in connection with an ongoing investigation and is intended to assist law
> enforcement authorities in resolving this matter effectively. Your prompt and thorough cooperation
> in this regard is highly appreciated.
> **Action Required: **
> • Please conduct a search within the specified network range to identify and locate the device
> in question.
> • Utilize the knowledge, that the device uses stateless autoconfiguration to connect to the IPv6
> networks, and it is secured by MoggiePounce software, i.e. the device provides a webpage
> with statistical and localization data on port 1701/TCP.
> • Provide us with any relevant information or findings that may assist in the investigation.
> We understand the sensitivity of this request and assure you that any information provided will be
> handled with the utmost confidentiality and in accordance with legal protocols.
> Thank you for your cooperation and assistance in this matter.
> 
> Sincerely,
> Lt. John Doe
> Cyberdetective
> District 20 PD

Ako už napovedá hint, existuje spôsob ako skonvertovať MAC adresu na IPV6 adresu. Konkrétne sa jedná o formát EUI-64

[https://eui64-calc.princelle.org/](https://eui64-calc.princelle.org/ "https://eui64-calc.princelle.org/")

Kalkulačka vyhodí 


    2001:db8:7cc::2CA:7AFF:FED0:EA71

Z textu vieme, že máme navštívť adresu na porte 1701. tak do prehliadača zadáme (keďže sa jedná o ipv6 adresu, tú je potrebbé ohraničiť hranatými zátvorkami)

    http://[2001:db8:7cc:0:2ca:7aff:fed0:ea71]:1701/

**Stránka obsahuje nasledovné info:**

|Parameter|Details|
|---|---|
|IP Address|`2001:DB8:7CC::2CA:7AFF:FED0:EA71`|
|GPS Coordinates|50.098° N, 14.388° E|
|Operating System|Windows 12 Imperial Edition|
|Time Zone|UTC (+00:00)|
|Processor|Intel Core i7-9700K|
|Memory|32 GB|
|Storage|512 GB SSD|
|Moggie Status|Enabled (Passive-Agressive mode)|
|Autodestruction|Disabled|
|Service Tag|**FLAG{Sxwr-slvA-pBuT-CzyD}**|

----------

## Vlajka ##
    FLAG{Sxwr-slvA-pBuT-CzyD}