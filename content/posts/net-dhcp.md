+++
date = '2026-08-14T11:12:00+02:00'
draft = false
title = 'Construire un routeur NAT Linux avec DHCP, iptables et deux interfaces réseau'
description = "Mise en place d’un routeur Linux avec DHCP sur eth1, accès Internet via eth0 et NAT avec iptables."
tags = ['NAT', 'iptables', 'DHCP', 'Linux', 'Network Security', 'Pentesting']
categories = ['Network Security']
+++

> `GREY-X // NETWORK LAB`
>
> `ETH0 → INTERNET`
>
> `ETH1 → LAN`
>
> `NAT → FORWARD → INTERNET`

Dans un laboratoire de pentesting réseau, il est souvent nécessaire de construire une petite infrastructure permettant à plusieurs machines de communiquer avec Internet tout en restant derrière un réseau privé.

Une machine Linux peut parfaitement jouer ce rôle.

Dans cette configuration :

* `eth0` est l'interface connectée à Internet ;
* `eth1` est l'interface connectée au réseau interne ;
* `dnsmasq` fournit les adresses DHCP aux clients ;
* `iptables` assure le routage et le NAT.

L'objectif est d'obtenir :

```text
                    INTERNET
                        │
                      eth0
                        │
              ┌─────────▼─────────┐
              │     LINUX BOX     │
              │                   │
              │  DHCP  dnsmasq    │
              │  NAT   iptables   │
              │  ROUTING          │
              └─────────┬─────────┘
                        │
                      eth1
                  192.168.50.1
                        │
              ┌─────────┴─────────┐
              │                   │
        192.168.50.100      192.168.50.101
             Client 1            Client 2
```

---

## `01 // Vérifier les interfaces`

Avant toute configuration, il faut identifier correctement les interfaces.

```bash
ip -br addr
```

On peut obtenir quelque chose comme :

```text
eth0    UP    192.168.1.20/24
eth1    UP    192.168.50.1/24
```

Dans notre laboratoire :

```text
eth0 → réseau externe
eth1 → réseau interne
```

Cette distinction est fondamentale.

Une erreur d'interface dans une règle NAT ou dans dnsmasq peut rapidement rendre le laboratoire inutilisable.

---

## `02 // Configurer l'interface LAN`

L'interface `eth1` doit posséder une adresse IP statique.

Exemple :

```bash
sudo ip addr flush dev eth1
sudo ip addr add 192.168.50.1/24 dev eth1
sudo ip link set eth1 up
```

Vérification :

```bash
ip addr show eth1
```

On doit retrouver :

```text
inet 192.168.50.1/24
```

Cette adresse devient la passerelle du réseau interne.

---

## `03 // Configurer dnsmasq`

Nous pouvons maintenant distribuer automatiquement des adresses IP aux clients connectés à `eth1`.

Exemple de configuration :

```ini
interface=eth1
bind-interfaces

dhcp-range=192.168.50.100,192.168.50.200,255.255.255.0,12h

dhcp-option=3,192.168.50.1
dhcp-option=6,192.168.50.1
```

La logique devient :

```text
Client
  │
  ├── DHCPDISCOVER
  │
  ▼
dnsmasq
  │
  ├── IP : 192.168.50.x
  ├── GW : 192.168.50.1
  └── DNS : 192.168.50.1
```

Tester la configuration :

```bash
sudo dnsmasq --test
```

Puis redémarrer le service :

```bash
sudo systemctl restart dnsmasq
```

---

## `04 // Activer le forwarding IPv4`

À ce stade, les clients peuvent communiquer avec la machine Linux.

Mais Linux ne transfère pas encore automatiquement leurs paquets entre `eth1` et `eth0`.

Il faut activer le forwarding IPv4 :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Vérification :

```bash
sysctl net.ipv4.ip_forward
```

Résultat attendu :

```text
net.ipv4.ip_forward = 1
```

C'est une étape essentielle.

Sans forwarding :

```text
LAN ──X──> INTERNET
```

Avec forwarding :

```text
LAN ───────────────> INTERNET
```

---

## `05 // Comprendre le NAT`

Le réseau interne utilise des adresses privées :

```text
192.168.50.0/24
```

Ces adresses ne sont généralement pas routables directement sur Internet.

Le NAT va permettre de traduire les connexions sortantes du réseau privé vers l'adresse IP de `eth0`.

Conceptuellement :

```text
Client LAN

192.168.50.100:54321
        │
        ▼
Linux Router
        │
        │ NAT
        ▼
192.168.1.20:54321
        │
        ▼
     INTERNET
```

Le serveur distant voit alors la connexion provenant de l'adresse de l'interface externe.

---

## `06 // La règle iptables MASQUERADE`

Pour un réseau interne utilisant une adresse IP dynamique sur `eth0`, `MASQUERADE` est généralement pratique :

```bash
sudo iptables -t nat -A POSTROUTING \
    -s 192.168.50.0/24 \
    -o eth0 \
    -j MASQUERADE
```

Décomposition :

```text
-t nat
```

Utilise la table NAT.

```text
-A POSTROUTING
```

Ajoute une règle à la chaîne POSTROUTING.

```text
-s 192.168.50.0/24
```

Concerne les paquets provenant du réseau interne.

```text
-o eth0
```

Concerne les paquets qui sortent par `eth0`.

```text
-j MASQUERADE
```

Effectue la traduction d'adresse source.

---

## `07 // Autoriser le forwarding`

Le NAT seul ne suffit pas.

Il faut également autoriser le transfert entre les interfaces :

```bash
sudo iptables -A FORWARD \
    -i eth1 \
    -o eth0 \
    -s 192.168.50.0/24 \
    -j ACCEPT
```

Puis autoriser le trafic de retour :

```bash
sudo iptables -A FORWARD \
    -i eth0 \
    -o eth1 \
    -d 192.168.50.0/24 \
    -m conntrack \
    --ctstate ESTABLISHED,RELATED \
    -j ACCEPT
```

Nous obtenons alors :

```text
             eth1                    eth0
              │                       │
              │                       │
              ▼                       ▼
        ┌───────────┐           ┌──────────┐
        │    LAN    │           │ INTERNET │
        └─────┬─────┘           └────▲─────┘
              │                       │
              │      FORWARD          │
              └───────────────────────┘
                       │
                      NAT
                       │
                 POSTROUTING
```

---

## `08 // Vérifier les règles`

Afficher les règles de forwarding :

```bash
sudo iptables -L FORWARD -n -v
```

Afficher les règles NAT :

```bash
sudo iptables -t nat -L -n -v
```

Les compteurs sont particulièrement intéressants.

Si les paquets traversent réellement la machine, les compteurs devraient augmenter.

Cela permet de vérifier que le trafic passe bien par les règles attendues.

---

## `09 // Tester depuis le client`

Sur un client connecté à `eth1` :

```bash
ip addr
```

On doit obtenir une adresse similaire à :

```text
192.168.50.100/24
```

Vérifier la route :

```bash
ip route
```

On devrait retrouver :

```text
default via 192.168.50.1
```

Tester la passerelle :

```bash
ping 192.168.50.1
```

Puis tester une adresse IP externe :

```bash
ping 1.1.1.1
```

Si cela fonctionne mais que les noms de domaine ne fonctionnent pas :

```bash
ping google.com
```

alors le problème est probablement lié au DNS plutôt qu'au NAT.

---

## `10 // Observer le NAT en temps réel`

Une des parties les plus intéressantes du laboratoire est d'observer les paquets.

Sur le routeur :

```bash
sudo tcpdump -ni eth1
```

Puis :

```bash
sudo tcpdump -ni eth0
```

On peut comparer les paquets entrant par `eth1` avec ceux sortant par `eth0`.

Avant NAT :

```text
SRC = 192.168.50.100
```

Après traduction :

```text
SRC = IP_DE_ETH0
```

C'est ici que le fonctionnement du NAT devient réellement concret.

On ne regarde plus uniquement une règle `iptables`.

On observe la transformation du trafic.

---

## `11 // Vérifier les connexions suivies`

Linux maintient également un état des connexions pour le suivi du trafic.

On peut inspecter les informations conntrack avec :

```bash
sudo conntrack -L
```

Cela permet notamment de comprendre comment Linux associe les connexions sortantes et leurs réponses.

Le mécanisme est particulièrement important lorsque l'on utilise :

```text
ESTABLISHED
RELATED
```

dans les règles du firewall.

---

## `12 // Architecture finale`

Notre laboratoire possède maintenant trois fonctions principales :

```text
                    INTERNET
                        │
                        │
                      eth0
                        │
                ┌───────▼───────┐
                │   LINUX BOX   │
                │               │
                │   iptables    │
                │      NAT      │
                │   FORWARDING  │
                │               │
                │   dnsmasq     │
                │      DHCP     │
                └───────┬───────┘
                        │
                      eth1
                        │
                 192.168.50.0/24
                        │
                 ┌──────┴──────┐
                 │             │
              Client        Client
```

La chaîne logique est donc :

```text
DHCP
 ↓
IP + Gateway + DNS
 ↓
Client → eth1
 ↓
FORWARD
 ↓
NAT / MASQUERADE
 ↓
eth0
 ↓
INTERNET
```

---

## `13 // Ce que ce laboratoire permet d'apprendre`

Cette configuration paraît simple, mais elle permet d'étudier plusieurs concepts fondamentaux :

* DHCP ;
* routage IPv4 ;
* forwarding Linux ;
* NAT ;
* conntrack ;
* iptables ;
* DNS ;
* capture réseau ;
* architecture LAN/WAN.

C'est également une excellente base pour construire ensuite des laboratoires plus complexes : segmentation VLAN, firewalling, proxy, DNS personnalisé, IPv6, captive portal ou encore environnements de test pour l'analyse réseau.

L'intérêt n'est pas seulement de réussir à obtenir Internet sur un client.

L'intérêt est de comprendre **chaque étape du chemin parcouru par le paquet**.

```text
┌──────────────────────────────────────┐
│              GREY-X                  │
│                                      │
│  DHCP → ROUTING → NAT → INTERNET     │
│                                      │
│       UNDERSTAND THE PACKET          │
└──────────────────────────────────────┘
```

**On apprend. On teste. On observe. On documente.**
