+++
date = '2026-08-14T12:07:00+02:00'
draft = false
title = 'net use en hacking : comprendre les connexions SMB sous Windows'
description = "Comprendre net use sous Windows pour énumérer, établir et gérer les connexions SMB dans un laboratoire de cybersécurité."
tags = ['Windows', 'net use', 'SMB', 'Red Team', 'Blue Team', 'Enumeration']
categories = ['Système', 'Hacking']
+++

> `GREY-X // WINDOWS LAB`
>
> `net use — WINDOWS SMB SESSIONS`

`net use` est une commande native de Windows permettant de **gérer les connexions vers des ressources réseau**.

Elle est particulièrement intéressante en cybersécurité car elle permet d'observer un élément essentiel de Windows :

```text
MACHINE
   ↓
SMB
   ↓
AUTHENTIFICATION
   ↓
SESSION
   ↓
RESOURCE
```

Dans un environnement de pentest ou de laboratoire, `net use` permet notamment de comprendre :

```text
connexions SMB
partages réseau
sessions utilisateur
lecteurs réseau
authentification
```

---

# `01 // Énumérer les connexions réseau`

La commande la plus simple :

```cmd
net use
```

Elle affiche les connexions réseau actuellement établies.

Exemple :

```text
Status       Local     Remote
------------------------------------------------
OK           Z:        \\192.168.1.20\Public
OK                     \\192.168.1.30\Backup
```

On peut alors identifier :

```text
Local
    → lettre de lecteur éventuellement utilisée

Remote
    → ressource distante

Status
    → état de la connexion
```

Cette information peut être intéressante lors d'une analyse d'une machine Windows.

---

# `02 // Comprendre le concept de session SMB`

Lorsqu'un poste Windows accède à :

```text
\\192.168.1.20\Public
```

une communication SMB est établie avec le serveur.

On peut simplifier le processus :

```text
CLIENT
  │
  │ SMB
  ▼
SERVER
  │
  ├── Authentication
  │
  └── Share
        │
        ▼
      Access
```

`net use` permet justement de gérer cette relation.

---

# `03 // Se connecter à un partage`

Dans un laboratoire :

```cmd
net use \\192.168.1.20\Public
```

Windows tente alors d'établir une connexion vers :

```text
\\192.168.1.20\Public
```

Selon la configuration du serveur et du compte courant, une authentification peut être nécessaire.

On peut également préciser un utilisateur :

```cmd
net use \\192.168.1.20\Public /user:LAB\alice
```

Windows demandera alors le mot de passe du compte.

Cette méthode est préférable dans un lab lorsqu'on veut comprendre précisément **quel compte est utilisé pour accéder à une ressource**.

---

# `04 // Monter un partage comme lecteur`

Une fonctionnalité très utilisée :

```cmd
net use Z: \\192.168.1.20\Public
```

Le partage devient alors accessible via :

```text
Z:\
```

Au lieu de :

```text
\\192.168.1.20\Public
```

On obtient :

```text
Windows Explorer
       │
       ▼
       Z:\
       │
       ▼
\\192.168.1.20\Public
       │
       ▼
      SMB
```

Cette technique est courante dans les environnements professionnels pour donner accès à des ressources partagées.

---

# `05 // Connexion persistante`

Par défaut, une connexion peut être liée à la session courante.

Pour demander à Windows de conserver la connexion :

```cmd
net use Z: \\192.168.1.20\Public /persistent:yes
```

Pour une connexion non persistante :

```cmd
net use Z: \\192.168.1.20\Public /persistent:no
```

Cela permet de comprendre une notion importante :

```text
Session temporaire
        ≠
Connexion persistante
```

Dans un audit, les lecteurs réseau persistants peuvent également être intéressants à identifier car ils révèlent parfois l'architecture interne d'une organisation.

---

# `06 // Utiliser des identifiants différents`

Windows peut utiliser un compte différent du compte actuellement connecté :

```cmd
net use \\192.168.1.20\Public /user:LAB\alice
```

Pour un compte local :

```cmd
net use \\192.168.1.20\Public /user:192.168.1.20\alice
```

Le serveur vérifie alors les informations d'authentification avant d'autoriser l'accès au partage.

Dans un lab, cela permet de comparer les droits de plusieurs comptes :

```text
alice
   ↓
SMB
   ↓
Public
   ↓
READ

bob
   ↓
SMB
   ↓
Public
   ↓
READ + WRITE
```

---

# `07 // Supprimer une connexion`

Pour supprimer une connexion précise :

```cmd
net use Z: /delete
```

Ou :

```cmd
net use \\192.168.1.20\Public /delete
```

Pour supprimer toutes les connexions réseau gérées par `net use` :

```cmd
net use * /delete
```

Windows demandera généralement une confirmation.

Cette opération est utile dans un laboratoire pour remettre l'environnement dans son état initial.

---

# `08 // Pourquoi c'est intéressant en Red Team ?`

Lorsqu'un pentester obtient l'accès à une machine Windows, les connexions réseau existantes peuvent fournir des informations intéressantes.

Par exemple :

```cmd
net use
```

peut révéler :

```text
serveurs internes
adresses IP
partages
lecteurs réseau
ressources métier
```

Imaginons :

```text
Z:  \\192.168.10.20\Finance
Y:  \\192.168.10.30\IT
X:  \\192.168.10.40\Backup
```

Cela donne déjà une vision partielle de l'environnement :

```text
Windows Client
      │
      ├── Finance
      ├── IT
      └── Backup
```

L'intérêt n'est donc pas seulement l'accès aux fichiers.

C'est aussi **l'information révélée par les connexions existantes**.

---

# `09 // Différence entre net share et net use`

Ces deux commandes sont souvent confondues.

### `net share`

Permet de gérer les **ressources partagées sur la machine**.

```text
SERVER
  │
  ├── Public
  ├── Backup
  └── Projects
```

### `net use`

Permet de gérer les **connexions de la machine vers des ressources réseau**.

```text
CLIENT
  │
  ├── Z: → \\SERVER\Public
  ├── Y: → \\SERVER\IT
  └── X: → \\SERVER\Backup
```

On peut retenir :

```text
net share
    → "Qu'est-ce que cette machine partage ?"

net use
    → "À quelles ressources réseau cette machine est-elle connectée ?"
```

---

# `10 // Le point important : les identifiants`

Lorsqu'une connexion SMB est établie, Windows doit déterminer **avec quelle identité** accéder à la ressource.

On peut donc avoir :

```text
CLIENT
  │
  │ Credentials
  ▼
SERVER
  │
  ▼
SMB SHARE
```

Le serveur applique ensuite ses contrôles d'accès.

Il faut donc distinguer :

```text
Authentication
       ↓
"Qui es-tu ?"

Authorization
       ↓
"Qu'as-tu le droit de faire ?"
```

Cette distinction est fondamentale lors d'un pentest Windows.

---

# `11 // Observer les connexions existantes`

Dans un laboratoire Windows :

```cmd
net use
```

Puis :

```cmd
net use Z: \\192.168.56.20\Lab
```

Vérifier :

```cmd
net use
```

On devrait retrouver une entrée correspondant à :

```text
Z:
\\192.168.56.20\Lab
```

Puis supprimer :

```cmd
net use Z: /delete
```

Et vérifier de nouveau :

```cmd
net use
```

Ce simple exercice permet d'observer le cycle :

```text
CONNECT
   ↓
SESSION
   ↓
ACCESS
   ↓
DISCONNECT
```

---

# `12 // Côté Blue Team`

`net use` est également intéressant pour le défenseur.

Une connexion inattendue vers :

```text
\\10.10.10.50\Backup
```

peut être un élément à investiguer.

Il faut alors chercher à corréler :

```text
Utilisateur
    +
Machine
    +
Connexion SMB
    +
Partage
    +
Fichiers accédés
    +
Horodatage
```

Une commande seule ne permet évidemment pas de déterminer qu'une activité est malveillante.

Mais elle peut fournir un **indice d'investigation**.

---

# `13 // Mini-lab GREY-X`

Créer ou utiliser deux machines Windows :

```text
CLIENT
10.10.10.10

SERVER
10.10.10.20
```

Sur le serveur, créer un partage de laboratoire :

```text
C:\Lab
```

Puis vérifier son partage.

Depuis le client :

```cmd
net view \\10.10.10.20
```

Puis :

```cmd
net use \\10.10.10.20\Lab
```

Vérifier :

```cmd
net use
```

Monter ensuite le partage :

```cmd
net use Z: \\10.10.10.20\Lab
```

Tester :

```cmd
dir Z:\
```

Enfin :

```cmd
net use Z: /delete
```

Le laboratoire permet d'observer :

```text
CLIENT
  │
  │ SMB
  ▼
SERVER
  │
  ▼
Lab Share
  │
  ▼
Z:\
```

---

# `14 // Chaîne d'énumération Windows`

Dans un scénario de pentest, `net use` peut être combiné avec d'autres commandes natives :

```text
net view
    ↓
identifier les ressources
    ↓
net use
    ↓
identifier / établir les connexions
    ↓
net share
    ↓
comprendre les partages
    ↓
permissions
    ↓
analyse des données
```

Chaque commande répond à une question différente.

```text
net view
→ Quelles ressources sont visibles ?

net use
→ Quelles connexions réseau existent ?

net share
→ Qu'est-ce que cette machine partage ?

permissions
→ Que peut réellement faire le compte ?
```

---

# `15 // À retenir`

```cmd
net use
```

→ afficher les connexions réseau.

```cmd
net use \\IP\SHARE
```

→ se connecter à un partage.

```cmd
net use Z: \\IP\SHARE
```

→ monter le partage comme lecteur réseau.

```cmd
net use \\IP\SHARE /user:DOMAIN\user
```

→ utiliser un compte spécifique.

```cmd
net use Z: /delete
```

→ supprimer une connexion.

```cmd
net use * /delete
```

→ supprimer les connexions gérées par `net use`.

---

# `16 // La vision pentest`

Une commande comme :

```cmd
net use
```

peut sembler anodine.

Mais dans un environnement Windows, elle permet de transformer une information locale en **cartographie partielle du réseau** :

```text
       WINDOWS CLIENT
             │
             ▼
         net use
             │
      ┌──────┼──────┐
      ▼      ▼      ▼
    FILE    IT    BACKUP
    SHARE  SHARE   SHARE
      │      │      │
      └──────┼──────┘
             ▼
            SMB
             │
             ▼
      AUTH + PERMISSIONS
```

C'est précisément ce qui rend les commandes Windows natives intéressantes en cybersécurité :

> **Elles ne sont pas des outils de hacking à proprement parler. Elles permettent surtout de comprendre comment Windows expose, consomme et contrôle les ressources réseau.**

> `GREY-X — Enumerate. Authenticate. Understand.`
