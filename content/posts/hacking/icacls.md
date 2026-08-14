+++
date = '2026-08-14T12:08:00+02:00'
draft = false
title = 'icacls en hacking : analyser les permissions NTFS sous Windows'
description = "Comprendre icacls sous Windows pour énumérer, analyser et administrer les permissions NTFS dans un laboratoire de cybersécurité."
tags = ['Windows', 'icacls', 'NTFS', 'Red Team', 'Blue Team', 'Permissions']
categories = ['Système', 'Hacking']
+++

> `GREY-X // WINDOWS LAB`
>
> `icacls — WINDOWS FILESYSTEM PERMISSIONS`

Sous Windows, avoir accès à un fichier ne signifie pas forcément pouvoir le modifier.

L'accès dépend notamment des **permissions NTFS** appliquées aux fichiers et aux répertoires.

La commande native :

```cmd
icacls
```

permet d'afficher et de modifier ces permissions.

En cybersécurité, elle est particulièrement intéressante pour répondre à une question fondamentale :

```text
QUI PEUT FAIRE QUOI ?
```

---

# `01 // Afficher les permissions`

La syntaxe la plus simple :

```cmd
icacls C:\Lab
```

Exemple :

```text
C:\Lab
    NT AUTHORITY\SYSTEM:(I)(F)
    BUILTIN\Administrators:(I)(F)
    LAB\alice:(I)(M)
    LAB\bob:(I)(RX)
```

On peut alors identifier :

```text
SYSTEM
Administrators
alice
bob
```

et surtout leurs niveaux d'accès.

---

# `02 // Comprendre les lettres`

Les permissions les plus courantes sont :

```text
F   → Full access
M   → Modify
RX  → Read & Execute
R   → Read
W   → Write
D   → Delete
```

Par exemple :

```text
LAB\alice:(M)
```

signifie que `alice` dispose de droits de modification.

Alors que :

```text
LAB\bob:(RX)
```

indique principalement :

```text
Read
+
Execute
```

---

# `03 // Le fameux `(I)``

Dans une sortie `icacls`, on rencontre souvent :

```text
(I)
```

Par exemple :

```text
LAB\alice:(I)(M)
```

`I` signifie :

```text
Inherited
```

La permission est donc **héritée d'un dossier parent**.

On peut visualiser :

```text
C:\Lab
   │
   └── Projects
          │
          └── script.ps1
```

Une permission définie sur :

```text
C:\Lab
```

peut être héritée par :

```text
C:\Lab\Projects
```

puis par :

```text
script.ps1
```

L'héritage est donc un élément essentiel de l'analyse des permissions Windows.

---

# `04 // Analyser un fichier`

Pour examiner un fichier précis :

```cmd
icacls C:\Lab\config.txt
```

Exemple :

```text
C:\Lab\config.txt
    NT AUTHORITY\SYSTEM:(F)
    BUILTIN\Administrators:(F)
    LAB\alice:(M)
    LAB\bob:(R)
```

On peut alors construire une matrice simple :

```text
SYSTEM          → Full
Administrators  → Full
alice           → Modify
bob             → Read
```

Cette information devient particulièrement importante lorsque le fichier contient :

```text
configuration
scripts
credentials
backups
logs
secrets
```

---

# `05 // Pourquoi c'est intéressant en Red Team ?`

Imaginons :

```text
C:\Program Files\APP\
```

avec :

```text
LAB\user:(M)
```

Un utilisateur standard disposant de droits de modification sur un emplacement sensible peut représenter un problème de sécurité.

Pourquoi ?

Parce qu'une application ou un service peut utiliser des fichiers présents dans ce répertoire.

La chaîne de risque peut alors ressembler à :

```text
Weak ACL
   ↓
User can modify file
   ↓
Application consumes file
   ↓
Security impact
```

L'objectif d'un pentest est donc de rechercher les **permissions excessives**, puis d'évaluer leur impact réel.

---

# `06 // Vérifier récursivement`

Pour analyser un répertoire et son contenu :

```cmd
icacls C:\Lab /T
```

`/T` signifie que l'analyse est effectuée **récursivement**.

On peut ainsi parcourir :

```text
C:\Lab
├── config
│   ├── app.ini
│   └── database.ini
├── scripts
│   ├── backup.ps1
│   └── deploy.ps1
└── backup
    └── server.zip
```

et examiner les ACL de chaque élément.

Pour un audit, c'est particulièrement pratique.

---

# `07 // Rechercher les permissions problématiques`

L'objectif n'est pas seulement de lire la sortie.

Il faut rechercher des situations comme :

```text
Users:(M)
Everyone:(F)
Authenticated Users:(M)
```

sur des emplacements sensibles.

Par exemple :

```text
C:\Program Files\APP
```

avec :

```text
Users:(M)
```

mérite une analyse.

Mais attention :

> Une permission dangereuse en apparence ne constitue pas automatiquement une vulnérabilité exploitable.

Il faut comprendre :

```text
QUI
 ↓
A QUEL DROIT
 ↓
SUR QUEL OBJET
 ↓
UTILISÉ PAR QUOI
 ↓
AVEC QUEL IMPACT
```

---

# `08 // Vérifier un répertoire sensible`

Dans un laboratoire :

```cmd
icacls "C:\Program Files"
```

Puis :

```cmd
icacls "C:\Program Files\NomApplication"
```

On peut comparer les permissions :

```text
Parent
  ↓
Application
  ↓
Executable
  ↓
Configuration
```

Cette approche permet de repérer les différences d'ACL entre les différents niveaux.

---

# `09 // Modifier une permission`

`icacls` ne sert pas uniquement à lire les ACL.

Il permet également de les modifier.

Par exemple :

```cmd
icacls C:\Lab /grant LAB\alice:(M)
```

Cette commande accorde à `alice` le droit :

```text
Modify
```

sur le répertoire.

Pour un laboratoire, cela permet de comprendre concrètement comment évoluent les ACL.

---

# `10 // Retirer une permission`

Pour retirer une entrée :

```cmd
icacls C:\Lab /remove LAB\alice
```

Il est également possible d'utiliser :

```cmd
icacls C:\Lab /grant:r LAB\alice:(R)
```

Le `:r` permet de remplacer les autorisations explicites existantes pour cet utilisateur par celles spécifiées.

Exemple :

```text
Avant
LAB\alice → Modify

Après
LAB\alice → Read
```

---

# `11 // Héritage des permissions`

L'héritage est un concept fondamental.

Pour désactiver l'héritage :

```cmd
icacls C:\Lab /inheritance:d
```

Pour le réactiver :

```cmd
icacls C:\Lab /inheritance:e
```

Il faut cependant être prudent avec les modifications d'ACL.

Une mauvaise manipulation peut provoquer :

```text
accès refusé
application cassée
service inaccessible
héritage incorrect
```

Dans un environnement réel, les ACL doivent donc être modifiées avec précaution.

---

# `12 // Sauvegarder les ACL`

Dans un audit, il peut être utile de sauvegarder les permissions avant modification :

```cmd
icacls C:\Lab /save C:\Temp\lab-acl.txt /T
```

Cela permet de conserver une représentation des ACL.

On peut ensuite restaurer les permissions avec :

```cmd
icacls C:\ /restore C:\Temp\lab-acl.txt
```

La restauration doit évidemment être réalisée avec prudence et sur le bon périmètre.

---

# `13 // Le lien avec net share`

C'est ici que `icacls`, `net share` et `net use` deviennent particulièrement intéressants ensemble.

On peut avoir :

```text
             WINDOWS SERVER
                  │
          ┌───────┴───────┐
          ▼               ▼
      net share        NTFS ACL
          │               │
          ▼               ▼
       SMB SHARE       icacls
          │               │
          └───────┬───────┘
                  ▼
             EFFECTIVE
              ACCESS
```

Exemple :

```text
Share Permissions
        +
NTFS Permissions
        ↓
Effective Access
```

C'est pourquoi voir :

```text
\\SERVER\Backup
```

ne suffit pas.

Il faut également comprendre :

```bash
Qui peut accéder ?
Qui peut lire ?
Qui peut écrire ?
Qui peut modifier ?
Qui peut supprimer ?
```

---

# `14 // Mini-lab GREY-X`

Créer un environnement :

```cmd
mkdir C:\Lab
echo GREY-X > C:\Lab\test.txt
```

Afficher les permissions :

```cmd
icacls C:\Lab
```

Puis :

```cmd
icacls C:\Lab\test.txt
```

Créer un utilisateur de laboratoire :

```cmd
net user labuser "LabPassword123!" /add
```

Accorder un accès en lecture :

```cmd
icacls C:\Lab /grant labuser:(R)
```

Vérifier :

```cmd
icacls C:\Lab
```

Puis modifier :

```cmd
icacls C:\Lab /grant:r labuser:(M)
```

Observer la différence.

Enfin, nettoyer le laboratoire :

```cmd
icacls C:\Lab /remove labuser
net user labuser /delete
```

L'objectif est de voir concrètement la différence entre :

```text
Read
Modify
Full Control
```

---

# `15 // Une ACL n'est pas une vulnérabilité`

C'est une distinction importante en pentest.

Trouver :

```text
Users:(M)
```

ne signifie pas automatiquement :

```text
RCE
```

ou :

```text
Privilege Escalation
```

Il faut analyser le contexte.

Par exemple :

```text
User can modify file
        ↓
Quel fichier ?
        ↓
Qui l'utilise ?
        ↓
Quel processus le charge ?
        ↓
Avec quels privilèges ?
        ↓
Quel impact ?
```

C'est cette analyse qui permet de transformer une **mauvaise configuration** en véritable finding de sécurité lorsqu'un impact démontrable existe.

---

# `16 // Côté Blue Team`

Pour le défenseur, `icacls` permet d'auditer rapidement les permissions :

```cmd
icacls C:\Important
```

Puis récursivement :

```cmd
icacls C:\Important /T
```

Il faut notamment rechercher :

```text
Everyone
Users
Authenticated Users
```

avec des permissions trop larges sur :

```rust
applications
scripts
services
configurations
sauvegardes
répertoires système
```

La règle générale reste :

```text
Least Privilege
        +
Controlled Inheritance
        +
Regular ACL Review
```

---

# `17 // Commandes essentielles`

```cmd
icacls C:\Lab
```

→ afficher les ACL.

```cmd
icacls C:\Lab /T
```

→ analyser récursivement.

```cmd
icacls C:\Lab /grant user:(R)
```

→ accorder la lecture.

```cmd
icacls C:\Lab /grant user:(M)
```

→ accorder la modification.

```cmd
icacls C:\Lab /remove user
```

→ supprimer une entrée.

```cmd
icacls C:\Lab /inheritance:d
```

→ désactiver l'héritage.

```cmd
icacls C:\Lab /inheritance:e
```

→ activer l'héritage.

```cmd
icacls C:\Lab /save C:\Temp\acl.txt /T
```

→ sauvegarder les ACL.

---

# `18 // La chaîne complète`

Avec les trois commandes :

```text
net share
```

on découvre ce que Windows **partage**.

```text
net use
```

on observe les **connexions aux ressources réseau**.

```text
icacls
```

on analyse les **permissions NTFS**.

On obtient alors :

```python
             WINDOWS
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
    net share  net use  icacls
        │       │        │
        ▼       ▼        ▼
     SHARES  SESSIONS   ACLs
        │       │        │
        └───────┼────────┘
                ▼
          ACCESS CONTROL
                │
                ▼
          SECURITY REVIEW
```

> **`net share` montre l'exposition.**

> **`net use` montre les connexions.**

> **`icacls` montre les permissions.**

Et c'est précisément la combinaison des trois qui permet de mieux comprendre la surface d'accès d'un environnement Windows.

> `GREY-X — Enumerate. Inspect. Understand. Secure.`
