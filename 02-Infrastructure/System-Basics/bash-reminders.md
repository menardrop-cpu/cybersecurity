# Bash Reminders — Rappels & Scripting

> Mes notes du module System Security (Jedha). Révision des bases Bash avec ce qui manquait dans le premier wiki : I/O, pipes, redirections, et scripting. C'est la fondation de tout ce que je vais automatiser en sécu.

## Sommaire

1. [L'arborescence Linux : ce que chaque dossier signifie](#1-larborescence-linux--ce-que-chaque-dossier-signifie)
2. [Navigation et manipulation de fichiers](#2-navigation-et-manipulation-de-fichiers)
3. [Les entrées/sorties (I/O)](#3-les-entréessorties-io)
4. [Les pipes](#4-les-pipes)
5. [Les redirections](#5-les-redirections)
6. [Scripting Bash : les bases](#6-scripting-bash--les-bases)
7. [Variables](#7-variables)
8. [Structures de contrôle](#8-structures-de-contrôle)

---

## 1. L'arborescence Linux : ce que chaque dossier signifie

Sur Linux, tout part d'un seul endroit : `/`, la **racine** (root) du système de fichiers. Pas de `C:\` ou `D:\` comme Windows : un seul arbre, tout y pend.

Les dossiers essentiels à connaître :

| Chemin | Contenu | Pourquoi je m'y intéresse |
|--------|---------|--------------------------|
| `/` | Racine, point de départ absolu | Tout commence ici |
| `/bin` | Binaires essentiels (`ls`, `cp`, `cat`...) | Commandes disponibles pour tous |
| `/etc` | Fichiers de configuration du système | Configs SSH, DNS, utilisateurs... |
| `/home` | Dossiers personnels des utilisateurs | `/home/pierre` = mon espace |
| `/dev` | Fichiers spéciaux représentant les périphériques | `/dev/null`, `/dev/sda`... |
| `/mnt` | Point de montage temporaire (clé USB, stockage réseau) | Pour accéder à des volumes externes |
| `/tmp` | Fichiers temporaires, vidés au redémarrage | Souvent accessible en écriture par tous |
| `/var` | Données variables : logs, bases de données | `/var/log/` pour les logs système |
| `/usr` | Programmes et bibliothèques installés par l'utilisateur | La plupart des outils sont là |

**Note utile** : mon dossier home (`/home/pierre`) peut s'écrire `~`. C'est un raccourci reconnu par tous les shells.

```bash
cd ~            # Va dans /home/pierre
cd ~/Documents  # Va dans /home/pierre/Documents
```

**Note sécu** : `/etc` est le premier endroit où chercher de la config sensible (mots de passe, clés, services actifs). `/tmp` est souvent utilisé comme zone de dépôt lors d'une exploitation parce qu'il est accessible en écriture sans droits particuliers.

---

## 2. Navigation et manipulation de fichiers

Rappel condensé, juste pour avoir tout sous les yeux.

```bash
# Savoir où je suis
pwd

# Se déplacer
cd /etc                    # Chemin absolu
cd Documents               # Chemin relatif (depuis où je suis)
cd ..                      # Remonter d'un niveau
cd -                       # Revenir au dossier précédent
cd ~                       # Retour au home

# Lister
ls                         # Contenu du dossier courant
ls /etc                    # Contenu d'un dossier spécifique
ls -la                     # Détails + fichiers cachés

# Lire un fichier
cat /etc/hosts

# Créer
mkdir mon-dossier
mkdir -p projets/cyber/notes    # Crée tous les niveaux d'un coup
touch fichier.txt               # Fichier vide

# Supprimer
rm fichier.txt
rm -r dossier/            # Dossier et son contenu
rmdir dossier-vide/       # Uniquement si le dossier est vide

# Copier / déplacer
cp source.txt copie.txt
cp -r dossier/ copie-dossier/
mv ancien.txt nouveau.txt       # Renommer
mv fichier.txt /tmp/            # Déplacer
```

---

## 3. Les entrées/sorties (I/O)

C'est un concept fondamental en Linux que beaucoup de gens utilisent sans vraiment comprendre ce qui se passe dessous.

### Les trois flux standard

Quand un programme tourne sur Linux, il a automatiquement trois canaux de communication ouverts :

| Flux | Nom | Fichier | Numéro | Description |
|------|-----|---------|--------|-------------|
| stdin | Entrée standard | `/dev/stdin` | 0 | Là où le programme lit ses données (clavier par défaut) |
| stdout | Sortie standard | `/dev/stdout` | 1 | Là où le programme envoie ses résultats normaux |
| stderr | Sortie d'erreur | `/dev/stderr` | 2 | Là où le programme envoie ses messages d'erreur |

Quand je tape `ls` et que je vois le résultat dans le terminal, ce qui se passe vraiment :

```
[Moi qui tape] → /dev/stdin → ls → résultat → /dev/stdout → [Mon terminal]
```

Si `ls` rencontre une erreur (dossier inexistant) :
```
ls /inexistant → message d'erreur → /dev/stderr → [Mon terminal]
```

### Les fichiers spéciaux de `/dev`

```bash
/dev/null       # Trou noir : tout ce qu'on envoie ici disparaît
/dev/urandom    # Source de données aléatoires (utile en crypto)
/dev/zero       # Produit des zéros à l'infini
/dev/stdin      # L'entrée standard
/dev/stdout     # La sortie standard
/dev/stderr     # La sortie d'erreur
```

**`/dev/null` en pratique** : je l'utilise pour jeter les messages d'erreur qui ne m'intéressent pas.

```bash
find / -name "*.conf" 2>/dev/null
#                      ↑
#             Redirige stderr (flux 2) vers /dev/null
#             → Plus de messages "Permission denied" qui polluent la sortie
```

---

## 4. Les pipes

Le pipe `|` c'est l'outil le plus puissant du shell. Il connecte la sortie d'une commande à l'entrée de la suivante.

**Principe** :
```
commande1 | commande2 | commande3
```

La sortie de `commande1` devient l'entrée de `commande2`, dont la sortie devient l'entrée de `commande3`, et ainsi de suite.

### Exemples concrets

```bash
# Trouver un utilisateur dans /etc/passwd
cat /etc/passwd | grep pierre

# Compter le nombre de lignes d'un fichier
cat /etc/passwd | wc -l

# Voir les 10 premières lignes
cat /var/log/syslog | head -10

# Voir les 10 dernières lignes
cat /var/log/syslog | tail -10

# Trier et dédupliquer
cat liste.txt | sort | uniq

# Trouver les 5 IPs qui apparaissent le plus dans un log
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -5
```

### Comment je lis un pipe

Je lis de gauche à droite comme une chaîne de traitement :

```bash
cat /etc/passwd | grep /bin/bash | cut -d: -f1
#      ↓                ↓                ↓
# Lire le fichier  Garder les lignes  Extraire le premier
#                  avec /bin/bash      champ (nom user)
```

Résultat : la liste des utilisateurs qui ont bash comme shell.

**Note sécu** : les pipes sont au cœur de l'analyse de logs, de la recherche de credentials dans des fichiers, et du traitement de résultats de scan. En CTF et en pentest, je vais en écrire des dizaines.

---

## 5. Les redirections

Là où les pipes connectent deux commandes entre elles, les redirections connectent une commande à un **fichier**.

### Redirection de sortie `>`

Envoie la sortie d'une commande vers un fichier. **Écrase** le fichier s'il existe déjà.

```bash
echo "Bonjour" > message.txt        # Crée ou écrase message.txt
ls -la > inventaire.txt             # Sauvegarde le résultat de ls
nmap 192.168.1.1 > scan.txt         # Sauvegarde un scan nmap
```

### Redirection en ajout `>>`

Ajoute la sortie à la fin d'un fichier **sans l'écraser**.

```bash
echo "Première ligne" > notes.txt
echo "Deuxième ligne" >> notes.txt   # Ajoute à la fin
echo "Troisième ligne" >> notes.txt  # Ajoute encore

cat notes.txt
# Première ligne
# Deuxième ligne
# Troisième ligne
```

### Redirection d'entrée `<`

Fournit le contenu d'un fichier comme entrée d'une commande.

```bash
cat < message.txt          # Même résultat que cat message.txt
grep "erreur" < logfile    # Chercher dans un fichier
```

En pratique je l'utilise moins souvent parce que la plupart des commandes acceptent un fichier en argument directement. Mais c'est utile dans les scripts.

### Redirection de stderr `2>`

Le `2` c'est le numéro du flux stderr.

```bash
commande 2> erreurs.txt              # Erreurs dans un fichier, sortie normale à l'écran
commande > sortie.txt 2> erreurs.txt # Sortie normale et erreurs dans des fichiers séparés
commande > tout.txt 2>&1             # Tout (stdout + stderr) dans le même fichier
commande 2>/dev/null                 # Jeter les erreurs
```

### Tableau récapitulatif

| Symbole | Action |
|---------|--------|
| `>` | Redirige stdout vers un fichier (écrase) |
| `>>` | Redirige stdout vers un fichier (ajoute) |
| `<` | Utilise un fichier comme stdin |
| `2>` | Redirige stderr vers un fichier |
| `2>/dev/null` | Jette les erreurs |
| `2>&1` | Fusionne stderr dans stdout |
| `&>` | Redirige stdout et stderr ensemble |

---

## 6. Scripting Bash : les bases

Un script Bash c'est un fichier texte qui contient une suite de commandes Bash. Au lieu de les taper une par une, je les écris une fois dans un fichier et je l'exécute autant de fois que je veux.

### Créer et exécuter un script

```bash
# Créer le fichier
touch mon_script.sh

# L'écrire (ou l'ouvrir dans un éditeur)
nano mon_script.sh

# Deux façons d'exécuter
bash mon_script.sh          # Façon explicite, toujours fonctionnelle
./mon_script.sh             # Façon directe (nécessite les droits d'exécution)

# Donner les droits d'exécution
chmod +x mon_script.sh
```

### Le shebang `#!`

La première ligne d'un script indique au système quel interpréteur utiliser. On l'appelle le **shebang**.

```bash
#!/bin/bash        # Bash
#!/bin/zsh         # Zsh (le shell par défaut sur Kali)
#!/usr/bin/python3 # Python 3
#!/usr/bin/perl    # Perl
```

Sans shebang, le script est exécuté avec le shell courant. Mieux vaut toujours le préciser pour être explicite.

```bash
#!/bin/bash

# Les lignes commençant par # sont des commentaires
# Elles ne sont pas exécutées

echo "Ce script tourne"
ls /home
whoami
```

### Exemple minimal

```bash
#!/bin/bash

echo "=== Infos système ==="
echo "Hostname : $(hostname)"
echo "Utilisateur : $(whoami)"
echo "Date : $(date)"
echo "Répertoire courant : $(pwd)"
```

---

## 7. Variables

### Déclarer et utiliser une variable

```bash
# Déclaration (pas d'espaces autour du =)
NOM="pierre"
PORT=4444
CHEMIN="/home/pierre/projets"

# Utilisation : préfixer avec $
echo "Bonjour $NOM"
echo "Je vais écouter sur le port $PORT"
echo "Mes projets sont dans $CHEMIN"
```

**Attention** : pas d'espaces autour du `=`. `NOM = "pierre"` provoque une erreur.

### Capturer la sortie d'une commande

Je peux stocker le résultat d'une commande dans une variable avec `$(...)` :

```bash
DATE_ACTUELLE=$(date)
IP_LOCALE=$(hostname -I)
UTILISATEUR=$(whoami)
NB_PROCESSUS=$(ps aux | wc -l)

echo "Date : $DATE_ACTUELLE"
echo "Mon IP : $IP_LOCALE"
echo "Je suis : $UTILISATEUR"
echo "Processus actifs : $NB_PROCESSUS"
```

L'ancienne syntaxe avec les backticks `` `...` `` fonctionne aussi mais `$(...)` est préférable : plus lisible et s'imbrique facilement.

```bash
# Imbrication possible avec $()
FICHIERS_MODIFIES=$(ls -la $(pwd))
```

### Variables spéciales utiles

```bash
$0          # Nom du script lui-même
$1, $2...   # Arguments passés au script ($1 = premier, $2 = deuxième...)
$#          # Nombre d'arguments
$?          # Code de retour de la dernière commande (0 = succès)
$$          # PID du script en cours
$@          # Tous les arguments
```

```bash
#!/bin/bash
# Usage : ./scan.sh 192.168.1.1

CIBLE=$1
echo "Je scanne $CIBLE"
nmap -sS $CIBLE
```

---

## 8. Structures de contrôle

### If — condition

```bash
if CONDITION
then
    commandes si vrai
else
    commandes si faux
fi
```

Le `fi` (if à l'envers) est **obligatoire** pour fermer le bloc.

```bash
#!/bin/bash

if pwd | grep '/home/'
then
    echo "Je suis dans un sous-dossier de /home"
else
    echo "Je ne suis pas dans /home"
fi
```

**Tests courants avec `[ ]`** :

```bash
# Comparaisons de chaînes
if [ "$NOM" = "pierre" ]         # Égalité
if [ "$NOM" != "root" ]          # Différence
if [ -z "$NOM" ]                 # Chaîne vide
if [ -n "$NOM" ]                 # Chaîne non vide

# Comparaisons numériques
if [ $AGE -eq 26 ]               # Égal
if [ $AGE -ne 26 ]               # Différent
if [ $AGE -gt 18 ]               # Supérieur
if [ $AGE -lt 100 ]              # Inférieur
if [ $AGE -ge 18 ]               # Supérieur ou égal

# Fichiers
if [ -f fichier.txt ]            # Le fichier existe
if [ -d /etc ]                   # Le dossier existe
if [ -r fichier.txt ]            # Le fichier est lisible
if [ -x script.sh ]              # Le fichier est exécutable
```

**Exemple concret** :

```bash
#!/bin/bash

CIBLE=$1

if [ -z "$CIBLE" ]
then
    echo "Usage : $0 <IP>"
    exit 1
fi

echo "Scan de $CIBLE en cours..."
nmap -sS $CIBLE > "scan_$CIBLE.txt"
echo "Résultat sauvegardé dans scan_$CIBLE.txt"
```

### While — boucle conditionnelle

Répète des commandes **tant que** la condition est vraie.

```bash
while CONDITION
do
    commandes
done
```

```bash
#!/bin/bash

REPONSE=""

while [ "$REPONSE" != "oui" ]
do
    echo "Voulez-vous continuer ? (tapez 'oui' pour continuer)"
    read REPONSE
done

echo "Continuation..."
```

`read` attend que l'utilisateur tape quelque chose et le stocke dans la variable.

**Boucle infinie avec break** :

```bash
while true
do
    echo "En attente d'une connexion..."
    sleep 5     # Attendre 5 secondes entre chaque itération
    # break pour sortir si besoin
done
```

### For — boucle sur une liste

Répète des commandes pour chaque élément d'une liste.

```bash
for VARIABLE in liste
do
    commandes (peut utiliser $VARIABLE)
done
```

```bash
# Boucle sur une liste fixe
for NOM in alice bob charlie
do
    echo "Bonjour $NOM !"
done

# Boucle sur une plage de nombres
for i in {1..10}
do
    echo "Tour $i"
done

# Boucle sur les fichiers d'un dossier
for fichier in /etc/*.conf
do
    echo "Config trouvée : $fichier"
done

# Boucle sur les lignes d'un fichier
for IP in $(cat liste_ips.txt)
do
    ping -c 1 $IP > /dev/null && echo "$IP : actif"
done
```

**Exemple pratique** : scanner une plage d'IPs

```bash
#!/bin/bash

RESEAU="192.168.1"

echo "Scan du réseau $RESEAU.0/24"

for i in {1..254}
do
    ping -c 1 -W 1 $RESEAU.$i > /dev/null 2>&1
    if [ $? -eq 0 ]
    then
        echo "$RESEAU.$i est actif"
    fi
done

echo "Scan terminé."
```

Décodage :
* `ping -c 1` : envoie un seul ping
* `-W 1` : timeout de 1 seconde
* `> /dev/null 2>&1` : jette toute la sortie (stdout + stderr)
* `$?` : code de retour de ping (0 = succès = hôte actif)

---

## Récapitulatif : flux de données

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  fichier.txt ──<── commande ──>── fichier_out.txt    │
│                        │                             │
│                        │ (pipe |)                    │
│                        ↓                             │
│                   commande2 ──>── stdout              │
│                        │                             │
│                     stderr ──2>── erreurs.txt        │
│                                                      │
└──────────────────────────────────────────────────────┘
```

| Concept | Symbole | Sens |
|---------|---------|------|
| Pipe | `cmd1 \| cmd2` | Sortie de cmd1 → Entrée de cmd2 |
| Redirection sortie | `cmd > fichier` | Sortie → fichier (écrase) |
| Redirection ajout | `cmd >> fichier` | Sortie → fichier (ajoute) |
| Redirection entrée | `cmd < fichier` | Fichier → entrée de cmd |
| Redirection erreurs | `cmd 2> fichier` | Erreurs → fichier |
| Jeter les erreurs | `cmd 2>/dev/null` | Erreurs → poubelle |
| Tout fusionner | `cmd > fichier 2>&1` | Stdout + stderr → fichier |
