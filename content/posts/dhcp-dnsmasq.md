+++
date = '2026-08-14T11:09:00+02:00'
draft = false
title = 'DHCP avec dnsmasq : comprendre et analyser les mécanismes d’attribution réseau'
description = "Comprendre le fonctionnement du DHCP avec dnsmasq et apprendre à analyser les échanges réseau dans un laboratoire contrôlé."
tags = ['DHCP', 'dnsmasq', 'Réseau', 'Pentesting', 'Linux']
categories = ['Réseau']
image = "img/dnsmasq.png" 
featuredImage = "img/dnsmasq.png"
+++

> `GREY-X // NETWORK LAB`
>
> `DHCP DISCOVERY → OFFER → REQUEST → ACK`

Le DHCP est l'un de ces protocoles que l'on utilise quotidiennement sans forcément réfléchir à ce qui se passe réellement sur le réseau.

Lorsqu'une machine se connecte à un réseau, elle doit généralement obtenir une adresse IP, une passerelle, des serveurs DNS et parfois plusieurs autres paramètres.

Derrière cette opération apparemment simple se trouve le protocole **DHCP — Dynamic Host Configuration Protocol**.

Dans cet article, nous allons utiliser **dnsmasq** pour construire un petit laboratoire DHCP et observer ce qui se passe réellement sur le réseau.

---

## `01 // Le rôle du DHCP`

Lorsqu'un client ne possède pas encore d'adresse IP, il peut commencer par envoyer une requête DHCP.

Le processus classique repose sur quatre étapes :

```text
DHCPDISCOVER
      ↓
DHCPOFFER
      ↓
DHCPREQUEST
      ↓
DHCPACK
```

On parle souvent de **DORA** :

* **Discover** — le client recherche un serveur DHCP ;
* **Offer** — le serveur propose une configuration ;
* **Request** — le client demande cette configuration ;
* **ACK** — le serveur confirme l'attribution.

À la fin du processus, le client peut disposer notamment de :

```text
IP address
Subnet mask
Default gateway
DNS server
Lease time
```

Le DHCP ne sert donc pas uniquement à donner une adresse IP.

Il peut également fournir de nombreux paramètres permettant au client de fonctionner correctement sur le réseau.

---

## `02 // Pourquoi dnsmasq ?`

`dnsmasq` est particulièrement intéressant dans un laboratoire réseau parce qu'il est léger et capable de fournir plusieurs services.

Il peut notamment être utilisé pour :

* DHCP ;
* DNS ;
* PXE ;
* annonces réseau dans certains environnements ;
* petits laboratoires et réseaux de test.

Pour une plateforme de test, cela permet de reproduire facilement différents scénarios réseau sans déployer une infrastructure DHCP complexe.

---

## `03 // Construire le laboratoire`

Pour éviter toute interaction avec un réseau réel, nous allons travailler sur une interface réseau dédiée à notre laboratoire.

Exemple :

```text
                    ┌─────────────────┐
                    │   DHCP Server   │
                    │    dnsmasq      │
                    │   192.168.50.1  │
                    └────────┬────────┘
                             │
                         eth1 / lab
                             │
              ┌──────────────┴──────────────┐
              │                             │
        ┌─────▼─────┐                 ┌─────▼─────┐
        │  Client 1 │                 │  Client 2 │
        │ DHCP      │                 │ DHCP      │
        └───────────┘                 └───────────┘
```

L'idée est simple :

**un serveur DHCP → un réseau isolé → plusieurs clients de test.**

---

## `04 // Configuration de dnsmasq`

Sur une machine Linux de laboratoire, installons dnsmasq :

```bash
sudo apt install dnsmasq
```

Avant de lancer le service, il est préférable de créer une configuration dédiée au laboratoire.

Par exemple :

```ini
interface=eth1
bind-interfaces

dhcp-range=192.168.50.100,192.168.50.200,255.255.255.0,12h

dhcp-option=3,192.168.50.1
dhcp-option=6,192.168.50.1
```

Cette configuration indique notamment que :

```text
Interface       : eth1
Réseau          : 192.168.50.0/24
Pool DHCP       : 192.168.50.100 - 192.168.50.200
Lease           : 12 heures
Gateway         : 192.168.50.1
DNS             : 192.168.50.1
```

Le réseau doit être **isolé du réseau de production**.

---

## `05 // Observer le trafic DHCP`

C'est ici que le laboratoire devient réellement intéressant.

Au lieu de simplement lancer dnsmasq, nous pouvons observer les paquets avec Wireshark ou `tcpdump`.

Par exemple :

```bash
sudo tcpdump -ni eth1 port 67 or port 68
```

Lorsqu'un client demande une adresse, on peut observer les échanges DHCP.

On retrouvera notamment :

```text
DHCP Discover
DHCP Offer
DHCP Request
DHCP ACK
```

Cette observation permet de comprendre concrètement ce que les outils affichent derrière leur interface.

---

## `06 // Lire une requête DHCP`

Une requête DHCP contient beaucoup plus d'informations qu'une simple demande d'adresse IP.

Selon le client et sa configuration, on peut retrouver différentes options DHCP.

Par exemple :

```text
Option 53 → DHCP Message Type
Option 50 → Requested IP Address
Option 55 → Parameter Request List
Option 61 → Client Identifier
Option 12 → Hostname
```

La **Parameter Request List** est particulièrement intéressante lors d'une analyse réseau.

Elle indique certains paramètres que le client souhaite recevoir du serveur DHCP.

Cela permet également de comprendre pourquoi deux systèmes différents peuvent générer des requêtes DHCP légèrement différentes.

---

## `07 // Pourquoi le DHCP est intéressant en pentesting ?`

Le DHCP est intéressant parce qu'il se trouve très tôt dans le processus de connexion d'un client au réseau.

Une mauvaise configuration de l'infrastructure DHCP peut avoir des conséquences importantes.

Lors d'un audit autorisé, on peut notamment vérifier :

* quels serveurs DHCP répondent sur un segment ;
* si plusieurs serveurs DHCP sont présents ;
* quelles options sont distribuées ;
* quelles informations sont exposées ;
* si les clients reçoivent la bonne passerelle ;
* si les serveurs DNS distribués sont ceux attendus ;
* si les mécanismes de protection du réseau sont correctement configurés.

Le point important est de distinguer **l'analyse** d'une infrastructure et la perturbation d'un réseau.

Dans un environnement réel, les tests DHCP doivent être explicitement autorisés et réalisés avec des précautions adaptées.

---

## `08 // Le laboratoire avant l'infrastructure réelle`

Un laboratoire DHCP permet de reproduire de nombreux scénarios sans toucher à une infrastructure de production.

On peut par exemple créer plusieurs VLANs, plusieurs clients, plusieurs serveurs DHCP ou combiner DHCP avec DNS, IPv6 et routage.

Cela permet également de développer un réflexe essentiel en pentesting réseau :

> **Ne pas seulement regarder le résultat. Observer le protocole qui produit ce résultat.**

Une adresse IP attribuée automatiquement paraît banale.

Mais derrière cette adresse se trouve tout un échange réseau.

---

## `09 // Quelques commandes utiles`

Vérifier l'état du service :

```bash
sudo systemctl status dnsmasq
```

Voir les logs :

```bash
sudo journalctl -u dnsmasq
```

Vérifier les ports DHCP :

```bash
sudo ss -lunp | grep -E ':67|:68'
```

Observer les paquets :

```bash
sudo tcpdump -ni eth1 port 67 or port 68
```

Tester la configuration avant de démarrer le service :

```bash
sudo dnsmasq --test
```

---

## `10 // À retenir`

Le DHCP paraît simple :

```text
Client → Demande une configuration
Serveur → Fournit une configuration
```

Mais lorsqu'on analyse réellement les paquets, on découvre un protocole riche en informations et fortement lié au fonctionnement du réseau.

Avec `dnsmasq`, il devient très facile de construire un laboratoire permettant d'étudier ces mécanismes.

Pour progresser en pentesting réseau, il est important de ne pas apprendre uniquement les commandes.

**Comprendre les protocoles permet de comprendre les attaques.**

```text
┌──────────────────────────────────────┐
│              GREY-X                  │
│                                      │
│  CAPTURE → ANALYZE → UNDERSTAND      │
│                                      │
│        NETWORK SECURITY LAB           │
└──────────────────────────────────────┘
```

**On apprend. On teste. On documente. On partage.**
