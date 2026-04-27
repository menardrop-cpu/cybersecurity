# User Management — Gestion des utilisateurs Linux

> Mes notes du module System Security (Jedha). Comprendre la gestion des utilisateurs c'est comprendre comment Linux contrôle qui peut faire quoi. C'est la base de toute politique de sécurité sur un système Unix.

## Sommaire

1. [C'est quoi un utilisateur](#1-cest-quoi-un-utilisateur)
2. [Le fichier /etc/passwd](#2-le-fichier-etcpasswd)
3. [Créer et gérer des utilisateurs](#3-créer-et-gérer-des-utilisateurs)
4. [Les mots de passe et /etc/shadow](#4-les-mots-de-passe-et-etcshadow)
5. [Permissions et propriété des fichiers](#5-permissions-et-propriété-des-fichiers)
6. [Les groupes](#6-les-groupes)
7. [Commandes de référence rapide](#7-commandes-de-référence-rapide)

---

## 1. C'est quoi un utilisateur

Sur Linux, **tout passe par les utilisateurs**. Chaque action, chaque fichier, chaque processus appartient à un utilisateur. Ce n'est pas uniquement pour les humains : la plupart des services (Apache, MySQL, SSH...) tournent sous leur propre compte utilisateur dédié, isolé du reste.

### Les composantes d'un utilisateur

* **Identité** : un nom de login unique + un **UID** (*User ID*), identifiant numérique unique
* **Authentification** : mot de passe (ou clé SSH, MFA...)
* **Home directory** : son espace personnel (`/home/pierre`)
* **Permissions** : ce qu'il peut lire, écrire, exécuter
* **Groupes** : les ensembles dont il fait partie, qui lui confèrent des droits supplémentaires

### Les types d'utilisateurs

| Type | UID | Description |
|------|-----|-------------|
| **Root** | 0 | Superuser. Droits absolus sur tout le système. Objectif privilégié des attaquants. |
| **Utilisateurs système** | 1-999 | Comptes créés par le système pour faire tourner des services (www-data, nobody...). Sans shell de login. |
| **Utilisateurs réguliers** | 1000+ | Les vraies personnes. Droits limités. |

```bash
whoami          # Mon nom d'utilisateur courant
id              # Mon UID, GID, et tous mes groupes
id -u           # Mon UID seulement
groups          # Les groupes dont je fais partie
```

**Note sécu** : l'UID 0 = root, peu importe le nom du compte. Un compte nommé "backup" avec UID 0 a les droits root. C'est un vecteur de backdoor classique : créer un utilisateur avec un nom innocent et UID 0.

---

## 2. Le fichier /etc/passwd

C'est le fichier central de gestion des utilisateurs. Son nom est trompeur : les mots de passe ne sont **pas** ici (ils sont dans `/etc/shadow`).

```bash
cat /etc/passwd
```

### Format d'une ligne

```
root:x:0:0:root:/root:/bin/bash
 │   │ │ │  │     │       └── Shell par défaut
 │   │ │ │  │     └────────── Home directory
 │   │ │ │  └──────────────── Champ GECOS (commentaire, nom complet...)
 │   │ │ └─────────────────── GID (groupe primaire)
 │   │ └───────────────────── UID
 │   └───────────────────────  x = mot de passe dans /etc/shadow
 └───────────────────────────── Nom de login
```

### Ce que je cherche dans /etc/passwd en sécu

```bash
# Utilisateurs avec un shell réel (peuvent se connecter)
cat /etc/passwd | grep -v nologin | grep -v /bin/false

# Utilisateurs avec UID 0 (root) — s'il y en a plusieurs, c'est suspect
cat /etc/passwd | awk -F: '$3 == 0'

# Comptes avec home directory dans /home (utilisateurs humains)
cat /etc/passwd | grep '/home/'
```

**`/usr/sbin/nologin`** : shell spécial qui refuse toute connexion interactive. Utilisé pour les comptes système. Réduit la surface d'attaque en limitant les comptes qui peuvent ouvrir une session.

---

## 3. Créer et gérer des utilisateurs

### Créer un utilisateur

```bash
useradd pierre                          # Créer l'utilisateur
useradd -m pierre                       # Créer avec son home directory
useradd -m -s /bin/bash pierre          # Avec bash comme shell
useradd -m -s /bin/bash -u 1050 pierre # Avec UID spécifique
```

Après création, l'utilisateur n'a pas de mot de passe. Il faut en définir un :

```bash
passwd pierre     # Définir/changer le mot de passe de pierre
```

### Se connecter en tant qu'un autre utilisateur

```bash
su pierre         # Changer d'utilisateur (reste dans le répertoire courant)
su - pierre       # Changer d'utilisateur + charger son environnement complet
su -              # Devenir root (si j'ai son mot de passe)
sudo su -         # Devenir root via sudo
exit              # Ou Ctrl+D pour revenir à l'utilisateur précédent
```

La différence entre `su pierre` et `su - pierre` : avec le `-`, je charge tout son environnement (variables, PATH, home directory). Sans le `-`, je garde l'environnement de l'utilisateur courant.

### Modifier un utilisateur

```bash
usermod -s /bin/bash pierre             # Changer de shell
usermod -d /home/nouveau pierre         # Changer le home directory
usermod -l nouveau_nom pierre           # Renommer le compte
usermod -L pierre                       # Verrouiller le compte (désactiver le login)
usermod -U pierre                       # Déverrouiller le compte
usermod -e 2025-12-31 pierre           # Date d'expiration du compte
usermod -aG sudo pierre                 # Ajouter au groupe sudo (droits admin)
```

### Supprimer un utilisateur

```bash
userdel pierre                          # Supprimer le compte (garde le home)
userdel -r pierre                       # Supprimer le compte ET le home directory
```

---

## 4. Les mots de passe et /etc/shadow

`/etc/shadow` contient les hashes des mots de passe. Il n'est lisible que par root.

```bash
cat /etc/shadow   # Root seulement
```

### Format d'une ligne

```
luke:$y$j9T$LjlklQV...:20320:0:99999:7:::
  │     │                │    │   │   └── Jours d'avertissement avant expiration
  │     │                │    │   └────── Durée max de validité du password (jours)
  │     │                │    └────────── Durée min avant de pouvoir changer
  │     │                └─────────────── Date du dernier changement (jours depuis epoch)
  │     └──────────────────────────────── Hash du mot de passe
  └────────────────────────────────────── Nom de login
```

### Lire le champ password

| Valeur | Signification |
|--------|--------------|
| `*` | Compte sans mot de passe, login désactivé |
| `!` | Compte verrouillé |
| `$y$...` | Hash yescrypt (moderne, sécurisé) |
| `$6$...` | Hash SHA-512 |
| `$2b$...` | Hash bcrypt |
| `$1$...` | Hash MD5 (obsolète, à éviter) |

### Pourquoi les mots de passe sont hashés

Un hash c'est une transformation **à sens unique** : je peux transformer un mot de passe en hash, mais je ne peux pas retrouver le mot de passe depuis le hash. Le système ne stocke jamais le mot de passe en clair, seulement son empreinte.

À la connexion : le système hashe ce que je tape et compare avec le hash stocké. Si ça correspond, c'est bon.

**Algorithmes à connaître** :
* `yescrypt` (`$y$`) : moderne, sécurisé, difficile à bruteforcer
* `SHA-512` (`$6$`) : solide, standard actuel sur beaucoup de distros
* `bcrypt` (`$2b$`) : conçu pour être lent, résistant au bruteforce
* `MD5` (`$1$`) : obsolète, cassable rapidement avec du matériel moderne

**Note sécu** : si j'arrive à lire `/etc/shadow` sur une machine cible, j'ai les hashes. Je peux tenter de les cracker offline avec Hashcat ou John the Ripper. La qualité du mot de passe et l'algorithme utilisé déterminent la difficulté.

---

## 5. Permissions et propriété des fichiers

### Lire les permissions avec `ls -l`

```bash
ls -l fichier.txt
-rw-r--r-- 1 pierre pierre 1234 jan 1 fichier.txt
│└─┬──┘└─┬──┘└─┬──┘
│  │      │     └── Autres utilisateurs
│  │      └──────── Groupe propriétaire
│  └─────────────── Propriétaire (owner)
└────────────────── Type : - = fichier, d = dossier, l = lien
```

Chaque groupe de trois lettres : `r` (lecture), `w` (écriture), `x` (exécution), `-` (permission absente).

### chmod — modifier les permissions

**Mode symbolique** :

```bash
# Qui : u (owner), g (group), o (others), a (all)
# Action : + (ajouter), - (retirer), = (définir exactement)
# Quoi : r (read), w (write), x (execute)

chmod u+x script.sh         # Ajouter exécution pour le propriétaire
chmod g-w fichier.txt       # Retirer écriture pour le groupe
chmod o-r secret.txt        # Retirer lecture pour les autres
chmod a+r public.txt        # Lecture pour tout le monde
chmod u=rwx,g=r,o= fich     # Définir exactement : proprio tout, groupe lecture, autres rien
chmod +x script.sh          # Sans préciser qui = tous (équivalent a+x)
```

**Mode numérique** : chaque permission a une valeur, on les additionne.

| Valeur | Permission |
|--------|-----------|
| 4 | r (lecture) |
| 2 | w (écriture) |
| 1 | x (exécution) |
| 0 | aucune |

```bash
chmod 755 script.sh     # rwxr-xr-x : proprio tout, groupe+autres lecture+exec
chmod 644 fichier.txt   # rw-r--r-- : proprio lecture+écriture, autres lecture
chmod 600 secret.txt    # rw------- : proprio seulement, les autres : rien
chmod 700 monscript.sh  # rwx------ : proprio tout, les autres : rien
chmod 777 fichier.txt   # rwxrwxrwx : tout le monde tout (dangereux)
```

**Permissions courantes à mémoriser** :

| Octal | Symbolique | Usage typique |
|-------|-----------|---------------|
| `755` | `rwxr-xr-x` | Scripts, binaires, dossiers publics |
| `644` | `rw-r--r--` | Fichiers de config, pages web |
| `600` | `rw-------` | Fichiers privés (clés SSH, configs sensibles) |
| `700` | `rwx------` | Scripts/dossiers strictement privés |
| `400` | `r--------` | Lecture seule stricte (clés SSH privées) |

```bash
# Récursif : appliquer à un dossier et tout son contenu
chmod -R 755 /var/www/html/
```

### chown — changer le propriétaire

```bash
chown pierre fichier.txt              # Changer le propriétaire
chown pierre:developpeurs fichier.txt # Changer propriétaire ET groupe
chown :developpeurs fichier.txt       # Changer le groupe seulement
chown -R pierre:pierre /home/pierre/ # Récursif
```

**Note sécu** : les permissions sont un vecteur d'escalade de privilèges classique. Un fichier avec des droits mal configurés (ex: un script root accessible en écriture par tous) peut permettre à un attaquant d'injecter du code qui sera exécuté en root. `chmod 777` sur n'importe quoi de sensible c'est une faute grave.

---

## 6. Les groupes

Les groupes permettent de gérer les permissions pour plusieurs utilisateurs à la fois plutôt que de les configurer un par un.

### Types de groupes

* **Groupe primaire** : groupe par défaut de l'utilisateur, mentionné dans `/etc/passwd`. Les fichiers créés par l'utilisateur appartiennent à ce groupe.
* **Groupes supplémentaires** : groupes additionnels qui donnent des droits supplémentaires.

```bash
groups                    # Mes groupes
groups pierre             # Les groupes de pierre
id pierre                 # UID, GID, et tous les groupes avec leurs IDs
```

### Le fichier /etc/group

```bash
cat /etc/group
```

Format :
```
developpeurs:x:1005:pierre,alice,bob
     │        │  │      └── Membres du groupe
     │        │  └───────── GID
     │        └──────────── Mot de passe (x = dans /etc/gshadow, rarement utilisé)
     └─────────────────────── Nom du groupe
```

### Créer et gérer des groupes

```bash
groupadd developpeurs           # Créer un groupe
groupadd -g 1050 devops         # Créer avec GID spécifique
groupmod -n devs developpeurs   # Renommer un groupe
groupmod -g 1005 devs           # Changer le GID
groupdel devs                   # Supprimer un groupe
```

### Gérer les utilisateurs dans les groupes

```bash
# Ajouter un utilisateur à un groupe supplémentaire
usermod -aG developpeurs pierre    # -a = append, -G = groupe supplémentaire
usermod -aG sudo pierre            # Donner les droits sudo
usermod -aG docker pierre          # Accès au daemon Docker

# Retirer un utilisateur d'un groupe (pas de commande directe simple)
# Éditer /etc/group manuellement ou utiliser gpasswd
gpasswd -d pierre developpeurs     # Retirer pierre du groupe
```

**Attention** : `usermod -G groupe pierre` sans le `-a` **remplace** tous les groupes supplémentaires de pierre. Toujours utiliser `-aG` pour ajouter sans écraser.

### Cas concret : gérer les droits sur un projet

```bash
# Contexte : dossier projet partagé entre alice et bob

groupadd projet-cyber
usermod -aG projet-cyber alice
usermod -aG projet-cyber bob

mkdir /opt/projet-cyber
chown root:projet-cyber /opt/projet-cyber
chmod 770 /opt/projet-cyber      # rwxrwx--- : groupe peut tout faire, autres : rien

# Alice et Bob peuvent maintenant lire/écrire dans le dossier
# Les autres utilisateurs n'y ont pas accès
```

---

## 7. Commandes de référence rapide

### Utilisateurs

| Commande | Action |
|----------|--------|
| `whoami` | Mon nom d'utilisateur |
| `id` | Mon UID, GID, groupes |
| `id -u` | Mon UID seulement |
| `cat /etc/passwd` | Tous les utilisateurs du système |
| `useradd -m -s /bin/bash nom` | Créer un utilisateur complet |
| `passwd nom` | Définir/changer un mot de passe |
| `usermod -s /bin/bash nom` | Changer le shell |
| `usermod -aG groupe nom` | Ajouter à un groupe |
| `usermod -L nom` | Verrouiller un compte |
| `usermod -U nom` | Déverrouiller un compte |
| `userdel -r nom` | Supprimer utilisateur + home |
| `su - nom` | Se connecter en tant que nom |

### Permissions

| Commande | Action |
|----------|--------|
| `ls -la` | Voir les permissions + fichiers cachés |
| `chmod 755 fichier` | rwxr-xr-x |
| `chmod 644 fichier` | rw-r--r-- |
| `chmod 600 fichier` | rw------- |
| `chmod u+x fichier` | Ajouter exec au propriétaire |
| `chmod -R 755 dossier/` | Récursif |
| `chown pierre fichier` | Changer le propriétaire |
| `chown pierre:groupe fichier` | Changer proprio et groupe |

### Groupes

| Commande | Action |
|----------|--------|
| `groups` | Mes groupes |
| `groups nom` | Groupes d'un utilisateur |
| `cat /etc/group` | Tous les groupes du système |
| `groupadd nom` | Créer un groupe |
| `groupmod -n nouveau ancien` | Renommer un groupe |
| `groupdel nom` | Supprimer un groupe |
| `gpasswd -d user groupe` | Retirer un user d'un groupe |

### Mots de passe et shadow

| Fichier | Contenu | Accès |
|---------|---------|-------|
| `/etc/passwd` | Utilisateurs, UID, GID, home, shell | Tout le monde (lecture) |
| `/etc/shadow` | Hashes des mots de passe | Root seulement |
| `/etc/group` | Groupes et membres | Tout le monde (lecture) |
| `/etc/gshadow` | Mots de passe de groupe | Root seulement |

---

## Points sécu à retenir

* `/etc/shadow` readable = j'ai les hashes. Hashcat peut tenter de les cracker offline.
* Un compte avec UID 0 mais un nom différent de "root" = backdoor potentielle.
* `chmod 777` sur quoi que ce soit de sensible = vecteur d'escalade de privilèges.
* Les comptes avec `/nologin` ou `/false` comme shell ne peuvent pas ouvrir de session interactive. Moins de comptes loginables = surface d'attaque réduite.
* `usermod -aG sudo user` donne les droits root via sudo. C'est l'équivalent d'une escalade de privilèges permanente sur la machine.
