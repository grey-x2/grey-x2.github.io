+++
date = '2026-08-14T11:23:00+02:00'
draft = false
title = 'Prise en main d’openSUSE : les bases à connaître'
description = "Découverte pratique d’openSUSE, de son terminal, de zypper, de la gestion des services et des outils système essentiels."
tags = ['openSUSE', 'Linux', 'zypper', 'Administration', 'Cybersécurité']
categories = ['Linux']
+++

> `GREY-X // OPENSUSE LAB`
>
> `user@opensuse:~> systemctl status`

openSUSE est une distribution Linux particulièrement intéressante pour apprendre l'administration système.

Comme d'autres distributions Linux, elle utilise le noyau Linux, systemd et les outils Unix classiques. Mais elle possède également ses propres outils et méthodes, notamment **Zypper**, **YaST** et **RPM**.

Pour quelqu'un qui vient de Debian, Ubuntu ou Kali Linux, la logique générale reste familière.

La principale différence apparaît surtout dans la gestion des paquets et certains outils d'administration.

---

## `01 // Identifier le système`

Afficher les informations du système :

```bash id="g7m2xp"
cat /etc/os-release
```

Afficher la version du noyau :

```bash id="k4v8zr"
uname -a
```

Afficher le nom de la machine :

```bash id="p9x3mc"
hostname
```

Afficher l'utilisateur courant :

```bash id="r6q2vn"
whoami
```

Afficher les informations de l'utilisateur :

```bash id="w3m8kp"
id
```

---

## `02 // Le gestionnaire de paquets : Zypper`

Sur openSUSE, **zypper** est l'un des outils principaux pour gérer les paquets.

Mettre à jour les dépôts :

```bash id="x5k7mr"
sudo zypper refresh
```

Mettre à jour le système :

```bash id="q8v3pc"
sudo zypper update
```

Installer un paquet :

```bash id="m2r9zx"
sudo zypper install nmap
```

Supprimer un paquet :

```bash id="j6k4wp"
sudo zypper remove nmap
```

Rechercher un paquet :

```bash id="c3v8qm"
zypper search nmap
```

Afficher les informations d'un paquet :

```bash id="n7x2kr"
zypper info nmap
```

Afficher les paquets installés :

```bash id="v4m9zp"
zypper packages --installed-only
```

---

## `03 // Zypper et équivalents Debian`

Si tu viens de Debian ou Kali, certaines commandes changent :

```text id="r5k8xm"
Debian / Kali              openSUSE

apt update            →    zypper refresh
apt upgrade           →    zypper update
apt install nmap      →    zypper install nmap
apt remove nmap       →    zypper remove nmap
apt search nmap       →    zypper search nmap
```

La logique reste cependant la même :

```text id="m8q3vp"
REPOSITORY
     ↓
PACKAGE
     ↓
INSTALL
     ↓
UPDATE
     ↓
REMOVE
```

---

## `04 // Naviguer dans le système`

Les commandes Unix classiques fonctionnent toujours.

Afficher le répertoire courant :

```bash id="x4n7kc"
pwd
```

Lister les fichiers :

```bash id="p6m2zr"
ls -la
```

Changer de répertoire :

```bash id="j8v5qx"
cd /etc
```

Revenir en arrière :

```bash id="c3r9wp"
cd ..
```

Créer un répertoire :

```bash id="m7k4zn"
mkdir laboratoire
```

Créer un fichier :

```bash id="q2x8vp"
touch notes.txt
```

---

## `05 // Gestion des fichiers`

Copier :

```bash id="v9m3kc"
cp notes.txt backup.txt
```

Déplacer :

```bash id="r5q7xm"
mv backup.txt /tmp/
```

Supprimer :

```bash id="k8p2vz"
rm notes.txt
```

Lire un fichier :

```bash id="x6m4qr"
cat notes.txt
```

Lire un fichier page par page :

```bash id="n3v8wp"
less notes.txt
```

Rechercher du texte :

```bash id="j7k2mc"
grep "error" /var/log/messages
```

---

## `06 // Les services avec systemd`

openSUSE utilise **systemd** pour gérer les services.

Afficher l'état d'un service :

```bash id="p4r9xz"
sudo systemctl status sshd
```

Démarrer un service :

```bash id="m8q3vk"
sudo systemctl start sshd
```

Arrêter un service :

```bash id="x5n7cr"
sudo systemctl stop sshd
```

Redémarrer :

```bash id="q2k6wp"
sudo systemctl restart sshd
```

Activer au démarrage :

```bash id="v9m4zr"
sudo systemctl enable sshd
```

Désactiver au démarrage :

```bash id="j3x8kc"
sudo systemctl disable sshd
```

Afficher les services actifs :

```bash id="r6p2vm"
systemctl list-units --type=service --state=running
```

---

## `07 // Examiner les logs`

Les journaux système peuvent être consultés avec `journalctl`.

Afficher les logs :

```bash id="k4m8xp"
sudo journalctl
```

Afficher les dernières lignes :

```bash id="c7v3qr"
sudo journalctl -n 50
```

Suivre les logs en temps réel :

```bash id="m2x9zk"
sudo journalctl -f
```

Afficher les logs d'un service :

```bash id="p8r4vn"
sudo journalctl -u sshd
```

Afficher les logs depuis le dernier démarrage :

```bash id="x6k3mq"
sudo journalctl -b
```

Pour l'analyse système, les logs sont souvent aussi importants que les commandes d'administration.

---

## `08 // Réseau`

Afficher les interfaces :

```bash id="v5q8kc"
ip addr
```

Afficher les routes :

```bash id="r3m7xp"
ip route
```

Tester la connectivité :

```bash id="j9x2vn"
ping 8.8.8.8
```

Résoudre un nom DNS :

```bash id="k6p4zr"
dig example.com
```

Afficher les sockets :

```bash id="m8v3qc"
ss -tulnp
```

Afficher les voisins réseau :

```bash id="x2r7km"
ip neigh
```

---

## `09 // Configuration réseau avec NetworkManager`

Sur une installation utilisant NetworkManager, on peut utiliser `nmcli`.

Afficher les connexions :

```bash id="q4k9vp"
nmcli connection show
```

Afficher les interfaces :

```bash id="v7m2xr"
nmcli device status
```

Afficher les détails d'une connexion :

```bash id="j5p8kc"
nmcli connection show "Wired connection 1"
```

Activer une connexion :

```bash id="m3x6zr"
nmcli connection up "Wired connection 1"
```

Désactiver :

```bash id="p9r4vq"
nmcli connection down "Wired connection 1"
```

Cela devient particulièrement intéressant lorsqu'on travaille sur des laboratoires réseau.

---

## `10 // Utilisateurs et privilèges`

Afficher l'utilisateur courant :

```bash id="x8m3kp"
whoami
```

Créer un utilisateur :

```bash id="q5v7zr"
sudo useradd -m alice
```

Définir son mot de passe :

```bash id="j2r9xc"
sudo passwd alice
```

Afficher les groupes :

```bash id="m6k4vp"
groups alice
```

Changer temporairement d'utilisateur :

```bash id="p3x8qm"
su - alice
```

Exécuter une commande avec les privilèges administrateur :

```bash id="v9r2kc"
sudo commande
```

---

## `11 // Permissions Linux`

Afficher les permissions :

```bash id="k7m3xp"
ls -l
```

Ajouter le droit d'exécution :

```bash id="x4q8vn"
chmod +x script.sh
```

Retirer le droit d'écriture :

```bash id="j6p2zr"
chmod -w fichier.txt
```

Changer le propriétaire :

```bash id="m8v5kc"
sudo chown alice:users fichier.txt
```

Les permissions restent fondamentales, quelle que soit la distribution Linux.

---

## `12 // Processus`

Afficher les processus :

```bash id="q3r7xm"
ps aux
```

Surveiller les processus :

```bash id="v5k9cp"
top
```

Rechercher un processus :

```bash id="m2x8zr"
ps aux | grep ssh
```

Arrêter un processus :

```bash id="j7q4vn"
kill PID
```

Pour une administration plus avancée, `htop` peut également être installé avec Zypper.

---

## `13 // YaST`

L'une des particularités d'openSUSE est **YaST — Yet another Setup Tool**.

Lancer YaST :

```bash id="x8m4kp"
sudo yast
```

YaST fournit une interface d'administration permettant notamment de gérer :

* le réseau ;
* les utilisateurs ;
* les services ;
* les logiciels ;
* le stockage ;
* certains paramètres système.

Il existe également des modules YaST utilisables directement depuis le terminal.

L'intérêt de YaST est de centraliser de nombreuses opérations d'administration dans un même environnement.

---

## `14 // Comprendre la philosophie openSUSE`

openSUSE propose plusieurs outils qui se complètent :

```text id="r6v2km"
             openSUSE
                │
       ┌────────┼────────┐
       │        │        │
     Zypper    YaST    systemd
       │        │        │
    Paquets  Config   Services
       │        │        │
       └────────┼────────┘
                │
             Linux
```

Mais les fondamentaux restent les mêmes :

```text id="p8x3zr"
FILES
  ↓
PROCESSES
  ↓
SERVICES
  ↓
NETWORK
  ↓
PERMISSIONS
  ↓
LOGS
```

C'est cette compréhension qui permet ensuite de travailler efficacement sur une machine Linux.

---

## `15 // Les commandes à retenir`

```text id="v4m7qx"
zypper              → gestion des paquets
yast                → administration système

pwd                 → répertoire courant
ls                  → fichiers
cd                  → navigation
cp                  → copier
mv                  → déplacer
rm                  → supprimer
grep                → rechercher

systemctl           → services
journalctl          → logs

ip                  → réseau
ss                  → sockets
nmcli               → NetworkManager

ps                  → processus
top                 → processus en temps réel

whoami              → utilisateur
sudo                → privilèges
passwd              → mot de passe

chmod               → permissions
chown               → propriétaire
```

---

## `16 // Pourquoi apprendre openSUSE ?`

Connaître plusieurs distributions Linux permet de ne pas dépendre d'un seul environnement.

Les commandes Unix fondamentales restent largement communes, mais les outils d'administration et de gestion des paquets peuvent varier.

Apprendre openSUSE permet donc de développer un réflexe important :

> **Ne pas mémoriser uniquement une commande. Comprendre le système derrière la commande.**

Que l'on travaille sur Debian, Kali, Ubuntu, Fedora ou openSUSE, les concepts fondamentaux restent présents.

```text id="z5r8mc"
┌──────────────────────────────────────┐
│              GREY-X                  │
│                                      │
│       OPENSUSE // LINUX LAB          │
│                                      │
│   LEARN → ADMIN → ANALYZE → SECURE   │
└──────────────────────────────────────┘
```

**Une distribution change. Les fondamentaux restent.**
