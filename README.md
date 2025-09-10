# Project: Analyzing Windows Registry
# 🔍 Windows Memory Forensics – Analyse du Registry

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
#  étape 1 : Identifier le profil mémoire

Avant d’extraire ou analyser le registre, il faut connaître le **profil Windows** du dump mémoire.  
Pour cela, on utilise la commande suivante :

```bash
python2 vol.py -f /media/sf_sharedF_pc_VM/Challenge/Challenge.raw imageinfo
python2 vol.py -f /media/sf_sharedF_pc_VM/Challenge/Challenge.raw imageinfo

## <img width="526" height="202" alt="{0692C3A0-FEE0-4B4F-A34A-E35F38666365}" src="https://github.com/user-attachments/assets/5cd0c63e-f091-4144-aa91-d38e547c1b0a" />


