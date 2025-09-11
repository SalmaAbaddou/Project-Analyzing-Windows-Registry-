# Project: Analyzing Windows Registry
#  Windows Memory Forensics – Analyse du Registry

Ce projet montre comment, en se basant sur un **Windows Memory Dump**, on peut **extraire les hives du registre Windows** et les analyser pour comprendre comment explorer et investiguer le **Windows Registry** lors d’une enquête forensic.


##  Objectifs du projet

- Extraire les **Registry Hives** à partir d’un dump mémoire
- Analyser les clés et valeurs importantes du registre
- Comprendre comment naviguer dans le registre Windows en mode forensic


##  Prérequis

Pour réaliser ce projet, il faut :

- Un **Windows Memory Dump** (fichier `.raw` ou `.mem`)
- L’outil **[Volatility]**
Pour ce projet, nous allons travailler sur un **dump mémoire Windows** nommé : Challenge.raw
Ce fichier servira de base pour toutes nos analyses du registre Windows.


Profile
#   Identifier le profil mémoire

Avant d’extraire ou analyser le registre, il faut connaître le **profil Windows** du dump mémoire.  
Pour cela, on utilise la commande suivante :
```bash
python2 vol.py -f /media/sf_sharedF_pc_VM/Challenge/Challenge.raw imageinfo
```
## <img width="526" height="202" alt="{0692C3A0-FEE0-4B4F-A34A-E35F38666365}" src="https://github.com/user-attachments/assets/5cd0c63e-f091-4144-aa91-d38e547c1b0a" />

#  Extraction des Registry Hives

Une fois que nous avons identifié le **profil correct** de la mémoire (exemple : `Win7SP0x86`),  
nous pouvons l’utiliser pour extraire la **liste des Registry Hives** présents dans le dump mémoire.

Commande utilisée :

```bash
python2 vol.py -f /media/sf_sharedF_pc_VM/Challenge/Challenge.raw --profile=Win7SP0x86 hivelist
```
<img width="532" height="216" alt="{9FD24FB4-A391-41F8-85D8-630D59133CB9}" src="https://github.com/user-attachments/assets/ccf1ef76-fda3-4870-8238-076899b34a6c" />

hivelist → plugin Volatility qui affiche la liste des Registry Hives trouvés dans le dump mémoire

# Evidence of Execution
c’est-à-dire identifier les traces des programmes et applications qui ont été exécutés sur un système Windows.
## Key Microsoft\Windows\CurrentVersion\Run
Cette clé contient la liste des programmes configurés pour démarrer automatiquement lorsque Windows se lance.

Chaque valeur de la clé correspond à un programme :

Nom de la valeur → nom du programme ou du service

Donnée de la valeur → chemin complet de l’exécutable à lancer

<img width="502" height="192" alt="{B2248F71-DAB3-40F9-8362-82A8AFE42E99}" src="https://github.com/user-attachments/assets/930b3489-92ce-45bf-ba19-734bfc20f121" />

Le plugin `printkey` de Volatility permet d’extraire et d’afficher **le contenu d’une clé spécifique du registre Windows** depuis un dump mémoire.

Values → liste des valeurs dans la clé.
REG_SZ → type de donnée : chaîne de caractères.
VBoxTray → nom de la valeur (ici c’est le programme configuré pour démarrage automatique).
(S) → valeur stable.

C:\Windows\system32\VBoxTray.exe → chemin complet du programme exécuté automatiquement au démarrage (dans ce cas, l’outil VirtualBox Guest Additions Tray).
Dans ton exemple, le seul programme trouvé est VBoxTray.exe.

Cela constitue une preuve d’exécution (Evidence of Execution) : on sait que ce programme a été lancé automatiquement par Windows.

## UserAssist key
```bash
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist
```

Chaque utilisateur a sa propre clé UserAssist dans son fichier NTUSER.DAT.
Cette clé enregistre les programmes et fichiers que l’utilisateur a lancés via l’interface graphique (clic sur le menu Démarrer, bureau, explorateur de fichiers, etc.).

    - Double-clicking an app icon on the desktop
    
    - Opening a program from the Start menu
    
    - Launching from a file association (e.g., double-clicking a `.docx` opens Word)

Ce que le plugin userassist fait

Le plugin userassist de Volatility va extraire et décoder ces entrées depuis le dump mémoire.

Les données dans UserAssist sont encodées en ROT13, et Volatility se charge de les décoder automatiquement.

- Le résultat montre :

       Nom du programme / chemin (ex: calc.exe, notepad.exe)

       Nombre d’exécutions (Run count)
     
       Dernière exécution (Last executed time)

<img width="585" height="369" alt="{D215E29F-2643-4E2D-B59A-9BB29451D419}" src="https://github.com/user-attachments/assets/c82436bc-9e6e-4cbf-86f9-0b7f309c60cf" />

- Interprétation forensic

Ces informations donnent une preuve directe de l’exécution de programmes par l’utilisateur.

Utile pour :

     Voir quels programmes ont été lancés et combien de fois.

     Identifier des programmes suspects utilisés par l’attaquant.

     Reconstituer une timeline d’exécution sur le poste analysé.

# ShimCache

- Ce que ShimCache fait

ShimCache est un cache de compatibilité des applications.

Windows enregistre dans ce cache les exécutables qui existent sur le système, avec certaines métadonnées (chemin, taille, date de modification).

- Ce que le plugin shimcache fait

Le plugin Volatility va extraire et afficher la liste des exécutables enregistrés dans le cache.

Il fournit pour chaque exécutable :

     Nom et chemin complet (ex: C:\Windows\System32\notepad.exe)

     Dernière date/heure de modification ou d’accès (parfois dernière exécution)

     Utile car même si un programme a été supprimé ou déplacé, il peut toujours apparaître dans le ShimCache.

<img width="567" height="230" alt="{45419AD5-8425-48B4-B20D-64872931D31D}" src="https://github.com/user-attachments/assets/d5f2e0e7-4a4c-47e5-b400-782810f3feed" />
