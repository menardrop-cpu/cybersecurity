# Storage & Cron Jobs — Stockage et automatisation

> Mes notes du module System Security (Jedha). Comment le système gère le stockage et comment automatiser les tâches répétitives. Deux besoins fondamentaux : avoir de l'espace disque et le maintenir propre.

## Sommaire

1. [Storage : de la physique au logique](#1-storage--de-la-physique-au-logique)
2. [Les périphériques de stockage](#2-les-périphériques-de-stockage)
3. [Partitions et partitionnement](#3-partitions-et-partitionnement)
4. [Systèmes de fichiers](#4-systèmes-de-fichiers)
5. [Monter et démonter](#5-monter-et-démonter)
6. [Monitorer le stockage](#6-monitorer-le-stockage)
7. [Cron Jobs — automatiser les tâches](#7-cron-jobs--automatiser-les-tâches)
8. [Syntaxe cron en détail](#8-syntaxe-cron-en-détail)
9. [Créer et gérer des cron jobs](#9-créer-et-gérer-des-cron-jobs)
10. [Commandes de référence rapide](#10-commandes-de-référence-rapide)

---

## 1. Storage : de la physique au logique

Pour utiliser un nouveau disque dur ou SSD, il faut passer par trois étapes :

1. **Détecter le disque** : le système reconnaît le nouveau périphérique
2. **Partitionner** : découper le disque en zones logiques indépendantes
3. **Formater** : créer un système de fichiers sur chaque partition pour pouvoir stocker des fichiers

Quand j'achète un ordinateur neuf, tout ça se fait automatiquement. Mais si j'ajoute un disque interne ou si j'ai besoin de répartitionner, je dois le faire manuellement.

---

## 2. Les périphériques de stockage

### Types de stockage

| Technologie | Description | Vitesse | Fiabilité |
|-------------|-------------|---------|-----------|
| **HDD** | Disques magnétiques avec plateaux tournants | Lente | Moins fiable (pièces mobiles) |
| **SSD** | Mémoire flash NAND, pas de pièces mobiles | Rapide | Plus fiable |

### Interfaces de connexion

| Interface | Vitesse | Usage |
|-----------|---------|-------|
| **NVMe** | Très rapide (PCIe) | Disques hauts de gamme, gaming |
| **SATA** | Modérée | Disques standards |

### Identifier les disques

```bash
lsblk                       # Liste tous les disques et partitions
lsblk -f                    # Ajouter le type de filesystem
fdisk -l                    # Info détaillée sur les disques (besoin root)
```

**Nomenclature** :
* `/dev/sda`, `/dev/sdb`... : disques SATA (ancienne norme)
* `/dev/nvme0n1`... : disques NVMe (moderne)
* `/dev/loop0`... : disques virtuels (images ISO, fichiers montés)

Les chiffres à la fin (`p1`, `p2`) désignent les partitions. Ex: `/dev/sda1` = première partition du premier disque SATA.

---

## 3. Partitions et partitionnement

### Pourquoi partitionner

* **Séparation** : OS sur une partition, données sur une autre. Réinstalle l'OS sans perdre les données.
* **Performances** : réduit la fragmentation, chacune croît indépendamment
* **Sécurité** : données sensibles sur une partition séparée et chiffrée
* **Management** : redimensionne, backup, snapshot une seule partition
* **Multiboot** : plusieurs OS sur le même disque

### Cas typique de partitionnement Linux

```
/boot         512 MB      Bootloader et kernel (séparé pour sécurité)
/             20 GB       Root filesystem (OS + applications)
/home         100 GB      Données utilisateurs
/swap         4 GB        Mémoire virtuelle
```

### Outil : fdisk

```bash
sudo fdisk /dev/sdX              # Modifier le disque X
# À l'intérieur de fdisk :
m                                # Menu d'aide
g                                # Créer une nouvelle table GPT (moderne)
n                                # Créer une nouvelle partition
p                                # Afficher les partitions
d                                # Supprimer une partition
t                                # Changer le type de partition
w                                # Écrire et quitter
q                                # Quitter sans sauvegarder
```

**GPT vs MBR** :
* **MBR** : ancien, limité à 4 partitions
* **GPT** : moderne, supporte beaucoup plus de partitions et disques > 2 TB

---

## 4. Systèmes de fichiers

Un système de fichiers est la structure utilisée pour organiser et accéder aux fichiers sur disque.

### Systèmes courants

| Filesystem | Système | Usage | Notes |
|-----------|---------|-------|-------|
| **ext4** | Linux | Standard | Fiable, performant, compatible |
| **NTFS** | Windows | Windows disks | Linux peut lire/écrire avec pilotes |
| **APFS** | macOS | Apple | Linux ne peut pas accéder nativement |
| **FAT32** | Windows/Universal | Clés USB | Limité à 4 GB par fichier |
| **XFS** | Linux | Gros volumes | Performant sur très gros fichiers |
| **Btrfs** | Linux | Moderne | Snapshots, compression, vérification |

### Formater une partition

```bash
# Créer un filesystem ext4 sur une partition
sudo mkfs.ext4 /dev/sdX1

# Autres types
sudo mkfs.vfat /dev/sdX1           # FAT32
sudo mkfs.ntfs /dev/sdX1           # NTFS
sudo mkfs.xfs /dev/sdX1            # XFS
```

⚠️ **Formatage = perte de données** : c'est destructif et irréversible. Vérifier 10 fois le bon disque avant de formater.

---

## 5. Monter et démonter

### Le concept de mountpoint

Un **mountpoint** c'est un dossier où le système "attache" un filesystem. Sans mountpoint, le filesystem existe mais n'est pas accessible.

```bash
# Créer un point de montage
sudo mkdir /mnt/my_disk

# Monter la partition
sudo mount /dev/sdX1 /mnt/my_disk

# Maintenant /mnt/my_disk contient le contenu de /dev/sdX1
ls /mnt/my_disk

# Démonter
sudo umount /mnt/my_disk
```

### Après montage

```bash
# Vérifier le montage
mount | grep my_disk
df /mnt/my_disk

# Changer les droits si besoin
sudo chown user:user /mnt/my_disk
chmod 755 /mnt/my_disk
```

### Montage permanent (dans /etc/fstab)

Pour que le disque soit remonté automatiquement au démarrage :

```bash
# Éditez /etc/fstab
sudo nano /etc/fstab

# Ajoutez une ligne :
/dev/sdX1    /mnt/my_disk    ext4    defaults    0    0
# device      mountpoint      fstype  options     dump fsck_order
```

**Note sécu** : les automontages automatiques (clés USB qui se montent d'elles-mêmes) peuvent être une faille de sécurité. Les systèmes critiques désactivent souvent l'automount.

---

## 6. Monitorer le stockage

### `df` — espace disque disponible

```bash
df -h                       # Human-readable (KB, MB, GB)
df -h /                     # Espace sur la partition root
df -i                       # Nombre d'inodes (nombre max de fichiers)
```

| Colonne | Signification |
|---------|--------------|
| `Filesystem` | Disque/partition |
| `Size` | Taille totale |
| `Used` | Espace utilisé |
| `Avail` | Espace disponible |
| `Use%` | % utilisé |
| `Mounted on` | Point de montage |

**Signal d'alerte** : si `Use%` atteint 85-90%, commencer à libérer de l'espace. À 100%, le système peut crasher.

### `du` — usage par répertoire

```bash
du -sh /home                # Taille totale du dossier /home
du -sh /home/*              # Taille de chaque dossier dans /home
du -h /home | sort -hr | head -10    # Top 10 gros dossiers
```

### `ncdu` — analyse visuelle

```bash
sudo apt install ncdu
ncdu /                      # Scan le système, navigation avec flèches
ncdu -x /                   # Scan local seulement (exclure montages externes)
```

Indispensable pour identifier rapidement les gros consommateurs.

---

## 7. Cron Jobs — automatiser les tâches

**Cron** c'est le système Unix de planification de tâches. Un "cron job" c'est une commande ou un script qui s'exécute automatiquement à des heures/jours/intervalles spécifiques.

### Cas d'usage typiques

* Backups quotidiens
* Rotation des logs (supprimer les vieux logs)
* Mises à jour système
* Nettoyage des fichiers temporaires
* Monitoring et alertes
* Rapports d'analyse

### Où vivent les cron jobs

| Localisation | Usage |
|-------------|-------|
| `/var/spool/cron/crontabs/USERNAME` | Cron jobs par utilisateur (éditer avec `crontab -e`) |
| `/etc/crontab` | Cron jobs système (éditez directement) |
| `/etc/cron.d/` | Fichiers cron système individuels |
| `/etc/cron.hourly/` | Scripts qui tournent chaque heure |
| `/etc/cron.daily/` | Scripts qui tournent chaque jour |
| `/etc/cron.weekly/` | Scripts qui tournent chaque semaine |
| `/etc/cron.monthly/` | Scripts qui tournent chaque mois |

---

## 8. Syntaxe cron en détail

### Format de base

```
* * * * * /chemin/commande
│ │ │ │ │
│ │ │ │ └─ Jour de la semaine (0-7, 0=dimanche)
│ │ │ └─── Mois (1-12)
│ │ └───── Jour du mois (1-31)
│ └─────── Heure (0-23)
└───────── Minute (0-59)
```

### Caractères spéciaux

| Caractère | Signification | Exemples |
|-----------|--------------|----------|
| `*` | N'importe quelle valeur | `* * * * *` = chaque minute |
| `,` | Valeurs multiples | `0,30 * * * *` = à 0 et 30 min de chaque heure |
| `-` | Plage | `9-17 * * * *` = 9 à 17 heures |
| `/` | Pas/intervalle | `*/5 * * * *` = toutes les 5 minutes |

### Exemples concrets

```cron
0 0 * * *     # Chaque jour à minuit
0 0 * * 0     # Chaque dimanche à minuit
0 * * * *     # Chaque heure pile
*/5 * * * *   # Toutes les 5 minutes
0 2 1 * *     # Le 1er jour du mois à 2h du matin
0 9-17 * * 1-5   # Chaque jour ouvrable de 9h à 17h
0 0 1,15 * *  # Le 1er et 15e jour du mois à minuit
30 3 * * *    # Chaque jour à 3h30 du matin (bons backups)
*/30 * * * *  # Toutes les 30 minutes
0 2 * * 1     # Chaque lundi à 2h (maintenances)
```

### Raccourcis

```
@yearly     0 0 1 1 *       Une fois par an
@monthly    0 0 1 * *       Une fois par mois
@weekly     0 0 * * 0       Une fois par semaine
@daily      0 0 * * *       Une fois par jour
@hourly     0 * * * *       Une fois par heure
@reboot     -               Au démarrage du système
```

```cron
@daily /home/user/backup.sh
@reboot /usr/local/bin/start_service.sh
```

---

## 9. Créer et gérer des cron jobs

### Éditer ses cron jobs

```bash
crontab -e                  # Éditer les cron jobs de l'utilisateur courant
crontab -l                  # Lister mes cron jobs
crontab -r                  # Supprimer TOUS mes cron jobs (attention!)
sudo crontab -e             # Éditer les cron jobs de root
sudo crontab -l -u USERNAME # Voir les cron jobs d'un autre utilisateur
```

### Workflow : créer un cron job

```bash
# 1. Créer le script
cat > /home/pierre/backup.sh << 'EOF'
#!/bin/bash
tar czf /backups/backup_$(date +%Y%m%d).tar.gz /home/pierre/important/
echo "Backup completed at $(date)" >> /home/pierre/backup.log
EOF

# 2. Rendre exécutable
chmod +x /home/pierre/backup.sh

# 3. Ajouter au crontab
crontab -e
# Ajouter : 0 3 * * * /home/pierre/backup.sh

# 4. Vérifier
crontab -l
```

### Déboguer les cron jobs

**Les erreurs courantes** :
* Le script n'a pas les droits d'exécution
* Le script utilise des chemins relatifs (utiliser des chemins absolus)
* Le script dépend de variables d'environnement non définies dans cron

```bash
# Vérifier les logs du cron
grep CRON /var/log/syslog          # Linux Debian/Ubuntu
grep crond /var/log/messages        # Linux RedHat/CentOS
log stream --predicate 'process == "cron"' --level debug    # macOS

# Ajouter les logs directement dans le cron job
0 3 * * * /home/pierre/backup.sh > /home/pierre/backup.log 2>&1
#                                   ^^^^^^ stdout
#                                                    ^^^ stderr

# Tester le script manuellement
/home/pierre/backup.sh
echo $?                             # Code de sortie (0 = succès)
```

### Faire taire le cron job

```bash
# Si le script produit de la sortie, cron envoie un email
# Pour éviter les emails de cron :

0 3 * * * /home/pierre/backup.sh > /dev/null 2>&1
#          Rediriger stdout ET stderr vers /dev/null

# Ou garder les logs dans un fichier de sa choice
0 3 * * * /home/pierre/backup.sh >> /home/pierre/backup.log 2>&1
```

### Cron job système (/etc/crontab)

```bash
# Au lieu de crontab -e, éditer directement
sudo nano /etc/crontab

# Format différent : ajoute l'utilisateur qui l'exécute
0 3 * * * root /usr/local/bin/system_backup.sh
#          ↑
#          utilisateur
```

---

## 10. Commandes de référence rapide

### Storage

| Commande | Action |
|----------|--------|
| `lsblk` | Lister disques et partitions |
| `lsblk -f` | Ajouter type filesystem |
| `sudo fdisk -l` | Info détaillée tous disques |
| `df -h` | Espace disponible par mountpoint |
| `du -sh /chemin` | Taille totale d'un dossier |
| `ncdu /chemin` | Analyse visuelle interactive |
| `sudo mount /dev/X /mnt/Y` | Monter une partition |
| `sudo umount /mnt/Y` | Démonter |
| `sudo mkfs.ext4 /dev/X` | Formater en ext4 |

### Cron

| Commande | Action |
|----------|--------|
| `crontab -e` | Éditer mes cron jobs |
| `crontab -l` | Lister mes cron jobs |
| `crontab -r` | Supprimer mes cron jobs |
| `sudo crontab -e` | Éditer cron jobs root |
| `cat /etc/crontab` | Voir les cron jobs système |
| `sudo crontab -l -u user` | Cron jobs d'un utilisateur |

### Monitoring important

```bash
# Alerte si stockage > 85%
df -h | grep -E "8[5-9]%|9[0-9]%"

# Trouver les plus gros dossiers
du -sh /* | sort -hr | head -5

# Vérifier que cron jobs tournent
grep CRON /var/log/syslog | tail -20
```

---

## Points sécu à retenir

* **Disque plein = crashs** : quand `/` atteint 100%, le système plante souvent. Monitorer avec `df -h` régulièrement.

* **Montage automatique = risque** : une clé USB qui se monte d'elle-même peut contenir des malwares. Certains systèmes sensibles le désactivent.

* **Cron jobs sont invisibles** : si une cronjob malfveillante tourne, elle n'apparaît pas dans `ps aux` à moins d'être en cours. C'est parfait pour les backdoors. Toujours vérifier `/var/spool/cron/` et `/etc/cron.d/` en audit.

* **Chemins absolus dans cron** : un script qui fonctionne en ligne de commande peut échouer en cron si on utilise des chemins relatifs. Toujours utiliser des chemins absolus.

* **Cron job de root = danger** : une cron job root qui exécute un script inscriptible par un utilisateur régulier = escalade de privilèges garantie. `chmod 755` sur les scripts root.

* **Logs des cron jobs** : rediriger vers `/dev/null` fait disparaître les erreurs. Garder un fichier de log pour déboguer.
