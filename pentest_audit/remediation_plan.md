# Plan de remédiation complet

## Priorité 1 — Actions immédiates (moins de 48h)

Ces quatre corrections neutralisent les 3 vulnérabilités CRITIQUE et ferment les deux chemins principaux vers root. Effort total estimé : moins de 2 heures.

### 1.1 Désactiver FTP anonyme (VULN-03)

```bash
# Éditer /etc/vsftpd.conf
sudo nano /etc/vsftpd.conf

# Modifier ou ajouter
anonymous_enable=NO

# Redémarrer
sudo systemctl restart vsftpd

# Vérification
ftp 10.10.10.83
# Name: anonymous -> doit être refusé
```

### 1.2 Corriger la crontab wildcard (VULN-06)

```bash
sudo nano /etc/crontab

# Remplacer
*/5 * * * * root cd /home/john/ && tar -zcf /home-john-backup.tgz *

# Par (avec -- avant le wildcard)
*/5 * * * * root cd /home/john/ && tar -zcf /home-john-backup.tgz -- *

# Ou mieux, chemin absolu et compte dédié sans droits root
*/5 * * * * backup_user tar -zcf /backup/home-john.tgz /home/john/
```

### 1.3 Supprimer la règle sudoers tee (VULN-07)

```bash
# Toujours utiliser visudo pour éditer sudoers
sudo visudo

# Supprimer la ligne
# (ALL : ALL) NOPASSWD: /usr/bin/tee -a *
```

### 1.4 Retirer le bit SUID de find (VULN-08)

```bash
chmod -s /home/bob/find

# Vérification
ls -la /home/bob/find
# doit afficher -rwxr-xr-x sans 's'
```

---

## Priorité 2 — Court terme (30 jours)

### 2.1 Corriger la RCE Pingozaurus (VULN-01)

Refactoriser le formulaire pour utiliser une fonction ping dédiée sans appel shell :

```python
import subprocess, re

def safe_ping(host):
    # Validation stricte : IP ou domaine uniquement
    if not re.match(r'^[a-zA-Z0-9.\-]+$', host):
        raise ValueError("Entrée invalide")
    result = subprocess.run(
        ["ping", "-c", "4", "-W", "3", host],
        capture_output=True, text=True, timeout=15
    )
    return result.stdout
```

### 2.2 Supprimer les credentials en clair (VULN-02)

```bash
# Rotation immédiate du mot de passe john
sudo passwd john

# Supprimer le script contenant les credentials
sudo rm /run/john-script.sh

# Si un script de connexion automatique est nécessaire,
# utiliser SSH par clé ou un gestionnaire de secrets
```

### 2.3 Supprimer le groupe secretgroup (VULN-05)

```bash
sudo groupdel secretgroup

# Vérification
grep secretgroup /etc/group  # doit être vide
```

### 2.4 Protection Slowloris (CVE-2007-6750)

```bash
# Activer mod_reqtimeout dans Apache
sudo a2enmod reqtimeout

# Ajouter dans /etc/apache2/conf-available/security.conf
RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500
KeepAliveTimeout 5

sudo systemctl restart apache2
```

### 2.5 SQL Injection (VULN-04)

Remplacer toutes les requêtes par des prepared statements :

```php
// Mauvais
$query = "SELECT * FROM users WHERE id = " . $_GET['id'];

// Correct
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id']]);
```

---

## Priorité 3 — Moyen terme (3 mois)

### 3.1 Supervision continue (SIEM)

Déployer Wazuh (open source) pour la détection en temps réel :

```bash
# Installation Wazuh agent
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list
apt-get update && apt-get install wazuh-agent
```

Règles à configurer :
* Alerte sur modification de /etc/sudoers
* Alerte sur création de fichiers SUID en dehors de /usr
* Alerte sur connexions FTP
* Alerte sur pic de connexions HTTP simultanées

### 3.2 Audit de retest

Programmer un audit de retest complet 3 mois après application des corrections pour valider que toutes les failles sont bien fermées et qu'aucune régression n'est apparue.

---

## Tableau récapitulatif

| ID | Action | Délai | Effort | Priorité |
|----|--------|-------|--------|----------|
| VULN-03 | Désactiver FTP anonyme | 48h | 30 min | 1 |
| VULN-06 | Corriger crontab tar | 48h | 10 min | 1 |
| VULN-07 | Supprimer règle sudoers tee | 48h | 5 min | 1 |
| VULN-08 | Retirer SUID find | 48h | 5 min | 1 |
| VULN-01 | Corriger RCE Pingozaurus | 30j | 1 à 3j dev | 2 |
| VULN-02 | Supprimer credentials en clair | 30j | 1h | 2 |
| VULN-05 | Supprimer groupe secretgroup | 30j | 2h | 2 |
| VULN-04 | Corriger SQLi | 30j | 1 à 5j dev | 2 |
| CVE-2007-6750 | Configurer mod_reqtimeout | 30j | 2h | 2 |
| SIEM | Déployer Wazuh | 3 mois | 2 à 4 semaines | 3 |
| Retest | Audit de validation | 3 mois | 1 semaine | 3 |
