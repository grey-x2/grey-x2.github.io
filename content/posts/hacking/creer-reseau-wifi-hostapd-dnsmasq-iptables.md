+++
date = '2026-08-14T11:32:00+02:00'
draft = false
title = 'Créer un réseau Wi-Fi avec hostapd, dnsmasq et iptables'
description = "Construire un laboratoire Wi-Fi Linux avec une Alfa AWUS1900, hostapd, dnsmasq et iptables : point d'accès, DHCP, routage et NAT."
tags = ['Wi-Fi', 'Linux', 'hostapd', 'dnsmasq', 'iptables', 'AWUS1900', 'Networking']
categories = ['Réseau', 'Linux', 'Hacking']
+++

> `GREY-X // WIFI LAB`
>
> `hostapd + dnsmasq + iptables`

Créer son propre point d'accès Wi-Fi sous Linux permet de comprendre concrètement ce qui se passe derrière une connexion sans fil.

Dans ce laboratoire, une machine Linux va jouer plusieurs rôles :

```text
                 INTERNET
                     │
                  eth0
                     │
              ┌──────▼──────┐
              │   LINUX     │
              │             │
              │  iptables   │
              │    NAT      │
              │             │
              │  dnsmasq    │
              │   DHCP/DNS  │
              │             │
              │  hostapd    │
              └──────┬──────┘
                     │
                 wlan1 / AWUS1900
                     │
                )) Wi-Fi ((
                     │
             ┌───────┴───────┐
             │               │
          Client 1         Client 2
```

L'objectif est de créer un réseau Wi-Fi de laboratoire avec :

* **eth0** → accès Internet ;
* **wlan1** → Alfa AWUS1900 ;
* **hostapd** → point d'accès Wi-Fi ;
* **dnsmasq** → DHCP + DNS ;
* **iptables** → routage/NAT.

---

# `01 // Vérifier les interfaces`

Commencer par identifier les interfaces :

```bash
ip addr
```

Dans notre exemple :

```text
eth0  → Internet
wlan1 → Alfa AWUS1900
```

Vérifier également le matériel Wi-Fi :

```bash
iw dev
```

Puis :

```bash
iw list
```

Rechercher notamment les capacités du mode AP.

La première chose à retenir :

> **Le nom de l'interface peut être différent sur votre machine.**

Il peut s'agir de `wlan0`, `wlan1`, `wlx...`, etc.

---

# `02 // Installer les outils`

Sur Debian/Kali :

```bash
sudo apt update
sudo apt install hostapd dnsmasq iptables
```

Sur Fedora :

```bash
sudo dnf install hostapd dnsmasq iptables
```

Vérifier les installations :

```bash
hostapd -v
dnsmasq --version
iptables --version
```

---

# `03 // Configurer l'interface Wi-Fi`

Avant de lancer le point d'accès, attribuer une adresse IP à l'interface Wi-Fi.

Dans notre laboratoire :

```text
192.168.50.1/24
```

Configurer :

```bash
sudo ip addr flush dev wlan1
sudo ip addr add 192.168.50.1/24 dev wlan1
sudo ip link set wlan1 up
```

Vérifier :

```bash
ip addr show wlan1
```

On doit obtenir quelque chose ressemblant à :

```text
inet 192.168.50.1/24
```

---

# `04 // Configurer hostapd`

Créer le fichier :

```bash
sudo nano /etc/hostapd/hostapd.conf
```

Configuration minimale :

```ini
interface=wlan1
driver=nl80211

ssid=GREY-X-LAB
hw_mode=g
channel=6

auth_algs=1

wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_passphrase=GreyX-Lab-2026
```

Ici :

```text
interface       → interface Wi-Fi
ssid            → nom du réseau
channel         → canal Wi-Fi
wpa=2           → WPA2
wpa_key_mgmt    → authentification PSK
rsn_pairwise    → chiffrement CCMP
```

Tester directement hostapd :

```bash
sudo hostapd /etc/hostapd/hostapd.conf
```

Si tout fonctionne, l'interface devrait commencer à diffuser :

```text
GREY-X-LAB
```

Pour arrêter le test :

```text
Ctrl + C
```

---

# `05 // Configurer dnsmasq`

Faire une sauvegarde de la configuration existante :

```bash
sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf.backup
```

Créer une configuration de laboratoire :

```bash
sudo nano /etc/dnsmasq.conf
```

Exemple :

```ini
interface=wlan1
bind-interfaces

dhcp-range=192.168.50.100,192.168.50.200,255.255.255.0,12h

dhcp-option=3,192.168.50.1
dhcp-option=6,192.168.50.1

server=1.1.1.1
server=8.8.8.8
```

Cette configuration indique à dnsmasq :

```text
192.168.50.100 ─┐
192.168.50.101  │
192.168.50.102  │
      ...       ├── DHCP
192.168.50.200 ─┘
```

La passerelle distribuée aux clients sera :

```text
192.168.50.1
```

---

# `06 // Activer le routage IPv4`

La machine Linux doit pouvoir transférer les paquets entre `wlan1` et `eth0`.

Vérifier :

```bash
sysctl net.ipv4.ip_forward
```

Activer temporairement :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Vérifier :

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Résultat attendu :

```text
1
```

---

# `07 // Configurer le NAT avec iptables`

Maintenant, nous voulons :

```text
Client Wi-Fi
     ↓
 wlan1
     ↓
 Linux
     ↓
  eth0
     ↓
 Internet
```

Ajouter le NAT :

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Autoriser le forwarding :

```bash
sudo iptables -A FORWARD -i wlan1 -o eth0 -j ACCEPT
```

Autoriser le trafic de retour :

```bash
sudo iptables -A FORWARD -i eth0 -o wlan1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

Afficher les règles :

```bash
sudo iptables -L -n -v
```

Afficher la table NAT :

```bash
sudo iptables -t nat -L -n -v
```

---

# `08 // Démarrer le laboratoire`

Démarrer dnsmasq :

```bash
sudo systemctl restart dnsmasq
```

Puis hostapd :

```bash
sudo systemctl restart hostapd
```

Vérifier :

```bash
sudo systemctl status dnsmasq
```

et :

```bash
sudo systemctl status hostapd
```

---

# `09 // Tester depuis un client`

Depuis un ordinateur ou un téléphone, rechercher :

```text
GREY-X-LAB
```

Se connecter avec :

```text
Mot de passe :
GreyX-Lab-2026
```

Le client devrait recevoir une adresse dans :

```text
192.168.50.100 - 192.168.50.200
```

Par exemple :

```text
IP       : 192.168.50.101
Gateway  : 192.168.50.1
DNS      : 192.168.50.1
```

Sur le serveur Linux :

```bash
ip neigh
```

permet d'observer les clients connus.

---

# `10 // Observer le DHCP`

Pour observer les échanges DHCP :

```bash
sudo tcpdump -i wlan1 -n port 67 or port 68
```

Lorsqu'un client se connecte, on peut observer la séquence :

```text
DHCPDISCOVER
      ↓
DHCPOFFER
      ↓
DHCPREQUEST
      ↓
DHCPACK
```

C'est une excellente manière de comprendre DHCP plutôt que de simplement utiliser `dnsmasq` comme une boîte noire.

---

# `11 // Observer le trafic`

Une fois le client connecté :

```bash
sudo tcpdump -i wlan1 -n
```

Et sur l'interface Internet :

```bash
sudo tcpdump -i eth0 -n
```

On peut alors observer la différence entre :

```text
wlan1
 ↓
192.168.50.x
 ↓
Linux
 ↓
eth0
 ↓
IP publique / réseau amont
```

Le NAT modifie notamment la représentation des connexions lorsqu'elles sortent par `eth0`.

---

# `12 // Vérifier le NAT`

Afficher les compteurs :

```bash
sudo iptables -t nat -L POSTROUTING -n -v
```

Si les clients utilisent Internet, les compteurs de la règle `MASQUERADE` devraient augmenter.

On peut également vérifier :

```bash
sudo iptables -L FORWARD -n -v
```

Cela permet de comprendre si les paquets traversent réellement la machine.

---

# `13 // Le chemin complet d'un paquet`

Lorsqu'un client demande une page Web :

```text
       CLIENT
    192.168.50.101
           │
           ▼
        wlan1
           │
           ▼
      Linux Server
           │
      iptables NAT
           │
           ▼
         eth0
           │
           ▼
       INTERNET
```

Et pour le retour :

```text
       INTERNET
           │
           ▼
         eth0
           │
      NAT / conntrack
           │
           ▼
        wlan1
           │
           ▼
       CLIENT
```

C'est exactement ce type de laboratoire qui permet de comprendre concrètement le fonctionnement du routage et du NAT.

---

# `14 // Dépannage`

Si le SSID n'apparaît pas :

```bash
sudo journalctl -u hostapd -n 50
```

Vérifier :

```bash
iw dev
```

et :

```bash
rfkill list
```

Si le DHCP ne fonctionne pas :

```bash
sudo journalctl -u dnsmasq -n 50
```

Vérifier également :

```bash
ip addr show wlan1
```

Si le client obtient une IP mais n'a pas Internet :

```bash
sysctl net.ipv4.ip_forward
```

Puis :

```bash
sudo iptables -t nat -L -n -v
```

et :

```bash
ip route
```

Les trois éléments à vérifier sont donc :

```text
DHCP
 ↓
ROUTAGE
 ↓
NAT
```

---

# `15 // La logique du laboratoire`

Au final, chaque composant possède une responsabilité précise :

```text
┌─────────────────────────────────────┐
│             GREY-X LAB              │
├─────────────────────────────────────┤
│                                     │
│ hostapd                             │
│     ↓                               │
│ crée le point d'accès Wi-Fi         │
│                                     │
│ dnsmasq                             │
│     ↓                               │
│ fournit DHCP + DNS                  │
│                                     │
│ Linux                               │
│     ↓                               │
│ route les paquets                   │
│                                     │
│ iptables                            │
│     ↓                               │
│ réalise le NAT                      │
│                                     │
│ eth0                                │
│     ↓                               │
│ accès Internet                      │
└─────────────────────────────────────┘
```

C'est cette séparation des rôles qui rend le laboratoire intéressant.

**hostapd ne fait pas le NAT.**

**dnsmasq ne fait pas le routage.**

**iptables ne crée pas le réseau Wi-Fi.**

Chaque composant fait une partie du travail.

---

## `16 // À retenir`

```text
AWUS1900
    ↓
hostapd
    ↓
Wi-Fi AP
    ↓
dnsmasq
    ↓
DHCP / DNS
    ↓
Linux routing
    ↓
iptables
    ↓
NAT
    ↓
Internet
```

Un simple adaptateur Wi-Fi peut donc devenir un véritable laboratoire réseau Linux.

L'intérêt n'est pas seulement de créer un point d'accès.

C'est de comprendre **ce qui se passe entre le moment où un client rejoint le Wi-Fi et celui où son paquet atteint Internet.**

> **GREY-X — Build the lab. Understand the network.**
