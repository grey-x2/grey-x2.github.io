+++
date = '2026-08-14T11:52:00+02:00'
draft = false
title = 'Commandes IP Linux : 80 % de pratique'
description = "Un guide pratique pour maîtriser les commandes IP sous Linux à travers des manipulations réseau concrètes."
tags = ['Linux', 'Networking', 'IP', 'iproute2', 'Network']
categories = ['Système', 'Réseau']
+++

> `GREY-X // LINUX NETWORK LAB`
>
> `iproute2 — PRATIQUE`

La commande `ip` est l'un des outils essentiels pour administrer et analyser un réseau Linux.

L'objectif ici n'est pas de mémoriser des commandes, mais de **les utiliser dans des situations réelles**.

---

# `01 // Voir les interfaces`

```bash
ip link
```

Ou :

```bash
ip addr
```

Observer :

```text
interface
MAC
IPv4
IPv6
état UP/DOWN
```

---

# `02 // Activer / désactiver une interface`

```bash
sudo ip link set eth0 up
```

Désactiver :

```bash
sudo ip link set eth0 down
```

Vérifier :

```bash
ip link show eth0
```

---

# `03 // Ajouter une adresse IP`

```bash
sudo ip addr add 192.168.10.10/24 dev eth0
```

Vérifier :

```bash
ip addr show eth0
```

Supprimer :

```bash
sudo ip addr del 192.168.10.10/24 dev eth0
```

---

# `04 // Ajouter une route`

Afficher la table de routage :

```bash
ip route
```

Ajouter une route :

```bash
sudo ip route add 192.168.20.0/24 via 192.168.10.1
```

Ajouter une route par défaut :

```bash
sudo ip route add default via 192.168.10.1
```

Supprimer :

```bash
sudo ip route del 192.168.20.0/24
```

---

# `05 // Comprendre le routage`

Tester :

```bash
ip route get 8.8.8.8
```

Linux indique notamment :

```text
dev eth0
via 192.168.10.1
src 192.168.10.10
```

Cette commande est extrêmement utile pour comprendre :

> **Par quelle interface Linux va réellement envoyer un paquet ?**

---

# `06 // Créer un petit laboratoire**

Créer deux interfaces virtuelles :

```bash
sudo ip link add veth0 type veth peer name veth1
```

Activer :

```bash
sudo ip link set veth0 up
sudo ip link set veth1 up
```

Attribuer des adresses :

```bash
sudo ip addr add 10.10.10.1/24 dev veth0
sudo ip addr add 10.10.10.2/24 dev veth1
```

Tester :

```bash
ping 10.10.10.2
```

Vous venez de créer une petite liaison réseau directement dans Linux.

---

# `07 // Observer les voisins`

```bash
ip neigh
```

On peut voir les correspondances :

```text
IP
 ↓
MAC
 ↓
interface
```

Exemple :

```text
192.168.10.20 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

---

# `08 // Ajouter une route spécifique`

Exemple :

```bash
sudo ip route add 10.20.0.0/16 via 192.168.10.254 dev eth0
```

Vérifier :

```bash
ip route
```

Puis :

```bash
ip route get 10.20.10.10
```

---

# `09 // Supprimer le laboratoire`

Supprimer les adresses :

```bash
sudo ip addr flush dev veth0
sudo ip addr flush dev veth1
```

Supprimer les interfaces :

```bash
sudo ip link del veth0
```

La suppression de `veth0` supprime également son pair `veth1`.

---

# `10 // Les commandes à retenir`

```text
ip link
    → interfaces

ip addr
    → adresses IP

ip route
    → routage

ip route get
    → chemin utilisé par un paquet

ip neigh
    → voisins / ARP / NDP

ip link set
    → état d'une interface

ip addr add/del
    → ajouter / supprimer une IP

ip route add/del
    → ajouter / supprimer une route
```

---

# `11 // 80% pratique`

Ne vous contentez pas de lire :

```bash
ip addr
ip route
ip neigh
```

Créez un laboratoire, modifiez la configuration, cassez-la, puis réparez-la.

```text
Créer
  ↓
Configurer
  ↓
Tester
  ↓
Observer
  ↓
Casser
  ↓
Dépanner
  ↓
Comprendre
```

C'est cette boucle qui permet réellement de maîtriser le réseau Linux.

> **GREY-X — 20% théorie. 80% pratique.**
