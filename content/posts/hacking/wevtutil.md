+++
date = '2026-08-14T11:57:00+02:00'
draft = false
title = 'wevtutil en hacking : travailler avec les logs Windows'
description = "Découvrir wevtutil sous Windows pour consulter et analyser les journaux d'événements pendant un lab de cybersécurité."
tags = ['Windows', 'wevtutil', 'Red Team', 'Blue Team', 'Logs', 'SIEM']
categories = ['Système', 'Hacking']
+++

> `GREY-X // WINDOWS LAB`
>
> `wevtutil — WINDOWS EVENT LOGS`

`wevtutil` est une commande native de Windows permettant de gérer les **Event Logs**.

En cybersécurité, elle est particulièrement utile pour comprendre ce qu'une action laisse comme traces sur une machine Windows.

---

# `01 // Lister les journaux`

```cmd
wevtutil el
```

On peut notamment retrouver :

```text
Security
System
Application
Windows PowerShell
Microsoft-Windows-...
```

---

# `02 // Lire le journal Security`

```cmd
wevtutil qe Security /c:10 /f:text
```

Ici :

```text
qe       → Query Events
Security → journal ciblé
/c:10    → 10 événements
/f:text  → format texte
```

---

# `03 // Rechercher un événement`

Par exemple, rechercher les événements liés aux connexions :

```cmd
wevtutil qe Security /q:"*[System[(EventID=4624)]]" /f:text
```

`4624` correspond à une **ouverture de session réussie**.

Pour les échecs :

```cmd
wevtutil qe Security /q:"*[System[(EventID=4625)]]" /f:text
```

Cela permet d'étudier les tentatives de connexion dans un laboratoire.

---

# `04 // PowerShell`

Les journaux PowerShell sont également intéressants :

```cmd
wevtutil qe "Windows PowerShell" /c:10 /f:text
```

On peut ainsi observer les événements générés par certaines activités PowerShell.

---

# `05 // Observer les événements système`

```cmd
wevtutil qe System /c:20 /f:text
```

Pratique pour analyser :

```text
services
drivers
erreurs système
démarrage
arrêt
```

---

# `06 // Exporter un journal`

Dans un lab, on peut exporter un journal pour l'analyser :

```cmd
wevtutil epl Security security.evtx
```

Le fichier `.evtx` peut ensuite être étudié avec des outils d'analyse Windows.

---

# `07 // Pourquoi c'est intéressant en hacking ?`

Lors d'un pentest ou d'un lab Red Team, une question importante est :

```text
"Quelles traces mon action génère-t-elle ?"
```

`wevtutil` permet donc de faire le lien :

```text
ACTION
  ↓
WINDOWS
  ↓
EVENT LOG
  ↓
wevtutil
  ↓
ANALYSE
```

Cela permet de comprendre la visibilité d'une activité pour le défenseur.

---

# `08 // Côté Blue Team`

Un défenseur peut utiliser les mêmes journaux pour rechercher :

```text
4624 → connexion réussie
4625 → échec de connexion
4688 → création d'un processus
```

L'intérêt est de corréler les événements plutôt que de regarder un événement isolé.

---

# `09 // Mini-lab`

Sur une VM Windows :

```cmd
wevtutil el
```

Puis :

```cmd
wevtutil qe Security /c:20 /f:text
```

Effectuer ensuite quelques actions normales :

```text
connexion utilisateur
lancement d'un programme
exécution PowerShell
accès à une ressource
```

Puis revenir aux logs et observer les événements générés.

> **Le but n'est pas seulement de savoir attaquer Windows. Il faut aussi savoir quelles traces l'attaque laisse derrière elle.**

---

# `10 // À retenir`

```text
wevtutil el
    → lister les logs

wevtutil qe
    → interroger les événements

wevtutil epl
    → exporter un log

4624
    → logon réussi

4625
    → logon échoué

4688
    → création d'un processus
```

> **GREY-X — Hack it. Observe it. Detect it.**
