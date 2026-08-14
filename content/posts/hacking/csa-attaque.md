+++
date = '2026-08-14T11:49:00+02:00'
draft = false
title = 'CSA Attack Wi-Fi : comprendre l’attaque'
description = "Comprendre le CSA Attack et son impact sur les réseaux Wi-Fi modernes."
tags = ['Wi-Fi', 'CSA', 'WPA2', 'WPA3', 'Sécurité']
categories = ['Wi-Fi', 'Hacking']
+++

> `GREY-X // WIFI LAB`
>
> `CSA ATTACK`

Le **Channel Switch Announcement (CSA)** est un mécanisme Wi-Fi permettant à un point d'accès d'indiquer à ses clients qu'il va changer de canal.

Dans certaines conditions, ce mécanisme peut être exploité pour perturber la communication entre les clients et le point d'accès et faciliter certaines phases d'une analyse de sécurité Wi-Fi.

---

# `01 // Principe`

Le fonctionnement normal :

```text
AP
 │
 │ CSA → changement de canal
 ▼
Client
 │
 └── suit le changement
```

Lors d'un scénario d'attaque :

```text
Attaquant
    │
    ▼
Perturbation / CSA
    │
    ▼
Client
    │
    ▼
Changement de canal
```

L'objectif est notamment d'étudier la manière dont les clients réagissent aux changements de canal et aux perturbations du réseau.

---

# `02 // Pourquoi c'est intéressant ?`

Le CSA Attack permet d'étudier :

* la gestion des changements de canal ;
* le comportement des clients Wi-Fi ;
* les mécanismes de reconnexion ;
* l'impact sur la capture de trafic d'authentification ;
* les différences de comportement entre WPA2 et WPA3.

Il ne s'agit donc pas de « casser WPA3 » directement.

> **Le point faible étudié est le comportement autour du canal et de la reconnexion, pas le chiffrement WPA3 lui-même.**

---

# `03 // Observer le phénomène`

Dans un laboratoire autorisé, on peut utiliser :

```bash
iw dev
```

et :

```bash
sudo tcpdump -i wlan1 -n
```

Wireshark permet ensuite d'analyser les trames de gestion et d'observer les changements de canal.

---

# `04 // WPA2 / WPA3`

Le CSA Attack ne signifie pas que :

```text
WPA2 = cassé
WPA3 = cassé
```

Il permet plutôt d'étudier les mécanismes entourant la connexion.

```text
Wi-Fi
 │
 ├── Gestion du canal
 │
 ├── Association
 │
 ├── Authentification
 │
 └── Chiffrement
```

Une faiblesse dans une couche ne signifie pas automatiquement que le chiffrement est compromis.

---

# `05 // À retenir`

```text
CSA
 ↓
Changement de canal
 ↓
Réaction du client
 ↓
Reconnexion / perturbation
 ↓
Analyse du comportement Wi-Fi
```

Le meilleur moyen de comprendre cette technique reste de la reproduire dans un **laboratoire contrôlé**, puis d'observer les trames avec Wireshark.

> **GREY-X — Understand the protocol, don't trust the illusion.**
