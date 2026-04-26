# 🗄️ How Data Is Stored

**Formation** : Jedha Cybersécurité — Essentials  
**Module** : How Computers Work  
**Statut** : ✅ Complété

---

## 1. File System

### Définition

Un système de fichiers (file system) est la méthode que l'OS utilise pour organiser, stocker et retrouver les données sur un support physique (HDD, SSD, clé USB, carte SD).

Sans file system, un disque dur n'est qu'une longue suite d'octets sans structure. Le file system impose une organisation qui permet de répondre à des questions simples :

* Où commence ce fichier sur le disque ?
* Quelle est sa taille ?
* Qui a le droit de le lire ou de le modifier ?
* Quand a-t-il été créé ou modifié ?

### Ce que le file system gère

| Fonctionnalité | Description |
|---|---|
| **Allocation** | Décide quels blocs physiques du disque sont utilisés par quel fichier |
| **Nommage** | Associe un nom lisible à une adresse physique sur le disque |
| **Hiérarchie** | Organise les fichiers en répertoires imbriqués |
| **Permissions** | Contrôle qui peut lire, écrire ou exécuter chaque fichier |
| **Métadonnées** | Stocke les informations sur chaque fichier (taille, dates, propriétaire) |
| **Intégrité** | Certains file systems journalisent les opérations pour éviter la corruption |

### Les métadonnées d'un fichier

Chaque fichier embarque des métadonnées que le file system maintient automatiquement :

| Métadonnée | Description |
|---|---|
| Nom | Le nom visible du fichier |
| Taille | En octets |
| Type | Répertoire, fichier ordinaire, lien symbolique... |
| Permissions | Qui peut lire / écrire / exécuter |
| Propriétaire | L'utilisateur et le groupe propriétaires |
| Date de création | Timestamp de création |
| Date de modification | Dernière modification du contenu |
| Date d'accès | Dernier accès en lecture |

> **Sécurité et forensics** : les timestamps sont une source d'information critique lors d'une investigation. Un attaquant qui modifie un fichier laisse une trace dans les métadonnées. Certains outils avancés modifient artificiellement les timestamps (technique appelée timestomping) pour brouiller les pistes.

### Blocs et clusters

Un disque n'est pas adressé octet par octet mais par **blocs** (ou clusters). Un bloc est la plus petite unité d'allocation, typiquement 4 Ko.

Si un fichier fait 1 Ko, il occupe quand même un bloc entier de 4 Ko. L'espace gaspillé s'appelle la **fragmentation interne**. C'est un compromis entre efficacité d'accès et utilisation de l'espace.

### Journalisation

Les file systems modernes utilisent un **journal** : avant de modifier des données, ils enregistrent l'opération prévue dans un journal. Si le système plante pendant une écriture, il peut relire le journal au redémarrage et terminer ou annuler l'opération. Cela évite la corruption du file system.

---

## 2. Directory (Répertoire)

### Définition

Un répertoire (ou dossier) est un conteneur logique qui regroupe des fichiers et d'autres répertoires. C'est l'élément fondamental de l'organisation hiérarchique du file system.

Un répertoire n'est techniquement qu'un fichier spécial qui contient une liste de noms de fichiers et leurs adresses physiques sur le disque.

### Structure hiérarchique

Tous les file systems modernes sont organisés en **arborescence** : un répertoire racine contient des répertoires enfants, qui contiennent eux-mêmes des fichiers et d'autres répertoires.

```
Racine
├── Répertoire A
│   ├── Fichier 1
│   └── Répertoire A1
│       └── Fichier 2
├── Répertoire B
│   ├── Fichier 3
│   └── Fichier 4
└── Fichier 5
```

### Chemins absolus et relatifs

Un **chemin** (path) est l'adresse complète d'un fichier ou répertoire dans l'arborescence.

**Chemin absolu** : part de la racine du système, toujours valide quel que soit le répertoire courant.

```
Linux / macOS   :   /home/pierre/Documents/rapport.pdf
Windows         :   C:\Users\pierre\Documents\rapport.pdf
```

**Chemin relatif** : part du répertoire courant. Dépend de l'endroit où l'on se trouve.

```
Si on est dans /home/pierre :
    Documents/rapport.pdf         (descend dans Documents)
    ../autre_user/fichier.txt     (remonte d'un niveau puis descend)
```

| Symbole | Signification |
|---|---|
| `/` (Linux) ou `\` (Windows) | Séparateur de répertoires |
| `.` | Répertoire courant |
| `..` | Répertoire parent (un niveau au-dessus) |
| `~` | Répertoire home de l'utilisateur (Linux/macOS) |

### Répertoires spéciaux

**Sur Linux :**

| Répertoire | Contenu |
|---|---|
| `/` | Racine du système, tout part de là |
| `/home/` | Répertoires personnels des utilisateurs |
| `/etc/` | Fichiers de configuration système |
| `/var/log/` | Journaux système |
| `/tmp/` | Fichiers temporaires, vidés au redémarrage |
| `/bin/` et `/usr/bin/` | Binaires exécutables système |
| `/root/` | Répertoire home du superutilisateur root |
| `/proc/` | Pseudo-filesystem, infos sur les processus en cours |
| `/dev/` | Fichiers de périphériques (disques, terminaux...) |

**Sur Windows :**

| Répertoire | Contenu |
|---|---|
| `C:\` | Racine du volume principal |
| `C:\Users\` | Profils utilisateurs |
| `C:\Windows\System32\` | Fichiers système critiques |
| `C:\Program Files\` | Applications 64 bits installées |
| `C:\Temp\` | Fichiers temporaires |
| `C:\Windows\Temp\` | Fichiers temporaires système |

> **Sécurité** : les répertoires temporaires (`/tmp/` sur Linux, `C:\Temp\` et `C:\Users\<user>\AppData\` sur Windows) sont des emplacements favoris des malwares pour déposer leurs payloads. Ils sont souvent moins surveillés et accessibles en écriture sans droits élevés.

### Liens symboliques et hardlinks

**Lien symbolique** (symlink) : un fichier qui pointe vers un autre fichier ou répertoire, comme un raccourci. Si le fichier cible est supprimé, le lien est cassé.

```bash
ln -s /chemin/cible /chemin/lien    # créer un symlink sous Linux
```

**Hardlink** : une seconde entrée dans le répertoire qui pointe vers les mêmes données physiques sur le disque. La suppression d'un hardlink ne supprime pas les données tant qu'il reste au moins un hardlink actif.

> **Sécurité** : les symlinks peuvent être exploités dans des attaques de type **symlink attack** où un attaquant remplace un fichier légitime par un symlink pointant vers un fichier sensible, trompant ainsi un programme qui tourne avec des droits élevés.

---

## 3. File Systems per OS

Chaque OS a développé ses propres file systems, optimisés pour ses besoins. Comprendre leurs différences est essentiel car les outils forensics, les mécanismes de sécurité et les possibilités de récupération de données varient d'un file system à l'autre.

### Windows

**NTFS (New Technology File System)**

Le file system principal de Windows depuis Windows NT. Il a remplacé FAT32 et est utilisé sur la quasi-totalité des systèmes Windows actuels.

| Caractéristique | Détail |
|---|---|
| Taille max de fichier | 16 To (théoriquement) |
| Taille max de volume | 256 To |
| Journalisation | Oui |
| Permissions | Oui, via ACL (Access Control Lists) |
| Chiffrement | Oui, via EFS (Encrypting File System) |
| Compression | Oui, native |
| Liens symboliques | Oui |

NTFS stocke toutes les métadonnées dans une structure appelée **MFT (Master File Table)**. Chaque fichier ou répertoire a une entrée dans la MFT. En forensics, la MFT est une mine d'or : même quand un fichier est supprimé, son entrée MFT reste souvent présente et contient les métadonnées du fichier supprimé.

**FAT32 (File Allocation Table)**

File system historique, encore utilisé sur les clés USB et cartes SD pour sa compatibilité universelle.

| Limitation | Valeur |
|---|---|
| Taille max de fichier | 4 Go (bloquant pour les ISOs ou vidéos) |
| Taille max de volume | 32 Go |
| Permissions | Non |
| Journalisation | Non |

**exFAT**

Conçu comme successeur de FAT32 pour les supports amovibles. Pas de limite de taille de fichier bloquante, compatible Windows, macOS et Linux.

### Linux

**ext4 (Fourth Extended File System)**

Le file system par défaut sur la majorité des distributions Linux.

| Caractéristique | Détail |
|---|---|
| Taille max de fichier | 16 To |
| Taille max de volume | 1 Eo (exaoctet) |
| Journalisation | Oui |
| Permissions | Oui, via permissions POSIX (rwx) |
| Timestamps | Précision à la nanoseconde |
| Extents | Blocs contigus pour réduire la fragmentation |

**XFS**

Performant sur les très gros volumes et les fichiers volumineux. Utilisé par défaut sur Red Hat Enterprise Linux (RHEL) et CentOS.

**Btrfs (B-tree File System)**

File system moderne avec des fonctionnalités avancées : snapshots instantanés, sommes de contrôle des données, compression native, RAID intégré. Utilisé par défaut sur Fedora.

**tmpfs**

Pseudo-filesystem stocké entièrement en RAM. Rapide mais volatile : tout est perdu au redémarrage. Utilisé pour `/tmp/` et `/run/` sur les systèmes modernes.

### macOS

**APFS (Apple File System)**

Introduit en 2017, APFS est le file system par défaut sur macOS 10.13 et ultérieurs, ainsi que sur iOS, iPadOS et watchOS.

| Caractéristique | Détail |
|---|---|
| Chiffrement | Natif, fort (AES-XTS 128 bits) |
| Snapshots | Oui, instantanés et peu coûteux |
| Clones de fichiers | Copie sans duplication des données sur le disque |
| Sparse files | Gestion efficace des fichiers creux |
| Timestamps | Précision à la nanoseconde |

**HFS+ (HFS Plus)**

L'ancien file system d'Apple, encore présent sur les volumes non convertis et les Time Machine. APFS le remplace progressivement.

### Comparatif global

| File System | OS natif | Journalisation | Permissions | Chiffrement natif | Cas d'usage principal |
|---|---|---|---|---|---|
| NTFS | Windows | Oui | ACL | EFS | Disques système Windows |
| FAT32 | Universel | Non | Non | Non | Compatibilité maximale, petits supports |
| exFAT | Universel | Non | Non | Non | Clés USB et cartes SD modernes |
| ext4 | Linux | Oui | POSIX | Non (via dm-crypt) | Disques système Linux |
| XFS | Linux | Oui | POSIX | Non | Serveurs, gros volumes |
| Btrfs | Linux | Oui | POSIX | Non | Snapshots, usage avancé |
| APFS | macOS / iOS | Oui | POSIX | Oui | Appareils Apple modernes |
| HFS+ | macOS (legacy) | Oui | POSIX | Non | Anciens systèmes Apple |

---

## 🛡️ Applications en cybersécurité

### Forensics et analyse

| File System | Point d'intérêt forensique |
|---|---|
| NTFS | La MFT conserve les métadonnées des fichiers supprimés. Les flux ADS (Alternate Data Streams) peuvent cacher des données. |
| ext4 | Les inodes supprimés restent partiellement accessibles. Les journaux ext4 tracent les opérations récentes. |
| APFS | Les snapshots permettent de revenir à un état antérieur du système. |
| FAT32 | Pas de journalisation, récupération de fichiers supprimés relativement simple. |

### Permissions et contrôle d'accès

**Linux (permissions POSIX) :**

```
-rwxr-xr--  1  pierre  groupe  4096  Apr 26  fichier.sh
```

Décomposition :

| Champ | Valeur | Signification |
|---|---|---|
| Type | `-` | Fichier ordinaire (`d` pour répertoire) |
| Propriétaire | `rwx` | Lecture, écriture, exécution pour pierre |
| Groupe | `r-x` | Lecture et exécution pour le groupe |
| Autres | `r--` | Lecture seulement pour tous les autres |

**Windows (ACL) :**

Les ACL NTFS sont plus granulaires : on peut définir des permissions pour chaque utilisateur ou groupe avec des droits précis (lecture, écriture, modification, contrôle total, etc.).

### Attaques liées aux file systems

| Attaque | Description |
|---|---|
| **Symlink attack** | Remplacer un fichier par un symlink pour tromper un programme privilégié |
| **Directory traversal** | Utiliser `../` dans une URL ou un chemin pour sortir du répertoire autorisé |
| **ADS (Alternate Data Streams)** | Cacher des données ou du code malveillant dans un flux alternatif NTFS |
| **Timestomping** | Modifier les timestamps d'un fichier pour brouiller la timeline forensique |
| **Slack space** | Données résiduelles dans l'espace non utilisé d'un bloc alloué |

> **Directory traversal en pratique** : si une application web charge un fichier avec le chemin fourni par l'utilisateur sans validation, un attaquant peut entrer `../../etc/passwd` pour lire des fichiers en dehors du répertoire prévu. C'est l'une des vulnérabilités listées dans l'OWASP Top 10.

---

## 📋 Références

* [Linux Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)
* [NTFS Documentation Microsoft](https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview)
* [APFS Reference Apple](https://developer.apple.com/support/downloads/Apple-File-System-Reference.pdf)
* [Forensics Wiki : File Systems](https://forensics.wiki/file_systems/)
* [OWASP : Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
