+++
date = '2026-08-14T12:12:00+02:00'
draft = false
title = 'netsh interface portproxy en hacking : faire du port forwarding sous Windows'
description = "Comprendre netsh interface portproxy pour mettre en place une redirection TCP sous Windows et étudier son utilisation dans un laboratoire de pivoting réseau."
tags = ['Windows', 'netsh', 'portproxy', 'TCP', 'Pivoting', 'Red Team', 'Blue Team']
categories = ['Réseau', 'Hacking']
+++

> `GREY-X // WINDOWS LAB`
>
> `netsh interface portproxy — TCP PORT FORWARDING`

Windows possède nativement un mécanisme permettant de **rediriger des connexions TCP** d'un port local vers une autre adresse et un autre port.

La commande utilisée est :

```cmd
netsh interface portproxy
```

C'est particulièrement intéressant en cybersécurité pour comprendre :

```text
Port Forwarding
TCP Redirection
Pivoting
Network Segmentation
Access Control
```

Contrairement à :

```cmd
netsh winhttp
```

qui concerne la configuration d'un proxy **WinHTTP**, `portproxy` travaille au niveau du **trafic TCP**.

---

# `01 // Le concept`

Imaginons :

```text
ATTACKER
   │
   │ TCP :8080
   ▼
WINDOWS PIVOT
   │
   │ TCP :80
   ▼
INTERNAL SERVER
```

Le poste Windows joue ici le rôle d'intermédiaire.

L'attaquant se connecte à :

```text
WINDOWS:8080
```

et Windows transmet la connexion vers :

```text
INTERNAL-SERVER:80
```

On peut représenter le flux :

```text
CLIENT
  │
  │ TCP :8080
  ▼
WINDOWS
  │
  │ TCP :80
  ▼
SERVER
```

---

# `02 // Syntaxe`

La syntaxe principale est :

```cmd
netsh interface portproxy add v4tov4 ^
listenaddress=0.0.0.0 ^
listenport=8080 ^
connectaddress=10.10.20.10 ^
connectport=80
```

Les paramètres importants :

```text
listenaddress
    → adresse sur laquelle Windows écoute

listenport
    → port exposé localement

connectaddress
    → destination réelle

connectport
    → port de destination
```

Dans notre exemple :

```text
0.0.0.0:8080
       ↓
10.10.20.10:80
```

---

# `03 // Comprendre v4tov4`

La commande :

```text
v4tov4
```

signifie :

```text
IPv4 → IPv4
```

Windows propose également des variantes pour les environnements IPv6.

Dans un laboratoire IPv4 classique, on utilisera généralement :

```cmd
netsh interface portproxy add v4tov4
```

---

# `04 // Créer une redirection`

Exemple de laboratoire :

```text
Windows Pivot
192.168.1.50

Serveur interne
10.10.10.20:80
```

Sur Windows :

```cmd
netsh interface portproxy add v4tov4 ^
listenaddress=192.168.1.50 ^
listenport=8080 ^
connectaddress=10.10.10.20 ^
connectport=80
```

La topologie devient :

```text
CLIENT
  │
  │ 192.168.1.50:8080
  ▼
WINDOWS
  │
  │ 10.10.10.20:80
  ▼
INTERNAL SERVER
```

Le client n'a donc pas besoin de se connecter directement au serveur interne.

---

# `05 // Lister les règles`

Pour afficher les règles existantes :

```cmd
netsh interface portproxy show all
```

Exemple :

```text
Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- --------    --------------- --------
192.168.1.50    8080        10.10.10.20     80
```

Cette commande est particulièrement importante en audit.

Elle permet de découvrir :

```text
ports exposés
destinations internes
interfaces utilisées
redirections TCP
```

---

# `06 // Tester le port`

Après avoir créé une règle, on peut vérifier que Windows écoute :

```cmd
netstat -ano | findstr :8080
```

Ou avec PowerShell :

```powershell
Get-NetTCPConnection -LocalPort 8080
```

L'objectif est de vérifier :

```text
LISTENING
```

sur le port configuré.

---

# `07 // Tester depuis une autre machine`

Depuis une machine du laboratoire :

```cmd
curl http://192.168.1.50:8080
```

ou avec PowerShell :

```powershell
Test-NetConnection 192.168.1.50 -Port 8080
```

Le flux attendu est :

```text
Client
  │
  │ :8080
  ▼
192.168.1.50
  │
  │ :80
  ▼
10.10.10.20
```

---

# `08 // Pourquoi c'est intéressant en Red Team ?`

`portproxy` devient particulièrement intéressant lorsqu'une machine possède accès à **plusieurs réseaux**.

Exemple :

```text
             INTERNET
                 │
                 │
          ┌──────▼──────┐
          │ Windows     │
          │ Pivot       │
          │             │
          │ 192.168.1.50│
          │ 10.10.10.50 │
          └──────┬──────┘
                 │
                 │
          INTERNAL NETWORK
                 │
        ┌────────┴────────┐
        ▼                 ▼
   10.10.10.20        10.10.10.30
      WEB               DB
```

La machine Windows possède une visibilité réseau que l'autre machine n'a peut-être pas.

On peut alors étudier le concept :

```text
ACCESS
  ↓
PIVOT
  ↓
PORT FORWARD
  ↓
INTERNAL SERVICE
```

Dans un pentest autorisé, cela permet notamment de démontrer l'impact d'une compromission d'un poste disposant d'une double connectivité.

---

# `09 // Port forwarding ≠ routage`

C'est une distinction importante.

Avec `portproxy` :

```text
TCP CONNECTION
      ↓
PORT FORWARD
      ↓
DESTINATION
```

Ce n'est pas la même chose qu'un routeur IP classique.

Le port forwarding concerne ici une **connexion TCP donnée**.

On peut donc avoir :

```text
192.168.1.50:8080
        ↓
10.10.10.20:80
```

sans transformer Windows en routeur permettant automatiquement à tout le trafic IP de traverser la machine.

---

# `10 // Supprimer une règle`

Pour supprimer la redirection :

```cmd
netsh interface portproxy delete v4tov4 ^
listenaddress=192.168.1.50 ^
listenport=8080
```

Puis :

```cmd
netsh interface portproxy show all
```

La règle ne doit plus apparaître.

Dans un laboratoire, toujours nettoyer les règles après les tests.

---

# `11 // Un exemple avec SSH`

Dans un laboratoire, on peut également rediriger un port TCP utilisé par SSH.

Par exemple :

```text
Windows Pivot
192.168.1.50:2222

        ↓

Linux interne
10.10.10.20:22
```

Configuration :

```cmd
netsh interface portproxy add v4tov4 ^
listenaddress=192.168.1.50 ^
listenport=2222 ^
connectaddress=10.10.10.20 ^
connectport=22
```

Un client autorisé peut alors tester :

```bash
ssh user@192.168.1.50 -p 2222
```

Le chemin devient :

```text
SSH CLIENT
    │
    │ :2222
    ▼
WINDOWS
    │
    │ :22
    ▼
LINUX INTERNAL
```

Cela constitue un excellent laboratoire pour comprendre le **TCP port forwarding**.

---

# `12 // Attention au pare-feu Windows`

Créer une règle `portproxy` ne signifie pas nécessairement que le port est accessible depuis le réseau.

Le pare-feu Windows peut bloquer :

```text
TCP 8080
```

Il faut donc distinguer :

```text
Portproxy
   +
Firewall
   +
Network reachability
```

Une règle peut exister mais rester inaccessible depuis une autre machine.

Dans un laboratoire, on peut vérifier avec :

```powershell
Test-NetConnection 192.168.1.50 -Port 8080
```

---

# `13 // Le rôle de IP Helper`

Le fonctionnement de `portproxy` dépend notamment du service Windows :

```text
IP Helper
```

On peut vérifier son état avec :

```cmd
sc query iphlpsvc
```

ou :

```powershell
Get-Service iphlpsvc
```

Dans un environnement de laboratoire, cela permet de comprendre pourquoi une configuration `portproxy` peut ne pas fonctionner comme prévu.

---

# `14 // Côté Blue Team`

`portproxy` est également intéressant du point de vue défense.

Une règle inattendue peut révéler :

```python
port forwarding non autorisé
pivoting
exposition d'un service interne
configuration temporaire oubliée
```

Un administrateur peut donc régulièrement vérifier :

```cmd
netsh interface portproxy show all
```

et rechercher les règles inconnues.

On peut ensuite corréler avec :

```text
Windows Firewall
Event Logs
netstat
services
processus
connexions réseau
```

Une règle seule ne prouve pas une compromission.

Mais une règle inconnue vers :

```text
10.10.10.20:22
```

sur un poste utilisateur mérite clairement une investigation.

---

# `15 // Mini-lab GREY-X`

### Topologie

```python
CLIENT
192.168.1.10
      │
      ▼
WINDOWS PIVOT
192.168.1.50
10.10.10.50
      │
      ▼
SERVER INTERNE
10.10.10.20:80
```

### Étape 1 — Créer la redirection

Sur Windows :

```cmd
netsh interface portproxy add v4tov4 ^
listenaddress=192.168.1.50 ^
listenport=8080 ^
connectaddress=10.10.10.20 ^
connectport=80
```

### Étape 2 — Vérifier

```cmd
netsh interface portproxy show all
```

### Étape 3 — Vérifier l'écoute

```cmd
netstat -ano | findstr :8080
```

### Étape 4 — Tester depuis le client

```bash
curl http://192.168.1.50:8080
```

### Étape 5 — Nettoyer

```python
netsh interface portproxy delete v4tov4 ^
listenaddress=192.168.1.50 ^
listenport=8080
```

Puis :

```cmd
netsh interface portproxy show all
```

---

# `16 // Commandes à retenir`

```cmd
netsh interface portproxy show all
```

→ afficher les redirections.

```cmd
netsh interface portproxy add v4tov4 listenaddress=192.168.1.50 listenport=8080 connectaddress=10.10.10.20 connectport=80
```

→ créer une redirection TCP IPv4 → IPv4.

```cmd
netsh interface portproxy delete v4tov4 listenaddress=192.168.1.50 listenport=8080
```

→ supprimer une redirection.

```cmd
netstat -ano | findstr :8080
```

→ vérifier l'écoute locale.

```cmd
sc query iphlpsvc
```

→ vérifier le service IP Helper.

---

# `17 // La vision pentest`

Dans un scénario de pivoting :

```python
             ATTACKER
                 │
                 │ TCP
                 ▼
          ┌──────────────┐
          │ Windows      │
          │ Pivot        │
          └──────┬───────┘
                 │
          portproxy
                 │
                 ▼
          INTERNAL SERVER
```

La commande :

```cmd
netsh interface portproxy
```

permet donc d'étudier une technique fondamentale :

> **utiliser une machine intermédiaire pour rendre accessible un service TCP situé sur un autre réseau.**

Pour un pentester, cela permet de démontrer les conséquences possibles d'une machine compromise disposant d'une connectivité privilégiée.

Pour un défenseur, cela rappelle qu'une machine Windows possédant plusieurs interfaces réseau doit être surveillée comme un **point potentiel de pivot**.

---

# `18 // À retenir`

```python
netsh winhttp
    → configuration proxy WinHTTP

netsh interface portproxy
    → redirection TCP

net use
    → connexions SMB

net share
    → partages Windows

icacls
    → permissions NTFS
```

On commence alors à avoir une véritable boîte à outils **native Windows** :

```python
          WINDOWS NATIVE TOOLS
                  │
     ┌────────────┼────────────┐
     ▼            ▼            ▼
   SMB           ACL          TCP
     │            │            │
 net use        icacls     portproxy
     │            │            │
     └────────────┼────────────┘
                  ▼
           NETWORK ANALYSIS
                  │
                  ▼
            SECURITY LAB
```

> **`portproxy` ne crée pas un proxy HTTP. Il crée une redirection TCP.**

> `GREY-X — Forward. Pivot. Understand. Secure.`
