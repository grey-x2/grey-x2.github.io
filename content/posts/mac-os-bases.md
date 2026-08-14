+++
date = '2026-08-14T11:22:00+02:00'
draft = false
title = 'Les commandes macOS essentielles : maîtriser le terminal'
description = "Une introduction pratique aux commandes essentielles du terminal macOS pour naviguer, gérer les fichiers, analyser le système et diagnostiquer le réseau."
tags = ['macOS', 'Terminal', 'Unix', 'Réseau', 'Cybersécurité']
categories = ['macOS']
+++

> `GREY-X // macOS LAB`
>
> `MacBook:~ user$ whoami`

macOS possède une interface graphique complète, mais son terminal reste un outil extrêmement puissant.

Basé sur **Unix**, macOS partage de nombreuses commandes avec Linux : `ls`, `cd`, `cp`, `mv`, `grep`, `find`, `chmod`, `ssh`, `curl`, `ps` et bien d'autres.

Pour l'administration système, le développement ou la cybersécurité, savoir utiliser le terminal permet d'aller beaucoup plus loin que l'interface graphique.

---

## `01 // Se déplacer dans le système`

Afficher le répertoire courant :

```bash id="s1r4kv"
pwd
```

Lister les fichiers :

```bash id="q7m2cx"
ls
```

Afficher les fichiers cachés :

```bash id="v5k9rp"
ls -la
```

Changer de répertoire :

```bash id="j3x8wm"
cd /Users
```

Revenir au répertoire parent :

```bash id="n6q4zt"
cd ..
```

Revenir dans son dossier personnel :

```bash id="p8c2vy"
cd ~
```

Nettoyer le terminal :

```bash id="m4r7kx"
clear
```

---

## `02 // Créer et manipuler des fichiers`

Créer un répertoire :

```bash id="x9v3qp"
mkdir laboratoire
```

Créer un fichier :

```bash id="k5m8wd"
touch notes.txt
```

Copier un fichier :

```bash id="r2q7xc"
cp notes.txt backup.txt
```

Déplacer un fichier :

```bash id="t6p3vn"
mv backup.txt ~/Documents/
```

Renommer un fichier :

```bash id="w8k4zm"
mv notes.txt notes-old.txt
```

Supprimer un fichier :

```bash id="c3r9xq"
rm notes-old.txt
```

Supprimer un répertoire :

```bash id="h7m2vp"
rm -r laboratoire
```

⚠️ Contrairement à la corbeille graphique, `rm` supprime directement les fichiers.

---

## `03 // Lire les fichiers`

Afficher un fichier :

```bash id="p4x8kn"
cat notes.txt
```

Lire un fichier page par page :

```bash id="v7q3mc"
less notes.txt
```

Afficher le début :

```bash id="j9m5zr"
head notes.txt
```

Afficher les dernières lignes :

```bash id="s2k6wp"
tail notes.txt
```

Suivre un fichier en temps réel :

```bash id="d8r4xv"
tail -f application.log
```

---

## `04 // Rechercher`

Rechercher un fichier :

```bash id="m3q7kc"
find . -name "notes.txt"
```

Rechercher du texte :

```bash id="x6v2rp"
grep "error" application.log
```

Recherche récursive :

```bash id="n8k4wm"
grep -R "password" ./laboratoire/
```

Trouver un programme :

```bash id="q5r9xz"
which python3
```

Obtenir des informations sur une commande :

```bash id="c7m3vp"
man ls
```

---

## `05 // Permissions`

Afficher les permissions :

```bash id="w4k8qr"
ls -l
```

Modifier les permissions :

```bash id="p6x2mc"
chmod +x script.sh
```

Afficher le propriétaire :

```bash id="j3v9zn"
ls -l script.sh
```

Changer le propriétaire nécessite généralement des privilèges administrateur :

```bash id="r7m4kp"
sudo chown user:staff script.sh
```

Les permissions Unix sont essentielles pour comprendre la sécurité d'un système macOS.

---

## `06 // Identifier le système`

Afficher l'utilisateur actuel :

```bash id="x8q2mv"
whoami
```

Afficher les informations système :

```bash id="k4r7wp"
uname -a
```

Afficher la version macOS :

```bash id="m9v3xc"
sw_vers
```

Afficher le nom de la machine :

```bash id="p5k8zr"
scutil --get ComputerName
```

Afficher l'architecture :

```bash id="d2q6vn"
uname -m
```

---

## `07 // Les processus`

Afficher les processus :

```bash id="r8m3kc"
ps aux
```

Rechercher un processus :

```bash id="v5q7xp"
ps aux | grep Safari
```

Surveiller les processus :

```bash id="j4n9wm"
top
```

Rechercher le PID d'un processus :

```bash id="c6r2zk"
pgrep Safari
```

Arrêter un processus :

```bash id="x3m8vp"
kill PID
```

Forcer l'arrêt :

```bash id="n7q4kr"
kill -9 PID
```

---

## `08 // Analyser le réseau`

Afficher les interfaces réseau :

```bash id="p9v3mc"
ifconfig
```

Afficher les interfaces avec leurs adresses :

```bash id="w6k2xr"
ifconfig en0
```

Afficher la route par défaut :

```bash id="m4q8zn"
route -n get default
```

Tester la connectivité :

```bash id="j7r3vp"
ping 8.8.8.8
```

Résoudre un nom DNS :

```bash id="c5x9km"
nslookup example.com
```

Ou :

```bash id="v2m6qr"
dig example.com
```

Afficher les connexions réseau :

```bash id="n8k4wp"
netstat -an
```

---

## `09 // Identifier les ports et services`

Pour rechercher les sockets réseau utilisés par les processus :

```bash id="q3r7xm"
lsof -i
```

Afficher les connexions TCP :

```bash id="p6v2kc"
lsof -iTCP -sTCP:ESTABLISHED
```

Rechercher un port particulier :

```bash id="m9x4zr"
lsof -i :443
```

Cette commande permet notamment d'associer un port à un processus.

Par exemple :

```text id="k2v8wp"
PORT
 ↓
PROCESS
 ↓
PID
```

C'est un réflexe très utile lors d'une analyse système.

---

## `10 // DNS et configuration réseau`

Afficher les informations DNS :

```bash id="r4m7xc"
scutil --dns
```

Afficher les interfaces réseau :

```bash id="j8q3vp"
networksetup -listallhardwareports
```

Afficher la configuration d'une interface :

```bash id="x5k9mr"
networksetup -getinfo Wi-Fi
```

Afficher les réseaux Wi-Fi disponibles :

```bash id="v3p6zn"
networksetup -listpreferredwirelessnetworks en0
```

Selon la version de macOS et l'interface utilisée, le nom de l'interface Wi-Fi peut être différent.

---

## `11 // Gestion des utilisateurs`

Afficher les informations de l'utilisateur courant :

```bash id="c7r2km"
id
```

Afficher les utilisateurs locaux :

```bash id="m5x8vp"
dscl . list /Users
```

Afficher les groupes :

```bash id="q9k3zr"
dscl . list /Groups
```

Exécuter une commande avec les privilèges administrateur :

```bash id="w4m7xc"
sudo commande
```

---

## `12 // SSH`

macOS permet également d'utiliser SSH directement depuis le terminal.

Se connecter à une machine distante :

```bash id="p3v8kn"
ssh user@192.168.1.10
```

Copier un fichier vers une machine distante :

```bash id="j6q2mr"
scp fichier.txt user@192.168.1.10:/tmp/
```

Récupérer un fichier :

```bash id="x8r4vp"
scp user@192.168.1.10:/tmp/fichier.txt .
```

SSH est particulièrement utile pour administrer des serveurs et des environnements de laboratoire.

---

## `13 // Télécharger et tester avec curl`

`curl` est disponible nativement sur macOS.

Tester une URL :

```bash id="n5k7zc"
curl https://example.com
```

Afficher uniquement les en-têtes HTTP :

```bash id="r3m9qx"
curl -I https://example.com
```

Afficher les informations détaillées de la connexion :

```bash id="v6p2kw"
curl -v https://example.com
```

`curl` est très utile pour comprendre les communications HTTP et tester des services dans un environnement autorisé.

---

## `14 // Gestion des logiciels avec Homebrew`

Sur de nombreux Mac utilisés pour le développement, **Homebrew** est un gestionnaire de paquets très pratique.

Rechercher un paquet :

```bash id="k8x3mr"
brew search nmap
```

Installer un paquet :

```bash id="q4v7zp"
brew install nmap
```

Mettre à jour Homebrew :

```bash id="m2r9kc"
brew update
```

Mettre à jour les logiciels :

```bash id="j5x8vn"
brew upgrade
```

---

## `15 // Les commandes à retenir`

Pour commencer, maîtrisez surtout :

```text id="p7m3xr"
pwd             → répertoire courant
ls              → fichiers
cd              → navigation
mkdir           → créer un dossier
touch           → créer un fichier
cp              → copier
mv              → déplacer / renommer
rm              → supprimer

cat             → afficher
less            → lire
grep            → rechercher
find            → rechercher des fichiers
man             → documentation

whoami          → utilisateur
uname           → informations système
sw_vers         → version macOS

ps              → processus
top             → surveillance
kill            → arrêter un processus

ifconfig        → interfaces réseau
ping            → connectivité
route           → routage
netstat         → connexions
lsof            → fichiers / ports / processus
dig             → DNS

ssh             → connexion distante
scp             → transfert de fichiers
curl            → HTTP / réseau
sudo            → privilèges administrateur
```

---

## `16 // macOS, Unix et cybersécurité`

L'un des avantages de macOS pour quelqu'un qui travaille avec plusieurs environnements est la proximité avec l'écosystème Unix.

Une partie importante des commandes utilisées sous Linux existe également sous macOS.

On retrouve donc rapidement une logique commune :

```text id="z6v2qa"
Linux
   │
   ├── ls
   ├── grep
   ├── find
   ├── ssh
   ├── curl
   └── chmod
          │
          ▼
       macOS
```

Mais macOS possède également ses propres outils et mécanismes d'administration.

L'objectif n'est donc pas de mémoriser une liste de commandes.

Il faut comprendre **ce que l'on cherche à observer** et choisir ensuite la commande adaptée.

```text id="h3m8vp"
┌──────────────────────────────────────┐
│              GREY-X                  │
│                                      │
│   macOS → UNIX → NETWORK → SECURITY  │
│                                      │
│       MASTER THE TERMINAL            │
└──────────────────────────────────────┘
```

**Le terminal permet de voir ce que l'interface graphique ne montre pas toujours.**
