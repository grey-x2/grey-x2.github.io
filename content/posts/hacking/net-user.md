+++
date = '2026-08-14T12:01:00+02:00'
draft = false
title = 'net user en hacking Windows'
description = "Découvrir la commande net user pour énumérer et administrer les comptes utilisateurs Windows dans un laboratoire."
tags = ['Windows', 'net user', 'Red Team', 'Blue Team', 'Active Directory']
categories = ['Système', 'Hacking']
+++

> `GREY-X // WINDOWS LAB`
>
> `net user — USER ENUMERATION`

`net user` est une commande native Windows permettant d'interroger et de gérer les comptes utilisateurs.

En cybersécurité, elle est particulièrement intéressante pour la **reconnaissance locale**.

---

# `01 // Lister les utilisateurs`

```cmd
net user
```

Résultat typique :

```text
User accounts for \\PC-LAB

Administrator
Guest
test
student
```

C'est souvent l'une des premières commandes utiles lors d'une reconnaissance Windows locale.

---

# `02 // Obtenir les détails d'un utilisateur`

```cmd
net user administrator
```

On peut obtenir différentes informations :

```text
User name
Account active
Last logon
Password last set
Local Group Memberships
Global Group memberships
```

---

# `03 // Vérifier son propre compte`

```cmd
net user %username%
```

Pratique pour connaître les informations disponibles sur le compte courant.

---

# `04 // Comptes d'un domaine`

Dans un environnement Active Directory, on peut préciser le domaine :

```cmd
net user /domain
```

Cela permet d'interroger les comptes du domaine lorsque le contexte et les permissions le permettent.

Pour obtenir les informations d'un utilisateur du domaine :

```cmd
net user username /domain
```

---

# `05 // Observer les groupes`

`net user` peut déjà montrer les appartenances aux groupes.

Pour compléter la reconnaissance :

```cmd
net localgroup
```

Puis :

```cmd
net localgroup administrators
```

On peut ainsi identifier les membres du groupe Administrateurs local.

---

# `06 // Pourquoi c'est intéressant en Red Team ?`

Lors d'une reconnaissance locale :

```text
Machine
   ↓
net user
   ↓
Utilisateurs
   ↓
net user username
   ↓
Informations du compte
   ↓
Groupes / privilèges
```

Cela permet de comprendre rapidement la structure des comptes présents sur une machine.

---

# `07 // Mini-lab`

Sur une VM Windows de laboratoire :

```cmd
net user
```

Puis sélectionner un utilisateur :

```cmd
net user test
```

Ensuite :

```cmd
net localgroup
```

Et :

```cmd
net localgroup administrators
```

Comparer les informations obtenues.

---

# `08 // À retenir`

```text
net user
    → lister les utilisateurs

net user username
    → informations sur un utilisateur

net user /domain
    → utilisateurs du domaine

net localgroup
    → lister les groupes locaux

net localgroup administrators
    → membres des Administrateurs
```

`net user` paraît simple, mais c'est une commande très utile pour comprendre rapidement **qui existe sur une machine Windows et quels groupes lui sont associés**.

> **GREY-X — Enumerate first. Understand the environment.**
