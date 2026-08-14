+++
date = '2026-08-14T11:16:00+02:00'
draft = false
title = 'Les commandes Linux essentielles : les bases à maîtriser'
description = "Une introduction pratique aux commandes Linux indispensables pour naviguer, manipuler des fichiers, gérer les processus et administrer un système."
tags = ['Linux', 'Terminal', 'Command Line', 'Pentesting', 'Cybersécurité']
categories = ['Linux']
+++

> `GREY-X // LINUX LAB`
>
> `LEARN THE TERMINAL`

Lorsqu'on commence la cybersécurité, on rencontre rapidement Linux.

Kali Linux, Debian, Ubuntu, Arch et de nombreuses autres distributions sont utilisées dans les environnements de développement, d'administration système, de sécurité offensive et de sécurité défensive.

Mais avant d'apprendre des outils complexes, il faut maîtriser quelque chose de beaucoup plus fondamental :

**le terminal.**

La ligne de commande permet de contrôler un système avec précision, d'automatiser des tâches et surtout de comprendre ce qui se passe réellement sur une machine.

---

## `01 // Se déplacer dans le système`

La première chose à maîtriser est la navigation dans l'arborescence Linux.

Afficher le répertoire courant :

```bash id="j9r3p4"
pwd
```

Lister les fichiers :

```bash id="b4m7xa"
ls
```

Afficher davantage d'informations :

```bash id="zq8h1w"
ls -la
```

Changer de répertoire :

```bash id="3k7f2c"
cd /var/log
```

Revenir au répertoire précédent :

```bash id="x6r2np"
cd ..
```

Revenir au répertoire personnel :

```bash id="k3j8vm"
cd ~
```

Ces commandes constituent la base de la navigation sous Linux.

---

## `02 // Créer et manipuler des fichiers`

Créer un fichier vide :

```bash id="n7v4qs"
touch notes.txt
```

Créer un répertoire :

```bash id="c5p9wd"
mkdir laboratoire
```

Créer plusieurs répertoires :

```bash id="y8m2kf"
mkdir -p projet/{logs,data,backup}
```

Copier un fichier :

```bash id="r1t6hx"
cp notes.txt sauvegarde.txt
```

Déplacer ou renommer un fichier :

```bash id="v3k9qa"
mv notes.txt notes-old.txt
```

Supprimer un fichier :

```bash id="f8x2jm"
rm notes-old.txt
```

Supprimer un répertoire et son contenu :

```bash id="s4n7cz"
rm -r laboratoire
```

⚠️ `rm` ne possède pas de corbeille par défaut. Une mauvaise commande peut donc supprimer des données immédiatement.

---

## `03 // Lire le contenu des fichiers`

Afficher rapidement un fichier :

```bash id="m2q7kp"
cat fichier.txt
```

Lire un fichier page par page :

```bash id="w5d1xr"
less fichier.txt
```

Afficher les premières lignes :

```bash id="h8v3bn"
head fichier.txt
```

Afficher les dernières lignes :

```bash id="p6k4tm"
tail fichier.txt
```

Suivre un fichier en temps réel :

```bash id="a9r2wc"
tail -f /var/log/syslog
```

Cette dernière commande est particulièrement utile pour analyser des logs pendant un test ou un dépannage.

---

## `04 // Rechercher des fichiers et du contenu`

Rechercher un fichier :

```bash id="e7q5vz"
find /home -name "notes.txt"
```

Rechercher du texte dans un fichier :

```bash id="u3m8qa"
grep "error" fichier.log
```

Rechercher récursivement :

```bash id="d2x6pf"
grep -R "password" ./projet/
```

Combiner plusieurs commandes :

```bash id="j4n9ks"
cat fichier.log | grep "ERROR"
```

Dans Linux, cette capacité à combiner les commandes avec `|` devient rapidement essentielle.

---

## `05 // Comprendre les permissions`

Afficher les permissions :

```bash id="q7w3mc"
ls -l
```

On peut obtenir :

```text id="c1v8za"
-rw-r--r-- 1 user user 1234 fichier.txt
```

Les permissions principales sont :

```text id="3z6nqp"
r = read
w = write
x = execute
```

Modifier les permissions :

```bash id="b5k2yd"
chmod +x script.sh
```

Modifier le propriétaire :

```bash id="r8m4vt"
sudo chown user:user fichier.txt
```

Les permissions sont fondamentales en sécurité Linux.

Une mauvaise configuration peut permettre à un utilisateur d'accéder ou de modifier des fichiers qui devraient rester protégés.

---

## `06 // Identifier l'utilisateur`

Savoir avec quel compte on travaille est important.

```bash id="t6p3wa"
whoami
```

Afficher les informations de l'utilisateur :

```bash id="n4x8cz"
id
```

Afficher les utilisateurs actuellement connectés :

```bash id="q2v7hm"
who
```

Changer temporairement d'utilisateur :

```bash id="k9d5rx"
su - utilisateur
```

Exécuter une commande avec des privilèges administrateur :

```bash id="m7c3vp"
sudo commande
```

---

## `07 // Gérer les processus`

Afficher les processus :

```bash id="x5j8qn"
ps aux
```

Surveiller les processus en temps réel :

```bash id="v2m6kc"
top
```

Ou, si disponible :

```bash id="p9w4zr"
htop
```

Rechercher un processus :

```bash id="e3k7xb"
ps aux | grep nginx
```

Arrêter un processus :

```bash id="h6n2qt"
kill PID
```

Forcer l'arrêt lorsque cela est nécessaire :

```bash id="s8v4jm"
kill -9 PID
```

Il est préférable d'utiliser d'abord un signal normal avant de recourir à `kill -9`.

---

## `08 // Comprendre le réseau`

Pour quelqu'un qui s'intéresse au pentesting réseau, ces commandes sont indispensables.

Afficher les interfaces :

```bash id="a4m7xs"
ip addr
```

Afficher les routes :

```bash id="k2q8vd"
ip route
```

Tester la connectivité :

```bash id="n6p3zr"
ping 8.8.8.8
```

Résoudre un nom DNS :

```bash id="w9c5ht"
dig example.com
```

Voir les connexions réseau :

```bash id="f3r7km"
ss -tunlp
```

Afficher les informations ARP/voisins :

```bash id="j8x2qb"
ip neigh
```

Ces commandes permettent déjà d'obtenir une vision intéressante de l'état réseau d'une machine.

---

## `09 // Installer des logiciels`

Sur Debian, Ubuntu et Kali Linux, le gestionnaire de paquets le plus courant est `apt`.

Mettre à jour l'index :

```bash id="q5v8nc"
sudo apt update
```

Mettre à jour les paquets :

```bash id="d7m3xp"
sudo apt upgrade
```

Installer un paquet :

```bash id="z2k6hr"
sudo apt install nmap
```

Supprimer un paquet :

```bash id="v4p9bs"
sudo apt remove nmap
```

Rechercher un paquet :

```bash id="c8j5mw"
apt search nmap
```

---

## `10 // L'espace disque et la mémoire`

Afficher l'espace disque :

```bash id="m6x2qa"
df -h
```

Voir la taille d'un répertoire :

```bash id="r9v4ck"
du -sh /var/log
```

Afficher la mémoire disponible :

```bash id="t3n7zp"
free -h
```

Ces commandes sont particulièrement utiles lorsqu'un système commence à manquer de ressources.

---

## `11 // Les redirections et les pipes`

L'un des concepts les plus puissants du terminal est la possibilité de connecter plusieurs commandes.

Par exemple :

```bash id="y5k8mv"
ps aux | grep ssh
```

Le résultat de `ps aux` est envoyé à `grep`.

Rediriger une sortie vers un fichier :

```bash id="b3q7xn"
ip addr > interfaces.txt
```

Ajouter à un fichier existant :

```bash id="h8m4zc"
ip route >> network.txt
```

Lire ensuite le résultat :

```bash id="p2v6kr"
cat network.txt
```

C'est cette logique de combinaison qui rend le shell particulièrement puissant.

---

## `12 // Les commandes à retenir`

Pour commencer, il n'est pas nécessaire de mémoriser des centaines de commandes.

Maîtrisez d'abord celles-ci :

```text id="q8r3wd"
pwd       → emplacement actuel
ls        → fichiers et répertoires
cd        → changer de répertoire
mkdir     → créer un répertoire
touch     → créer un fichier
cp        → copier
mv        → déplacer / renommer
rm        → supprimer
cat       → afficher
less      → lire
grep      → rechercher du texte
find      → rechercher des fichiers
chmod     → permissions
chown     → propriétaire
whoami    → utilisateur actuel
ps        → processus
kill      → arrêter un processus
ip        → réseau
ss        → sockets / connexions
df        → espace disque
free      → mémoire
sudo      → privilèges administrateur
```

---

## `13 // Le terminal avant les outils`

Dans le domaine de la cybersécurité, il est facile de se concentrer directement sur les outils.

Mais avant Nmap, Burp Suite, Wireshark, Metasploit ou Hashcat, il faut savoir travailler confortablement avec le système qui les exécute.

Comprendre Linux permet de mieux comprendre :

* les processus ;
* les permissions ;
* les fichiers ;
* les services ;
* le réseau ;
* les logs ;
* les utilisateurs ;
* les privilèges.

C'est cette base qui permet ensuite d'aller plus loin.

```text id="e4k7yp"
┌──────────────────────────────────────┐
│              GREY-X                  │
│                                      │
│   LINUX → NETWORK → SECURITY         │
│                                      │
│      MASTER THE FUNDAMENTALS         │
└──────────────────────────────────────┘
```

**Avant d'apprendre à attaquer un système, apprenez d'abord à le comprendre.**
