# CTF Jedha — Challenges 00 à 08

> Mon parcours à travers le CTF Jedha. Chaque challenge a aidé à consolider les bases de Linux et du shell scripting. Ce document est un writeup complet + mes notes personnelles.

## Vue d'ensemble

| Challenge | Concept principal | Difficulté |
|-----------|-------------------|-----------|
| 00 | Lecture basique | ⭐ |
| 01 | Navigation simple | ⭐ |
| 02 | Chemins imbriqués | ⭐ |
| 03 | Permissions de fichiers | ⭐⭐ |
| 04 | Exécution + encodage Base64 | ⭐⭐ |
| 05 | Création de fichiers + redirections | ⭐⭐ |
| 06 | Arguments de script | ⭐⭐ |
| 07 | Pipes et synchronisation | ⭐⭐⭐ |
| 08 | Filtrage avec grep | ⭐⭐ |

---

## Challenge 00 : Les bases absolues

### Consigne
Afficher le contenu d'un fichier dans le répertoire courant.

### Writeup
```bash
ls                    # Voir ce qu'il y a
00.txt  01  02  03  04  05  06  07  08

cat 00.txt            # Afficher le fichier
# Congrats, you found your first flag!
# Jedha{You_f0und_me!}
```

### Ce que j'ai appris
* **`ls`** : lister les fichiers du répertoire courant
* **`cat`** : afficher le contenu d'un fichier
* **Concept** : tout est fichiers sous Linux

### Flag
```
Jedha{You_f0und_me!}
```

---

## Challenge 01 : Navigation simple

### Consigne
Accéder au dossier 01 et lire le flag.

### Writeup
```bash
cd 01                 # Changer de répertoire
ls                    # Voir le contenu
flag.txt

cat flag.txt          # Lire le flag
# Jedha{That_w4s_quick!}
```

### Ce que j'ai appris
* **`cd`** : changer de répertoire (Change Directory)
* **Notation** : `~/01` affiche le chemin relatif au home
* **Concept** : chaque dossier est un point de départ pour naviguer

### Flag
```
Jedha{That_w4s_quick!}
```

---

## Challenge 02 : Chemins imbriqués (labyrinthe)

### Consigne
Naviguer dans une structure de dossiers profonds : `02/here/is/your/flag/flag.txt`

### Writeup
```bash
cd 02
cd here
cd is
cd your
cd flag
cat flag.txt
# Jedha{St!ll_pretty_Easy}
```

**Ou en une ligne** :
```bash
cd 02/here/is/your/flag && cat flag.txt
```

### Ce que j'ai appris
* **Chemins imbriqués** : accès par `/` successifs
* **`cd ..`** : remonter d'un niveau (je ne l'ai pas utilisé ici, mais c'est utile)
* **Concept** : l'arborescence se navigue comme une hiérarchie

### Flag
```
Jedha{St!ll_pretty_Easy}
```

---

## Challenge 03 : Le Bouclier des Permissions

### Consigne
Un fichier existe mais sans droits de lecture. Je dois le rendre lisible.

### Writeup
```bash
cd 03
cat flag.txt
# cat: can't open 'flag.txt': Permission denied

ls -l flag.txt
# ----------    1 0a3c2d6d 0a3c2d6d       129 Aug 21 07:37 flag.txt
#  ↑ ↑ ↑ ↑
#  aucun droit
```

**Le problème** : `----------` signifie que même le propriétaire n'a pas de droits.

**La solution** : ajouter le droit de lecture au propriétaire avec `chmod`.

```bash
chmod u+r flag.txt    # Ajouter lecture (r) au propriétaire (u)

cat flag.txt
# Jedha{readPermission_fl@g}
```

### Ce que j'ai appris

#### Lire les permissions avec `ls -l`
```
-rw-r--r--  1  pierre  groupe  1234  date  fichier.txt
│└─┬──┘└─┬──┘
│  │     └── Autres
│  └──────── Groupe
└─────────── Propriétaire
```

* `r` = lecture (read)
* `w` = écriture (write)
* `x` = exécution (execute)

#### chmod : modifier les permissions
```bash
chmod u+r   fichier   # Ajouter lecture au propriétaire
chmod g+w   fichier   # Ajouter écriture au groupe
chmod o-r   fichier   # Retirer lecture aux autres
chmod 755   fichier   # Mode numérique : rwxr-xr-x
```

### Concept clé
Sous Linux, être propriétaire d'un fichier te donne le pouvoir de changer ses permissions, même si elles t'interdisent l'accès. C'est une règle fondamentale de la sécurité Unix.

### Flag
```
Jedha{readPermission_fl@g}
```

---

## Challenge 04 : Exécution et Encodage

### Consigne
Un script contient le flag, mais il est encodé. Je dois l'exécuter pour le décoder.

### Writeup
```bash
cd 04
ls
# run_me.sh

cat run_me.sh
#!/bin/bash
echo "Congrats! You successfully ran this script!"
echo "Your flag is" $(echo '=0nb1Z2Xzl2X0YTZzFmQ7FGakVmS' | rev | base64 -d)
```

**Le problème** : je vois le flag encodé, mais pas le flag réel.

**Comprendre l'encodage** :
1. `echo '=0nb1Z2Xzl2X0YTZzFmQ7FGakVmS'` : émettre le texte encodé
2. `rev` : inverser la chaîne (le pipe `|` passe le résultat)
3. `base64 -d` : décoder depuis Base64

**La solution** : exécuter le script pour qu'il fasse le décodage.

```bash
chmod +x run_me.sh    # Ajouter droit d'exécution
./run_me.sh           # Exécuter (le ./ signifie "dans le répertoire courant")

# Output:
# Congrats! You successfully ran this script!
# Your flag is Jedha{Base64_is_fun}
```

**Raccourci** : copier-coller la ligne et la modifier :
```bash
echo '=0nb1Z2Xzl2X0YTZzFmQ7FGakVmS' | rev | base64 -d
# Jedha{Base64_is_fun}
```

### Ce que j'ai appris

#### `chmod +x` : ajouter l'exécution
```bash
chmod +x script.sh    # Ajouter droit d'exécution (x) pour tous
./script.sh           # Exécuter le script
bash script.sh        # Alternative : exécuter via bash
```

#### Pipes et redirections
```bash
echo 'texte' | rev    # Passer la sortie de echo à rev
#            ↑
#        le pipe (|)
```

#### Base64 : encodage réversible
```bash
echo "mon message" | base64
# bW9uIG1lc3NhZ2U=

echo "bW9uIG1lc3NhZ2U=" | base64 -d
# mon message
```

#### `rev` : inverser une chaîne
```bash
echo "hello" | rev
# olleh
```

### Concept clé
Un fichier `.sh` c'est juste du texte. Pour l'exécuter, il faut :
1. Avoir les droits d'exécution (`chmod +x`)
2. Appeler un interpréteur (bash, sh, python...)

Sans ces droits, `cat` l'affiche mais `./` échoue.

### Flag
```
Jedha{Base64_is_fun}
```

---

## Challenge 05 : Création et Manipulation de Fichiers

### Consigne
Un script me demande de créer une structure spécifique : un dossier `gimmeflag` contenant un fichier `canihaz` avec le contenu `please`.

### Writeup — Version "propre"

Suivre les instructions :

```bash
cd 05

mkdir gimmeflag                    # Créer un dossier
cd gimmeflag
echo "please" > canihaz            # Créer un fichier avec du contenu
# (le > redirige la sortie vers un fichier)
cd ..

./run_me.sh
# Your flag is Jedha{Cheezburg3r}
```

### Writeup — Version "cheater"

Lire le script pour voir comment il cherche le flag :

```bash
cat run_me.sh
# ...
# if grep 'please' /home/$USER/05/gimmeflag/canihaz 2>/dev/null; then
#     echo "Your flag is" $(echo '9J3MnJXdipXZlh2Q7FGakVmS' | rev | base64 -d)
# ...

echo "Your flag is" $(echo '9J3MnJXdipXZlh2Q7FGakVmS' | rev | base64 -d)
# Your flag is Jedha{Cheezburg3r}
```

### Ce que j'ai appris

#### `mkdir` : créer un dossier
```bash
mkdir nom_dossier
mkdir -p chemin/complet/dossier    # Créer tous les niveaux
```

#### `touch` : créer un fichier vide
```bash
touch fichier.txt
```

#### `echo` et redirections : créer/modifier des fichiers
```bash
echo "contenu" > fichier.txt       # Créer/écraser le fichier
echo "plus" >> fichier.txt         # Ajouter à la fin

# Différence
>                                  # Écrase (overwrite)
>>                                 # Ajoute (append)
```

#### `grep` : chercher une ligne contenant un motif
```bash
grep 'please' /chemin/fichier
# Retour de code : 0 si trouvé, 1 sinon
# C'est utile dans des conditions if
```

### Concept clé
Les redirections (`>` et `>>`) sont la façon la plus simple de créer/modifier des fichiers depuis le shell. C'est utilisé partout en automatisation.

### Flag
```
Jedha{Cheezburg3r}
```

---

## Challenge 06 : Les Arguments (Paramètres)

### Consigne
Passer des arguments à un script. Les paramètres corrects sont `first` et `second`.

### Writeup
```bash
cd 06

# Sans arguments : le script affiche la consigne
./params.sh
# For this challenge, you need to supply two parameters to this script :
# The first one should be 'first'
# The second one should be 'second'

# Avec les bons arguments
./params.sh first second
# Congrats, you finished this exercice! Your flag is Jedha{P4r4ms_Flag}
```

### Lire le script pour comprendre
```bash
cat params.sh
# #!/bin/bash
# echo -e "For this challenge..."
# if [ "$1" = "first" ] && [ "$2" = "second" ];then
#     echo "Congrats... Your flag is Jedha{P4r4ms_Flag}";
# fi
```

### Ce que j'ai appris

#### Les variables spéciales du shell
```bash
$0    # Nom du script lui-même
$1    # Premier argument
$2    # Deuxième argument
$3    # Troisième argument
$@    # Tous les arguments
$#    # Nombre d'arguments
```

#### Conditions en bash
```bash
if [ condition ]; then
    # commandes
fi

# Comparaisons de chaînes
[ "$1" = "first" ]        # Égalité
[ "$1" != "first" ]       # Différence

# Combinaisons
[ "$1" = "first" ] && [ "$2" = "second" ]   # ET logique
[ "$1" = "first" ] || [ "$1" = "second" ]   # OU logique
```

### Concept clé
Les arguments passés à un script sont accessibles via `$1`, `$2`, etc. C'est la base de toute automatisation shell.

### Flag
```
Jedha{P4r4ms_Flag}
```

---

## Challenge 07 : Le Pipe et la Synchronisation (⭐⭐⭐)

### Consigne
Un script génère un hash basé sur la seconde courante. Je dois :
1. Générer un hash avec `./fast.sh generate`
2. Le passer immédiatement à `./fast.sh read`
3. Si les hashes correspondent, j'obtiens le flag

**Le piège** : les hashes changent chaque seconde. Je suis trop lent à la main.

### Writeup — La naïve
```bash
./fast.sh generate
# 3e6231b7b4cc72f5ce63048d5cb0668f1de778e9

# Je copie, je colle, j'exécute read...
./fast.sh read
# Enter the current hash : 3e6231b7b4cc72f5ce63048d5cb0668f1de778e9
# [hash généré maintenant]
# Hash mismatch! You failed.
```

Trop lent. Entre-temps, le script a généré un **nouveau** hash basé sur la **nouvelle** seconde.

### Writeup — La bonne
Utiliser le **pipe** pour synchroniser les deux processus :

```bash
./fast.sh generate | ./fast.sh read
# Enter the current hash :
# Congrats! Your flag is Jedha{gotta_g0_fast}
```

**Comment ça marche** :
1. `./fast.sh generate` produit un hash → stdout
2. Le pipe `|` redirige immédiatement vers stdin de `./fast.sh read`
3. Les deux commands tournent quasiment en parallèle, pas de délai humain

### Ce que j'ai appris

#### Le pipe `|` : connecter deux commandes
```bash
commande1 | commande2
#          ↑
# La sortie (stdout) de commande1 devient l'entrée (stdin) de commande2
```

#### Exemple concret
```bash
cat fichier.txt | grep "motif"
# cat affiche le fichier
# grep filtre les lignes contenant "motif"

ps aux | grep bash
# ps aux liste tous les processus
# grep filtre pour bash uniquement
```

#### stdin/stdout/stderr
```bash
# stdout (sortie normale)
echo "hello"

# stderr (erreurs)
echo "erreur" >&2

# stdin (entrée)
read variable        # Attend de l'utilisateur
cat < fichier        # Lit depuis un fichier
command1 | command2  # command1's stdout → command2's stdin
```

### Concept clé
Le pipe est la raison d'être de Unix. C'est ce qui permet de chaîner des outils simples pour faire des choses complexes.

### Flag
```
Jedha{gotta_g0_fast}
```

---

## Challenge 08 : Filtrer l'indésirable

### Consigne
Un script génère 15 000 lignes de garbage. Le flag est dedans, quelque part.

### Writeup
```bash
cd 08

./lines_of_hell.sh letsgo
# [15 000 lignes de texte aléatoire]
# One of those lines, chosen randomly, contains your flag.
# Your flag is: Jedha{So_mAny_Lines!}
# [15 000 autres lignes]
```

**Plutôt que de lire 15 000 lignes** : utiliser `grep` pour filtrer.

En supposant que le flag contient le mot "flag" ou "Flag" :

```bash
./lines_of_hell.sh letsgo | grep -i flag
# Your flag is: Jedha{So_mAny_Lines!}
```

### Ce que j'ai appris

#### `grep` : filtrer des lignes
```bash
grep motif fichier            # Afficher les lignes contenant "motif"
grep -i motif fichier         # Case-insensitive
grep -v motif fichier         # Inverser : exclure "motif"
grep -c motif fichier         # Compter les lignes
grep "^motif" fichier         # ^ = début de ligne
grep "motif$" fichier         # $ = fin de ligne
```

#### Grep + pipe
```bash
cat fichier | grep motif
# Idem, mais via pipe

ps aux | grep firefox
# Chercher les processus firefox
```

#### Grep pour trouver des secrets
```bash
# Chercher des mots de passe
grep -ri "password" /etc/ 2>/dev/null

# Chercher une adresse IP
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" fichier

# Chercher avec une ou plusieurs variantes
grep -E "(password|passwd|pwd)" /etc/config
```

### Concept clé
Le filtrage avec `grep` est indispensable pour l'analyse de logs et de données massives. En sécu, je l'utiliserai en permanence.

### Flag
```
Jedha{So_mAny_Lines!}
```

---

## Récapitulatif des apprentissages

### Commandes essentielles maîtrisées

| Commande | Rôle | Utilisé en |
|----------|------|-----------|
| `ls` | Lister | 00, 01, 02 |
| `cd` | Naviguer | 00, 01, 02, 03... |
| `cat` | Afficher | 00, 01, 02, 03 |
| `chmod` | Modifier les permissions | 03, 04 |
| `mkdir` | Créer dossier | 05 |
| `echo` | Afficher/créer | 04, 05, 06 |
| `grep` | Filtrer | 05, 08 |
| `rev` | Inverser | 04 |
| `base64` | Encoder/décoder | 04 |

### Concepts maîtrisés

| Concept | Challenge | Importance |
|---------|-----------|-----------|
| Navigation filesystem | 00-02 | ⭐⭐⭐ |
| Permissions et droits | 03 | ⭐⭐⭐ |
| Exécution vs lecture | 04 | ⭐⭐⭐ |
| Redirections et fichiers | 05 | ⭐⭐⭐ |
| Arguments de script | 06 | ⭐⭐⭐ |
| Pipes et synchronisation | 07 | ⭐⭐⭐ |
| Filtrage de données | 08 | ⭐⭐⭐ |

### Techniques acquises

✅ Lire et modifier des fichiers depuis le terminal
✅ Créer une structure de dossiers et fichiers
✅ Exécuter et déboguer des scripts bash
✅ Chaîner des commandes avec pipes
✅ Encoder/décoder des données
✅ Filtrer des données massives
✅ Comprendre les permissions Unix

---

## Ressources personnelles pour progresser

* **Notes bash** : `bash_reminders.md`
* **Gestion des utilisateurs** : `user_management.md`
* **RTFM complet** : `rtfm_wiki.md`
* **Scripting avancé** : À la suite du CTF

---

## Flags collectés

```
Challenge 00 : Jedha{You_f0und_me!}
Challenge 01 : Jedha{That_w4s_quick!}
Challenge 02 : Jedha{St!ll_pretty_Easy}
Challenge 03 : Jedha{readPermission_fl@g}
Challenge 04 : Jedha{Base64_is_fun}
Challenge 05 : Jedha{Cheezburg3r}
Challenge 06 : Jedha{P4r4ms_Flag}
Challenge 07 : Jedha{gotta_g0_fast}
Challenge 08 : Jedha{So_mAny_Lines!}
```
