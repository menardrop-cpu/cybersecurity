# Synthèse des findings

## Vue d'ensemble

| ID | Vulnérabilité | CVSS | Sévérité | OWASP | CWE |
|----|--------------|------|----------|-------|-----|
| VULN-01 | Remote Code Execution (RCE) | 6.2 | MOYEN | A03:2021 Injection | CWE-78 |
| VULN-02 | Information Disclosure | 5.1 | MOYEN | A02:2021 Crypto Failures | CWE-256 |
| VULN-03 | Unauthenticated Access (FTP) | 6.2 | MOYEN | A07:2021 Auth Failures | CWE-287 |
| VULN-04 | SQL Injection | 6.2 | MOYEN | A03:2021 Injection | CWE-89 |
| VULN-05 | Broken Privilege Isolation | 5.1 | MOYEN | A01:2021 Broken Access | CWE-269 |
| VULN-06 | CRONTAB Wildcard Injection | 9.3 | CRITIQUE | A03:2021 Injection | CWE-77/88 |
| VULN-07 | Sudoers Privilege Escalation | 9.3 | CRITIQUE | A01:2021 Broken Access | CWE-269/732 |
| VULN-08 | SUID find Privilege Escalation | 9.3 | CRITIQUE | A01:2021 Broken Access | CWE-250/732 |
| CVE-2007-6750 | Denial of Service (Slowloris) | 8.2 | ÉLEVÉ | A05:2021 Security Misc. | CWE-400 |

---

## Répartition par sévérité

```
CRITIQUE (9.0+)  ████████  3 findings  VULN-06, 07, 08
ÉLEVÉ    (7.0+)  ███       1 finding   CVE-2007-6750
MOYEN    (4.0+)  ████████████████  5 findings  VULN-01 à 05
FAIBLE   (0-3.9) 0 finding
```

---

## Mapping ISO 27001:2022

| Contrôle ISO | Description | Vulnérabilités concernées |
|-------------|-------------|--------------------------|
| A.5.15 | Contrôle d'accès | VULN-03, VULN-05, VULN-07 |
| A.8.2 | Droits d'accès privilégiés | VULN-06, VULN-07, VULN-08 |
| A.8.9 | Gestion de la configuration | VULN-06 |
| A.8.24 | Utilisation de la cryptographie | VULN-02 |
| A.8.28 | Codage sécurisé | VULN-01, VULN-04 |
| A.8.6 | Gestion de la capacité | CVE-2007-6750 |

---

## Mapping NIS2 (Article 21)

| Mesure NIS2 | Vulnérabilités |
|-------------|---------------|
| Hygiène cyber et formation | VULN-01, VULN-04, VULN-06 |
| Contrôle d'accès, MFA | VULN-03, VULN-05, VULN-07 |
| Gestion des secrets | VULN-02 |
| Gestion des incidents | CVE-2007-6750 |

---

## Priorités de remédiation

### Priorité 1 — Immédiat (moins de 48h)

| Action | Effort | Impact |
|--------|--------|--------|
| Désactiver FTP anonyme | 30 min | Ferme la voie A complète |
| Supprimer la règle sudoers tee NOPASSWD | 5 min | Ferme VULN-07 |
| Retirer le bit SUID sur /home/bob/find | 5 min | Ferme VULN-08 |
| Corriger la crontab tar (ajouter -- avant *) | 10 min | Ferme VULN-06 |

### Priorité 2 — Court terme (30 jours)

| Action | Effort | Impact |
|--------|--------|--------|
| Validation des entrées Pingozaurus | 1 à 3 jours dev | Ferme VULN-01 |
| Migration secrets vers gestionnaire (Vault) | 1 semaine | Ferme VULN-02 |
| Suppression groupe secretgroup | 2h | Ferme VULN-05 |
| Correction SQLi (prepared statements) | 1 à 5 jours dev | Ferme VULN-04 |
| Protection Slowloris (mod_reqtimeout) | 2h | Ferme CVE-2007-6750 |

### Priorité 3 — Moyen terme (3 mois)

| Action | Effort | Impact |
|--------|--------|--------|
| Mise en place SIEM (Wazuh) | 2 à 4 semaines | Détection continue |
| AppArmor / SELinux en enforcing | 1 semaine | Confinement systémique |
| Audit de retest | 1 semaine | Validation des corrections |

---

## Référence writeups

| ID | Writeup |
|----|---------|
| Reconnaissance | [00_reconnaissance_nmap.md](./writeups/00_reconnaissance_nmap.md) |
| VULN-03 | [01_VULN03_ftp_anonymous.md](./writeups/01_VULN03_ftp_anonymous.md) |
| VULN-01 | [02_VULN01_rce_pingozaurus.md](./writeups/02_VULN01_rce_pingozaurus.md) |
| VULN-02 | [03_VULN02_information_disclosure.md](./writeups/03_VULN02_information_disclosure.md) |
| VULN-05 | [04_VULN05_broken_privilege_isolation.md](./writeups/04_VULN05_broken_privilege_isolation.md) |
| VULN-06 | [05_VULN06_wildcard_injection.md](./writeups/05_VULN06_wildcard_injection.md) |
| VULN-07 | [06_VULN07_sudoers_privesc.md](./writeups/06_VULN07_sudoers_privesc.md) |
| VULN-08 | [07_VULN08_suid_find_privesc.md](./writeups/07_VULN08_suid_find_privesc.md) |
| CVE-2007-6750 | [08_CVE-2007-6750_slowloris.md](./writeups/08_CVE-2007-6750_slowloris.md) |
