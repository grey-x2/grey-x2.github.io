+++
date = '2026-08-14T12:11:00+02:00'
draft = false
title = 'netsh en hacking : configurer et analyser un proxy WinHTTP sous Windows'
description = "Comprendre netsh winhttp pour configurer, inspecter et réinitialiser le proxy utilisé par les applications WinHTTP dans un laboratoire Windows."
tags = ['Windows', 'netsh', 'WinHTTP', 'Proxy', 'Red Team', 'Blue Team']
categories = ['Système', 'Hacking']
+++

> `GREY-X // WINDOWS LAB`
>
> `netsh — WINHTTP PROXY`

`netsh` est un outil natif de Windows permettant de configurer et d'inspecter différents composants réseau.

Dans un laboratoire de cybersécurité, une partie particulièrement intéressante est :

```cmd
netsh winhttp
```

Cette fonctionnalité permet de gérer la configuration **WinHTTP Proxy** utilisée par certaines applications et certains services Windows.

Il faut cependant faire une distinction importante :

```text
netsh
    ≠
proxy complet
```

`netsh` permet principalement de **configurer le client WinHTTP pour utiliser un proxy existant**.

Il ne transforme pas à lui seul une machine Windows en serveur proxy HTTP comme pourrait le faire Squid ou mitmproxy.

---

# `01 // Afficher la configuration actuelle`

La première commande à retenir :

```cmd
netsh winhttp show proxy
```

Exemple :

```text
Current WinHTTP proxy settings:

    Direct access (no proxy server).
```

Ou :

```text
Proxy Server(s) : proxy.lab.local:8080
Bypass List     : localhost;127.0.0.1
```

Cette commande permet donc de répondre rapidement à la question :

```text
Windows utilise-t-il actuellement
un proxy WinHTTP ?
```

---

# `02 // Configurer un proxy`

Dans un laboratoire, on peut configurer un proxy existant :

```cmd
netsh winhttp set proxy proxy.lab.local:8080
```

Avec une adresse IP :

```cmd
netsh winhttp set proxy 192.168.56.20:8080
```

Le fonctionnement devient :

```text
WINDOWS
   │
   │ HTTP/HTTPS
   ▼
PROXY
192.168.56.20:8080
   │
   ▼
   INTERNET
```

Le proxy doit évidemment être réellement présent et accepter les connexions.

`netsh` ne crée pas le service proxy.

---

# `03 // Ajouter une liste d'exclusion`

On peut également définir des destinations qui ne doivent pas utiliser le proxy :

```cmd
netsh winhttp set proxy proxy.lab.local:8080 "localhost;127.0.0.1;*.lab.local"
```

La logique devient :

```text
Application
    │
    ├── destination exclue
    │       ↓
    │     DIRECT
    │
    └── autre destination
            ↓
          PROXY
```

Cette notion est importante dans les environnements d'entreprise.

---

# `04 // Vérifier après modification`

Après avoir configuré le proxy :

```cmd
netsh winhttp show proxy
```

Il faut vérifier que Windows retourne bien la configuration attendue.

Exemple :

```text
Proxy Server(s) : 192.168.56.20:8080
Bypass List     : localhost;127.0.0.1
```

On peut alors comparer :

```text
AVANT
Direct access

        ↓

CONFIGURATION

        ↓

APRÈS
192.168.56.20:8080
```

---

# `05 // Réinitialiser le proxy`

Pour revenir au comportement direct :

```cmd
netsh winhttp reset proxy
```

La configuration revient généralement à :

```text
Direct access (no proxy server)
```

Puis vérifier :

```cmd
netsh winhttp show proxy
```

Cette étape est importante dans un laboratoire afin de ne pas laisser une configuration réseau temporaire derrière soi.

---

# `06 // WinHTTP ≠ navigateur`

Une confusion fréquente consiste à penser que :

```cmd
netsh winhttp set proxy ...
```

configure automatiquement tous les navigateurs.

Ce n'est pas le cas.

WinHTTP possède sa propre configuration.

On peut donc avoir :

```text
Navigateur
   ↓
Proxy navigateur

Service Windows
   ↓
WinHTTP Proxy
```

Les deux configurations peuvent être différentes.

---

# `07 // Importer la configuration Internet`

Windows fournit également :

```cmd
netsh winhttp import proxy source=ie
```

Cette commande permet d'importer la configuration proxy des paramètres Internet du système vers WinHTTP.

Pour vérifier ensuite :

```cmd
netsh winhttp show proxy
```

Cela peut être utile dans certains environnements où les paramètres proxy sont déjà définis au niveau Windows.

---

# `08 // Pourquoi c'est intéressant en Red Team ?`

Lors d'un audit Windows, la configuration proxy peut révéler une partie de l'architecture réseau :

```text
POSTE
  │
  ▼
PROXY ENTREPRISE
  │
  ▼
INTERNET
```

La commande :

```cmd
netsh winhttp show proxy
```

peut donc révéler :

```text
adresse du proxy
port
liste d'exclusion
architecture réseau
```

Par exemple :

```text
Proxy Server(s) : 10.20.30.40:8080
```

Cette information peut être utile pour comprendre :

```text
comment les applications sortent du réseau
```

---

# `09 // Proxy dans un laboratoire`

On peut construire un laboratoire simple :

```text
┌──────────────┐
│ Windows VM   │
│ 10.10.10.10  │
└──────┬───────┘
       │
       │ HTTP
       ▼
┌──────────────┐
│ Proxy        │
│ 10.10.10.20  │
│ :8080        │
└──────┬───────┘
       │
       ▼
    Internet
```

Sur Windows :

```cmd
netsh winhttp set proxy 10.10.10.20:8080
```

Puis :

```cmd
netsh winhttp show proxy
```

Le proxy doit être configuré séparément sur la machine `10.10.10.20`.

Par exemple, dans un laboratoire contrôlé, on peut utiliser un outil de proxy HTTP prévu à cet effet.

---

# `10 // Observer le trafic`

Une fois le proxy configuré, l'intérêt du laboratoire est d'observer le chemin réseau :

```text
Application
     │
     ▼
WinHTTP
     │
     ▼
Proxy
     │
     ▼
Destination
```

On peut alors comparer :

```text
DIRECT
```

avec :

```text
PROXY
```

et observer les différences avec un analyseur réseau comme Wireshark.

---

# `11 // HTTPS : point important`

Configurer un proxy ne signifie pas automatiquement que celui-ci peut lire le contenu HTTPS.

Avec :

```text
HTTPS
```

la communication est chiffrée.

On peut avoir :

```text
Windows
   │
   │ TLS
   ▼
Proxy
   │
   │ TLS
   ▼
Server
```

Le proxy peut alors relayer la connexion sans nécessairement voir le contenu applicatif.

L'interception HTTPS nécessite un mécanisme supplémentaire, notamment une configuration TLS appropriée et, dans un laboratoire autorisé, une infrastructure de certificat de test.

---

# `12 // Côté Blue Team`

Le proxy est également un élément important de la défense.

Un administrateur peut vérifier :

```cmd
netsh winhttp show proxy
```

et rechercher :

```text
proxy inconnu
adresse inattendue
port inhabituel
bypass trop large
configuration persistante non documentée
```

Une configuration comme :

```text
Proxy Server(s) : 10.10.10.50:8080
```

doit être cohérente avec l'architecture réseau de l'organisation.

---

# `13 // Mini-lab GREY-X`

### Étape 1 — Vérifier

```cmd
netsh winhttp show proxy
```

### Étape 2 — Configurer le proxy du laboratoire

```cmd
netsh winhttp set proxy 192.168.56.20:8080
```

### Étape 3 — Vérifier

```cmd
netsh winhttp show proxy
```

### Étape 4 — Générer du trafic avec une application utilisant WinHTTP

Observer ensuite le trafic côté proxy et avec Wireshark.

### Étape 5 — Restaurer la configuration

```cmd
netsh winhttp reset proxy
```

Puis :

```cmd
netsh winhttp show proxy
```

L'objectif est de comprendre :

```text
APPLICATION
     ↓
   WinHTTP
     ↓
   PROXY
     ↓
  NETWORK
```

---

# `14 // Commandes à retenir`

```cmd
netsh winhttp show proxy
```

→ afficher la configuration.

```cmd
netsh winhttp set proxy 192.168.56.20:8080
```

→ configurer un proxy WinHTTP.

```cmd
netsh winhttp set proxy proxy.lab.local:8080 "localhost;127.0.0.1"
```

→ configurer un proxy avec exclusions.

```cmd
netsh winhttp reset proxy
```

→ supprimer la configuration proxy WinHTTP.

```cmd
netsh winhttp import proxy source=ie
```

→ importer une configuration proxy Internet vers WinHTTP.

---

# `15 // La vision pentest`

Une simple commande :

```cmd
netsh winhttp show proxy
```

peut révéler une information importante sur le chemin réseau d'une machine :

```text
             WINDOWS
                │
                ▼
             WinHTTP
                │
        ┌───────┴───────┐
        │               │
      DIRECT           PROXY
                        │
                        ▼
                    INTERNET
```

Dans un pentest, l'intérêt est donc moins de « créer un proxy avec `netsh` » que de comprendre **comment Windows est configuré pour utiliser un proxy**.

> **`netsh` configure le client WinHTTP. Le serveur proxy, lui, doit être fourni par un autre composant.**

> `GREY-X — Configure. Observe. Understand.`