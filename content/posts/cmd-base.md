+++
date = '2026-08-14T11:18:00+02:00'
draft = false
title = 'Les commandes CMD Windows essentielles : les bases du terminal'
description = "Une introduction pratique aux commandes CMD indispensables pour naviguer dans Windows, gérer les fichiers, diagnostiquer le système et analyser le réseau."
tags = ['Windows', 'CMD', 'Command Line', 'Réseau', 'Cybersécurité']
categories = ['Windows']
+++

> `GREY-X // WINDOWS LAB`
>
> `C:\> ACCESSING COMMAND LINE...`

Windows possède également son propre environnement en ligne de commande.

Le **Command Prompt**, plus connu sous le nom de **CMD**, permet d'effectuer de nombreuses opérations directement depuis le terminal : naviguer dans les fichiers, gérer des processus, diagnostiquer le réseau, vérifier les connexions et automatiser certaines tâches.

Pour quelqu'un qui s'intéresse à l'administration système ou à la cybersécurité, connaître les commandes essentielles de Windows est aussi important que connaître celles de Linux.

---

## `01 // Se déplacer dans Windows`

Afficher le répertoire courant :

```cmd
cd
```

Changer de répertoire :

```cmd
cd C:\Users
```

Revenir au répertoire parent :

```cmd
cd ..
```

Changer de lecteur :

```cmd
D:
```

Afficher le contenu du répertoire :

```cmd
dir
```

Afficher également les fichiers cachés :

```cmd
dir /a
```

Nettoyer l'écran :

```cmd
cls
```

---

## `02 // Créer et manipuler des fichiers`

Créer un répertoire :

```cmd
mkdir laboratoire
```

Ou :

```cmd
md laboratoire
```

Créer un fichier :

```cmd
type nul > notes.txt
```

Copier un fichier :

```cmd
copy notes.txt backup.txt
```

Déplacer un fichier :

```cmd
move notes.txt C:\Temp\
```

Renommer un fichier :

```cmd
ren backup.txt sauvegarde.txt
```

Supprimer un fichier :

```cmd
del sauvegarde.txt
```

Supprimer un répertoire :

```cmd
rmdir laboratoire
```

Pour supprimer un répertoire contenant des fichiers :

```cmd
rmdir /s laboratoire
```

⚠️ Les commandes de suppression doivent être utilisées avec attention.

---

## `03 // Lire des fichiers`

Afficher le contenu d'un fichier texte :

```cmd
type notes.txt
```

Afficher un fichier page par page :

```cmd
more notes.txt
```

Rechercher une chaîne de caractères :

```cmd
findstr "password" notes.txt
```

Recherche récursive :

```cmd
findstr /s /i "error" *.log
```

`findstr` est particulièrement pratique pour rechercher rapidement des informations dans plusieurs fichiers.

---

## `04 // Identifier la machine`

Afficher le nom de l'ordinateur :

```cmd
hostname
```

Afficher la version de Windows :

```cmd
ver
```

Obtenir davantage d'informations système :

```cmd
systeminfo
```

Afficher l'utilisateur courant :

```cmd
whoami
```

Afficher les utilisateurs locaux :

```cmd
net user
```

Afficher les groupes locaux :

```cmd
net localgroup
```

Ces commandes sont utiles lorsqu'on doit rapidement comprendre l'environnement dans lequel on travaille.

---

## `05 // Comprendre le réseau`

Afficher la configuration réseau :

```cmd
ipconfig
```

Afficher davantage d'informations :

```cmd
ipconfig /all
```

Vider le cache DNS :

```cmd
ipconfig /flushdns
```

Renouveler une adresse DHCP :

```cmd
ipconfig /release
ipconfig /renew
```

Tester la connectivité :

```cmd
ping 8.8.8.8
```

Tester la résolution DNS :

```cmd
nslookup example.com
```

Afficher la table ARP :

```cmd
arp -a
```

Afficher la table de routage :

```cmd
route print
```

---

## `06 // Analyser le chemin réseau`

Pour comprendre le chemin utilisé vers une destination :

```cmd
tracert 8.8.8.8
```

Pour analyser les pertes et latences sur les différents sauts :

```cmd
pathping 8.8.8.8
```

Ces commandes sont très utiles lors du diagnostic d'un problème réseau.

---

## `07 // Observer les connexions`

Afficher les connexions réseau actives :

```cmd
netstat -ano
```

Afficher uniquement les ports en écoute :

```cmd
netstat -ano | findstr LISTENING
```

Le paramètre `-o` permet notamment d'obtenir le **PID** du processus associé à une connexion.

On peut ensuite rechercher le processus correspondant :

```cmd
tasklist | findstr 1234
```

Cette combinaison est particulièrement utile :

```text
netstat → PID → tasklist → processus
```

Elle permet de passer d'un port réseau à l'application qui l'utilise.

---

## `08 // Gérer les processus`

Afficher les processus :

```cmd
tasklist
```

Rechercher un processus :

```cmd
tasklist | findstr chrome
```

Arrêter un processus par son PID :

```cmd
taskkill /PID 1234
```

Forcer l'arrêt :

```cmd
taskkill /F /PID 1234
```

Arrêter un processus par son nom :

```cmd
taskkill /IM notepad.exe
```

---

## `09 // Services Windows`

Afficher les services :

```cmd
sc query
```

Rechercher un service :

```cmd
sc query | findstr "Running"
```

Interroger un service spécifique :

```cmd
sc query wuauserv
```

Arrêter un service :

```cmd
sc stop wuauserv
```

Démarrer un service :

```cmd
sc start wuauserv
```

Certaines opérations nécessitent un terminal lancé avec des privilèges administrateur.

---

## `10 // Gestion des utilisateurs`

Afficher les utilisateurs :

```cmd
net user
```

Afficher les informations d'un utilisateur :

```cmd
net user utilisateur
```

Créer un utilisateur :

```cmd
net user testuser /add
```

Supprimer un utilisateur :

```cmd
net user testuser /delete
```

Afficher les groupes locaux :

```cmd
net localgroup
```

Ces commandes sont particulièrement utiles dans un laboratoire Windows.

---

## `11 // Permissions et fichiers`

Afficher les permissions d'un fichier ou répertoire :

```cmd
icacls fichier.txt
```

Modifier les permissions :

```cmd
icacls fichier.txt /grant utilisateur:R
```

Dans un environnement de test, `icacls` permet de comprendre concrètement le modèle de permissions NTFS.

Il est important de bien comprendre les droits avant de les modifier sur une machine réelle.

---

## `12 // Diagnostiquer le système`

Vérifier les fichiers système Windows :

```cmd
sfc /scannow
```

Vérifier l'image Windows :

```cmd
DISM /Online /Cleanup-Image /CheckHealth
```

Afficher les variables d'environnement :

```cmd
set
```

Afficher une variable spécifique :

```cmd
echo %PATH%
```

Afficher l'heure :

```cmd
time
```

Afficher la date :

```cmd
date
```

---

## `13 // Redirections et commandes combinées`

Comme sous Linux, CMD permet de rediriger la sortie d'une commande.

Par exemple :

```cmd
ipconfig /all > network.txt
```

Ajouter à un fichier existant :

```cmd
ipconfig /all >> network.txt
```

Utiliser un pipe :

```cmd
tasklist | findstr chrome
```

Ou :

```cmd
netstat -ano | findstr LISTENING
```

Cette capacité à combiner les commandes permet de transformer CMD en véritable outil d'analyse.

---

## `14 // Les commandes à retenir`

Pour commencer, concentrez-vous sur celles-ci :

```text
dir             → lister les fichiers
cd              → changer de répertoire
mkdir           → créer un dossier
copy            → copier
move            → déplacer
ren             → renommer
del             → supprimer
type            → afficher un fichier
findstr         → rechercher du texte

whoami          → utilisateur courant
hostname        → nom de la machine
systeminfo      → informations système
tasklist        → processus
taskkill        → arrêter un processus
sc              → gérer les services

ipconfig        → configuration réseau
ping            → tester la connectivité
nslookup        → DNS
arp             → table ARP
route           → routage
tracert         → traceroute Windows
netstat         → connexions réseau

net user        → utilisateurs
net localgroup  → groupes
icacls          → permissions
```

---

## `15 // CMD avant les outils de sécurité`

En cybersécurité, on peut rapidement passer à des outils spécialisés.

Mais avant d'utiliser des outils avancés, il est important de savoir observer le système lui-même.

Avec quelques commandes CMD, on peut déjà répondre à des questions essentielles :

```text
Qui suis-je ?
       ↓
Quelle est cette machine ?
       ↓
Quelle est sa configuration réseau ?
       ↓
Quels processus tournent ?
       ↓
Quels ports sont ouverts ?
       ↓
Quels services fonctionnent ?
       ↓
Quels utilisateurs existent ?
       ↓
Quels droits sont appliqués ?
```

C'est cette méthode d'observation qui constitue une bonne base pour l'analyse d'un environnement Windows.

```text
┌──────────────────────────────────────┐
│              GREY-X                  │
│                                      │
│  C:\> ENUMERATE                      │
│  C:\> ANALYZE                        │
│  C:\> UNDERSTAND                     │
│                                      │
│       WINDOWS COMMAND LINE            │
└──────────────────────────────────────┘
```

**Avant d'utiliser des outils complexes, apprenez à interroger directement le système.**
