+++
date = '2026-08-14T11:24:00+02:00'
draft = false
title = 'Prise en main d’Arch Linux : les bases à connaître'
description = "Une introduction pratique à Arch Linux : terminal, pacman, systemd, réseau, utilisateurs, permissions et maintenance du système."
tags = ['Arch Linux', 'Linux', 'Pacman', 'Administration', 'Cybersécurité']
categories = ['Linux']
+++

> `GREY-X // ARCH LAB`
>
> `user@archlinux ~ $`

Arch Linux est une distribution Linux connue pour sa simplicité, sa flexibilité et son approche minimaliste.

Contrairement à certaines distributions qui installent de nombreux composants par défaut, Arch permet de construire son environnement selon ses besoins.

Cette philosophie en fait également un excellent environnement pour apprendre à comprendre Linux.

Mais Arch demande une chose essentielle :

**savoir ce que l'on configure.**

---

## `01 // Identifier le système`

Afficher les informations de la distribution :

```bash
cat /etc/os-release
```

Afficher le noyau :

```bash
uname -a
```

Afficher l'utilisateur :

```bash
whoami
```

Afficher le nom de la machine :

```bash
hostname
```

Afficher l'architecture :

```bash
uname -m
```

---

## `02 // Naviguer dans le système`

Les commandes Linux classiques fonctionnent également sous Arch.

```bash
pwd
```

Afficher les fichiers :

```bash
ls -la
```

Changer de répertoire :

```bash
cd /etc
```

Revenir au répertoire parent :

```bash
cd ..
```

Créer un dossier :

```bash
mkdir laboratoire
```

Créer un fichier :

```bash
touch notes.txt
```

---

## `03 // Pacman : le gestionnaire de paquets`

La commande centrale pour gérer les paquets sur Arch Linux est :

```bash
pacman
```

Synchroniser les bases de données et mettre à jour le système :

```bash
sudo pacman -Syu
```

Installer un paquet :

```bash
sudo pacman -S nmap
```

Supprimer un paquet :

```bash
sudo pacman -R nmap
```

Rechercher un paquet dans les dépôts :

```bash
pacman -Ss nmap
```

Afficher les informations d'un paquet :

```bash
pacman -Si nmap
```

Rechercher les paquets installés :

```bash
pacman -Q
```

Rechercher un paquet installé :

```bash
pacman -Qs nmap
```

Afficher les fichiers appartenant à un paquet :

```bash
pacman -Ql nmap
```

---

## `04 // Pacman : comprendre la logique`

Si tu viens de Debian ou openSUSE, voici les équivalences principales :

```text
Debian / Kali          Arch

apt update             pacman -Sy
apt upgrade            pacman -Su
apt update + upgrade   pacman -Syu
apt install nmap       pacman -S nmap
apt remove nmap        pacman -R nmap
apt search nmap        pacman -Ss nmap
```

La commande la plus importante à retenir pour maintenir Arch à jour reste :

```bash
sudo pacman -Syu
```

---

## `05 // AUR`

Arch possède également un écosystème très important appelé **AUR — Arch User Repository**.

L'AUR contient des recettes de paquets maintenues par la communauté.

Un outil comme `yay` peut faciliter l'utilisation de l'AUR lorsqu'il est installé.

Rechercher un paquet :

```bash
yay -Ss nom-paquet
```

Installer :

```bash
yay -S nom-paquet
```

⚠️ L'AUR n'est pas équivalent aux dépôts officiels. Avant d'installer un paquet provenant de l'AUR, il est important de vérifier son contenu et sa provenance.

---

## `06 // Systemd`

Arch utilise `systemd` pour gérer les services.

Vérifier un service :

```bash
systemctl status sshd
```

Démarrer :

```bash
sudo systemctl start sshd
```

Arrêter :

```bash
sudo systemctl stop sshd
```

Redémarrer :

```bash
sudo systemctl restart sshd
```

Activer au démarrage :

```bash
sudo systemctl enable sshd
```

Activer et démarrer immédiatement :

```bash
sudo systemctl enable --now sshd
```

Afficher les services actifs :

```bash
systemctl list-units --type=service --state=running
```

---

## `07 // Les logs`

Arch utilise également `journalctl` pour consulter les journaux de systemd.

Afficher les logs :

```bash
sudo journalctl
```

Afficher les dernières lignes :

```bash
sudo journalctl -n 50
```

Suivre les logs en temps réel :

```bash
sudo journalctl -f
```

Afficher les logs d'un service :

```bash
sudo journalctl -u sshd
```

Afficher les logs depuis le dernier démarrage :

```bash
sudo journalctl -b
```

Pour diagnostiquer un problème, savoir lire les logs est souvent plus important que de lancer immédiatement un outil.

---

## `08 // Réseau`

Afficher les interfaces :

```bash
ip addr
```

Afficher les routes :

```bash
ip route
```

Tester la connectivité :

```bash
ping 8.8.8.8
```

Tester le DNS :

```bash
dig example.com
```

Afficher les sockets :

```bash
ss -tulnp
```

Afficher les voisins réseau :

```bash
ip neigh
```

Ces commandes constituent une excellente base pour l'administration réseau et le pentesting en laboratoire.

---

## `09 // NetworkManager`

Si NetworkManager est utilisé, `nmcli` permet de gérer les connexions.

Afficher les interfaces :

```bash
nmcli device status
```

Afficher les connexions :

```bash
nmcli connection show
```

Afficher les détails :

```bash
nmcli device show
```

Activer une connexion :

```bash
nmcli connection up "nom-connexion"
```

Désactiver :

```bash
nmcli connection down "nom-connexion"
```

---

## `10 // Processus`

Afficher les processus :

```bash
ps aux
```

Surveiller le système :

```bash
top
```

Si installé :

```bash
htop
```

Rechercher un processus :

```bash
ps aux | grep ssh
```

Trouver un PID :

```bash
pgrep sshd
```

Arrêter un processus :

```bash
kill PID
```

---

## `11 // Utilisateurs`

Afficher l'utilisateur actuel :

```bash
whoami
```

Afficher ses groupes :

```bash
groups
```

Créer un utilisateur :

```bash
sudo useradd -m alice
```

Définir son mot de passe :

```bash
sudo passwd alice
```

Afficher les utilisateurs locaux :

```bash
cat /etc/passwd
```

Afficher les groupes :

```bash
cat /etc/group
```

---

## `12 // Permissions`

Afficher les permissions :

```bash
ls -l
```

Modifier les permissions :

```bash
chmod +x script.sh
```

Modifier le propriétaire :

```bash
sudo chown alice:alice fichier.txt
```

Les permissions sont fondamentales dans l'administration Linux.

Comprendre `r`, `w` et `x` permet déjà de mieux comprendre les mécanismes de contrôle d'accès.

```text
r → read
w → write
x → execute
```

---

## `13 // Rechercher des fichiers`

Rechercher un fichier :

```bash
find /home -name "notes.txt"
```

Rechercher du contenu :

```bash
grep -R "error" /var/log/
```

Rechercher une commande :

```bash
which nmap
```

Afficher le chemin d'une commande :

```bash
command -v nmap
```

---

## `14 // Arch et la philosophie KISS`

Arch est souvent associé à la philosophie :

**KISS — Keep It Simple, Stupid.**

L'idée n'est pas simplement d'avoir un système minimal.

C'est surtout de comprendre les composants qui constituent le système et de pouvoir les contrôler directement.

```text
              ARCH LINUX
                   │
        ┌──────────┼──────────┐
        │          │          │
      pacman     systemd     Linux
        │          │          │
     packages    services   kernel
        │          │          │
        └──────────┼──────────┘
                   │
               USER SPACE
```

Cette approche est particulièrement intéressante pour apprendre l'administration Linux.

---

## `15 // Arch pour apprendre la cybersécurité`

Arch n'est pas une distribution dédiée au pentesting comme Kali Linux.

Et c'est justement une distinction importante.

Kali fournit un environnement spécialisé avec de nombreux outils de sécurité.

Arch, lui, permet de construire son environnement et d'installer uniquement les composants nécessaires.

On peut donc l'utiliser comme laboratoire pour apprendre :

* Linux ;
* réseau ;
* administration système ;
* scripting ;
* services ;
* sécurité ;
* compilation ;
* gestion des paquets.

L'objectif n'est pas d'avoir le plus grand nombre d'outils.

**L'objectif est de comprendre ce que fait chaque outil.**

---

## `16 // Les commandes à retenir`

```text
pacman              → gestion des paquets
yay                 → gestion AUR

pwd                 → répertoire courant
ls                  → fichiers
cd                  → navigation
mkdir               → créer
cp                  → copier
mv                  → déplacer
rm                  → supprimer

grep                → rechercher
find                → rechercher des fichiers
cat                 → lire

systemctl           → services
journalctl          → logs

ip                  → réseau
ss                  → sockets
nmcli               → NetworkManager

ps                  → processus
top                 → surveillance

whoami              → utilisateur
groups              → groupes
sudo                → privilèges

chmod               → permissions
chown               → propriétaire
```

---

## `17 // La vraie compétence`

Apprendre Arch Linux ne consiste pas à mémoriser des commandes spécifiques.

Il faut comprendre les relations entre les composants :

```text
PACKAGE
   ↓
SERVICE
   ↓
PROCESS
   ↓
NETWORK
   ↓
LOGS
   ↓
PERMISSIONS
```

Une fois cette logique comprise, passer d'Arch à Debian, Ubuntu, openSUSE ou Fedora devient beaucoup plus simple.

```text
┌──────────────────────────────────────┐
│              GREY-X                  │
│                                      │
│       ARCH LINUX // LAB              │
│                                      │
│    LEARN → BUILD → TEST → SECURE     │
└──────────────────────────────────────┘
```

**Une bonne maîtrise de Linux commence lorsque l'on comprend ce que l'on configure, et pas seulement les commandes que l'on tape.**
