+++
date = '2026-08-14T12:05:00+02:00'
draft = false
title = 'net share en hacking : énumérer les partages réseau Windows'
description = "Comprendre net share sous Windows pour énumérer, analyser et administrer les partages réseau dans un laboratoire de cybersécurité."
tags = ['Windows', 'net share', 'SMB', 'Red Team', 'Blue Team', 'Enumeration']
categories = ['Système', 'Hacking']
+++

> `GREY-X // WINDOWS LAB`
>
> `net share — WINDOWS NETWORK SHARES`

Dans un environnement Windows, les **partages réseau SMB** permettent d'exposer des dossiers et des ressources à d'autres machines.

La commande native `net share` permet d'énumérer et d'administrer ces partages.

En pentest, elle est intéressante parce qu'un partage mal configuré peut révéler :

```text
documents
scripts
sauvegardes
fichiers de configuration
ressources d'administration
données internes
```

L'objectif n'est donc pas simplement de connaître la commande, mais de comprendre **ce qu'elle révèle sur une machine Windows**.

---

# `01 // Lister les partages`

La commande la plus simple :

```cmd
net share
```

Exemple :

```text
Share name   Resource
------------------------------------------------
ADMIN$       C:\Windows
C$           C:\
IPC$
Users        C:\Users
Public       C:\Users\Public
```

Cette sortie permet déjà d'identifier les ressources exposées par la machine.

On peut distinguer deux grandes catégories.

### Partages administratifs

```text
ADMIN$
C$
IPC$
```

Ils sont généralement liés au fonctionnement et à l'administration de Windows.

### Partages configurés par l'administrateur

Par exemple :

```text
Public
Documents
Backup
Projects
```

Ce sont souvent ces partages qui méritent une attention particulière lors d'un audit.

---

# `02 // Comprendre les partages administratifs`

Les partages terminés par `$` sont généralement **cachés de l'énumération classique**.

Par exemple :

```text
C$
ADMIN$
IPC$
```

Le caractère `$` indique qu'il s'agit d'un partage caché dans l'interface normale de navigation réseau.

Cela ne signifie pas :

```text
inaccessible
```

mais plutôt :

```text
non affiché normalement
```

L'accès reste soumis aux mécanismes d'authentification et d'autorisation Windows.

---

# `03 // Interroger un partage précis`

On peut demander davantage d'informations sur un partage :

```cmd
net share Public
```

Exemple :

```text
Share name        Public
Path              C:\Users\Public
Remark            Public files
Maximum users     No limit
Users
Caching           Manual caching of documents
```

Cette commande permet notamment de connaître :

```text
nom du partage
chemin local
description
limitation du nombre d'utilisateurs
configuration du partage
```

Le point intéressant pour un pentester est le **chemin local**.

Par exemple :

```text
Public → C:\Users\Public
Backup → D:\Backups
Projects → C:\Projects
```

Cela permet de comprendre comment les ressources réseau sont organisées sur le système.

---

# `04 // Le lien avec SMB`

`net share` travaille au niveau de la configuration des **partages Windows**.

L'accès à ces ressources passe généralement par **SMB**.

On peut donc représenter le fonctionnement ainsi :

```text
CLIENT
  │
  │ SMB
  ▼
WINDOWS SERVER
  │
  ├── Share: Public
  │      └── C:\Users\Public
  │
  ├── Share: Backup
  │      └── D:\Backups
  │
  └── Share: Projects
         └── C:\Projects
```

Lors d'un pentest réseau, l'énumération SMB permet donc de passer de :

```text
IP
 ↓
SMB
 ↓
SHARES
 ↓
PERMISSIONS
 ↓
DATA
```

---

# `05 // Tester l'accès à un partage`

Une fois qu'un partage est identifié, il faut déterminer si le compte utilisé peut réellement y accéder.

Depuis une machine Windows :

```cmd
net use \\192.168.1.10\Public
```

Selon la configuration du serveur, Windows peut demander des informations d'authentification.

Pour utiliser explicitement un compte :

```cmd
net use \\192.168.1.10\Public /user:DOMAIN\user
```

L'objectif dans un laboratoire est d'observer la différence entre :

```text
partage visible
```

et :

```text
partage accessible
```

Ce sont deux choses différentes.

---

# `06 // Énumération locale vs distante`

Un point important :

```cmd
net share
```

est principalement utilisé pour afficher les partages configurés sur **la machine locale**.

Pour analyser une machine distante, d'autres outils Windows ou SMB peuvent être utilisés.

Par exemple :

```cmd
net view \\192.168.1.10
```

Cette commande permet d'interroger les ressources partagées visibles sur une machine distante.

On obtient alors une logique d'énumération :

```text
net view
    ↓
identifier les ressources réseau
    ↓
identifier les shares
    ↓
tester les permissions
    ↓
analyser les données accessibles
```

---

# `07 // Pourquoi c'est intéressant en Red Team ?`

Imaginons un environnement de laboratoire :

```text
DC01
 ├── SYSVOL
 ├── NETLOGON
 ├── Backup
 └── IT
```

Un pentester peut chercher à comprendre :

```text
Quels partages existent ?
Qui peut y accéder ?
Quelles données sont exposées ?
Les permissions sont-elles correctement configurées ?
```

Un partage comme :

```text
Backup
```

peut être particulièrement intéressant à examiner.

Pourquoi ?

Parce qu'une sauvegarde peut contenir :

```text
configurations
scripts
exports
fichiers XML
fichiers INI
archives
informations techniques
```

Et parfois des informations sensibles mal protégées.

Le problème n'est donc pas nécessairement le partage lui-même.

Le véritable problème peut être :

```text
SHARE
  +
PERMISSIONS TROP LARGES
  +
DONNÉES SENSIBLES
```

---

# `08 // Les permissions : deux couches`

Lorsqu'on analyse un partage Windows, il faut éviter une erreur classique :

> Voir un partage accessible ne signifie pas forcément que tous les fichiers sont accessibles.

Windows peut appliquer plusieurs niveaux de permissions.

```text
             SMB SHARE
                 │
        Share Permissions
                 │
                 ▼
          NTFS Permissions
                 │
                 ▼
              FILE
```

Par exemple :

```text
Share → Read
NTFS  → Modify
```

Le résultat final dépend de la combinaison des permissions.

Lors d'un audit, il faut donc examiner **les permissions du partage ET les permissions NTFS**.

---

# `09 // Créer un partage dans un lab`

Dans un laboratoire Windows, un administrateur peut créer un partage :

```cmd
net share Lab=C:\Lab
```

Windows expose alors :

```text
\\HOSTNAME\Lab
```

Pour supprimer le partage :

```cmd
net share Lab /delete
```

Cette partie est particulièrement utile pour construire un environnement de test.

Par exemple :

```text
C:\Lab
 ├── Public
 ├── Backup
 ├── Scripts
 └── Documents
```

Puis :

```cmd
net share Lab=C:\Lab
```

On dispose alors d'une ressource SMB contrôlée pour expérimenter.

---

# `10 // Modifier un partage`

La commande peut également être utilisée pour administrer les ressources partagées.

Par exemple :

```cmd
net share Lab=C:\Lab /remark:"Laboratoire Windows"
```

On peut ensuite vérifier :

```cmd
net share Lab
```

L'objectif pédagogique est de comprendre le cycle :

```text
CREATE
   ↓
CONFIGURE
   ↓
ENUMERATE
   ↓
TEST
   ↓
REMOVE
```

---

# `11 // Côté Blue Team`

Du côté défenseur, `net share` permet rapidement de vérifier les ressources exposées :

```cmd
net share
```

Il faut rechercher notamment :

```text
shares inutilisés
shares hérités
shares trop permissifs
données sensibles
partages temporaires oubliés
```

Une bonne pratique consiste à limiter l'exposition :

```text
Need to know
     +
Least privilege
     +
NTFS permissions
     +
Share permissions
```

Un partage qui n'est plus nécessaire devrait être supprimé.

---

# `12 // Mini-lab GREY-X`

Sur une VM Windows de laboratoire :

### Étape 1 — Énumérer

```cmd
net share
```

### Étape 2 — Créer un partage de test

```cmd
mkdir C:\Lab
echo GREY-X > C:\Lab\test.txt
```

Puis :

```cmd
net share Lab=C:\Lab
```

### Étape 3 — Vérifier

```cmd
net share Lab
```

### Étape 4 — Tester depuis une autre machine Windows

```cmd
net view \\IP_WINDOWS
```

Puis :

```cmd
net use \\IP_WINDOWS\Lab
```

### Étape 5 — Supprimer

```cmd
net share Lab /delete
```

Ce petit laboratoire permet de comprendre concrètement :

```text
Windows
   ↓
Share
   ↓
SMB
   ↓
Authentication
   ↓
Permissions
   ↓
Access
```

---

# `13 // Quelques commandes à retenir`

```cmd
net share
```

→ énumérer les partages locaux.

```cmd
net share NOM_DU_SHARE
```

→ afficher les informations d'un partage.

```cmd
net share NOM=CHEMIN
```

→ créer un partage.

```cmd
net share NOM /delete
```

→ supprimer un partage.

```cmd
net view \\IP
```

→ énumérer les ressources visibles d'une machine distante.

```cmd
net use \\IP\SHARE
```

→ établir une connexion vers un partage SMB.

---

# `14 // Perspective Pentest`

`net share` est une petite commande, mais elle s'intègre dans une chaîne d'énumération beaucoup plus large :

```text
      DISCOVERY
          │
          ▼
        SMB
          │
          ▼
      NET SHARE
          │
          ▼
      SHARES
          │
          ▼
     PERMISSIONS
          │
          ▼
       CONTENT
          │
          ▼
    RISK ANALYSIS
```

L'objectif d'un pentest n'est pas de simplement dire :

```text
"Le port SMB est ouvert."
```

Il faut aller plus loin :

```text
Quel service ?
Quels partages ?
Quelles permissions ?
Quelles données ?
Quel impact ?
```

C'est cette progression qui transforme une simple **énumération technique** en véritable **analyse de sécurité**.

---

# `15 // À retenir`

```text
net share
    → énumérer les partages locaux

net share SHARE
    → analyser un partage

net view \\IP
    → voir les ressources réseau

net use \\IP\SHARE
    → accéder à une ressource SMB

C$
ADMIN$
IPC$
    → partages Windows particuliers

Share Permissions
        +
NTFS Permissions
        =
Effective Access
```

> **Un partage réseau n'est pas seulement un dossier accessible à distance.**
>
> **C'est une frontière de sécurité entre le système de fichiers Windows et le réseau.**

> `GREY-X — Enumerate. Understand. Validate. Secure.`
