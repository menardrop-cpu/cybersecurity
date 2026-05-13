# Pentest EvilCorp — Groupe Horizon Sécurité IT

![Jedha Bootcamp](https://img.shields.io/badge/Jedha-Cybersecurity%20Bootcamp-028090?style=flat-square)
![Demo Day](https://img.shields.io/badge/Demo%20Day-12%20Mai%202026-FE0162?style=flat-square)
![Niveau](https://img.shields.io/badge/Niveau-Essentials%20%2B%20Fullstack-0E3449?style=flat-square)
![Statut](https://img.shields.io/badge/Statut-Complet-10B981?style=flat-square)
![Langue](https://img.shields.io/badge/Langue-Fran%C3%A7ais-blueviolet?style=flat-square)

> Rapport de test d'intrusion réalisé dans le cadre du projet final du bootcamp Jedha Cybersecurity (Essentials + Fullstack).
> Cible : serveur lab EvilCorp SAS — 10.10.10.83 — black-box — mai 2026.

---

## Équipe

| Nom | Rôle |
|-----|------|
| DELPUECH Eddy | Pentest, reconnaissance |
| MENARD Pierre | Pentest, exploitation web, GRC |
| MARZIN Melissa | Pentest, escalade de privilèges |
| PARISOT Wilfried | Pentest, post-exploitation |
| QUESNEL Solène | Pentest, reporting |

**Groupe :** Horizon Sécurité IT | **Certification :** Jedha Cybersecurity Essentials + Fullstack (RNCP niveau 6)

---

## Résumé des findings

| ID | Vulnérabilité | Sévérité | CVSS | Statut |
|----|--------------|----------|------|--------|
| VULN-01 | Remote Code Execution (RCE) | MOYEN | 6.2 | Ouvert |
| VULN-02 | Information Disclosure (credentials en clair) | MOYEN | 5.1 | Ouvert |
| VULN-03 | Unauthenticated Access Control (FTP anonyme) | MOYEN | 6.2 | Ouvert |
| VULN-04 | SQL Injection (SQLi) | MOYEN | 6.2 | Ouvert |
| VULN-05 | Broken Privilege Isolation | MOYEN | 5.1 | Ouvert |
| VULN-06 | CRONTAB Wildcard Injection (privesc root) | CRITIQUE | 9.3 | Ouvert |
| VULN-07 | Sudoers Privilege Escalation | CRITIQUE | 9.3 | Ouvert |
| VULN-08 | SUID find Privilege Escalation | CRITIQUE | 9.3 | Ouvert |
| CVE-2007-6750 | Denial of Service (Slowloris) | ÉLEVÉ | 8.2 | Ouvert |

**Bilan :** 3 CRITIQUE, 1 ÉLEVÉ, 5 MOYEN. Compromission root démontrée par 3 chemins indépendants.

---

## Structure du repo

```
evilcorp-pentest-jedha/
├── README.md                         # Ce fichier
├── LEGAL.md                          # Avertissement légal et périmètre autorisé
├── methodology.md                    # Approche méthodologique PTES
├── findings_summary.md               # Synthèse complète des vulnérabilités
├── writeups/
│   ├── 00_reconnaissance_nmap.md     # Phase de reconnaissance
│   ├── 01_VULN03_ftp_anonymous.md    # Accès FTP anonyme et vol de clé SSH
│   ├── 02_VULN01_rce_pingozaurus.md  # Remote Code Execution web
│   ├── 03_VULN02_information_disclosure.md # Credentials en clair
│   ├── 04_VULN05_broken_privilege_isolation.md # Isolation de privilèges
│   ├── 05_VULN06_wildcard_injection.md # Wildcard injection crontab (root)
│   ├── 06_VULN07_sudoers_privesc.md  # Escalade via sudoers tee
│   ├── 07_VULN08_suid_find_privesc.md # Escalade via SUID find
│   └── 08_CVE-2007-6750_slowloris.md # DoS Slowloris
└── remediation/
    └── remediation_plan.md           # Plan de remédiation complet
```

---

## Kill chain

```
[Attaquant externe]
        |
        |-- Scan Nmap --> ports 21, 22, 80, 8081 ouverts
        |
        |-- Voie A : FTP anonyme (VULN-03)
        |       |
        |       `-- Vol clé SSH Alice --> SSH Alice
        |               |
        |               |-- sudoers tee NOPASSWD (VULN-07) --> ROOT
        |               |-- find SUID dans /home/bob (VULN-08) --> ROOT
        |               `-- Crontab wildcard tar (VULN-06) --> ROOT
        |
        `-- Voie B : RCE Pingozaurus (VULN-01)
                |
                `-- Credentials en clair john (VULN-02 + VULN-05)
                        |
                        `-- su john --> Crontab wildcard tar (VULN-06) --> ROOT
```

---

## Environnement de lab

Projet réalisé sur la machine virtuelle `little-big-ctf` via la CLI Jedha.

```bash
jedha-cli launch little-big-ctf
# IP cible : 10.10.10.83
```

Système détecté : Ubuntu 20.04.6 LTS (Linux 4.15 à 5.19), vsftpd 3.0.5, OpenSSH 8.2p1.

---

## Outils utilisés

| Outil | Version | Usage |
|-------|---------|-------|
| Nmap | 7.95 | Scan réseau, détection de services, scripts NSE |
| Client FTP | natif | Connexion anonyme, exfiltration de fichiers |
| OpenSSH | 8.2p1 | Connexion par clé privée |
| Navigateur + DevTools | Chrome | Test d'injection sur formulaire Pingozaurus |
| GTFOBins | référence | Exploitation de binaires Linux (tar, tee, find) |
| Kali Linux | 2025 | Environnement d'attaque |

---

## Références

| Ressource | URL |
|-----------|-----|
| OWASP Top 10 2021 | https://owasp.org/Top10/2021/ |
| CWE List | https://cwe.mitre.org/ |
| GTFOBins | https://gtfobins.github.io/ |
| CVSS Calculator v3.1 | https://www.first.org/cvss/calculator/3.1 |
| NVD CVE-2007-6750 | https://nvd.nist.gov/vuln/detail/CVE-2007-6750 |
| PTES | http://www.pentest-standard.org/ |
| MITRE ATT&CK | https://attack.mitre.org/ |
| ANSSI Hygiène informatique | https://cyber.gouv.fr/publications/guide-dhygiene-informatique |

---

## Avertissement

Ce projet a été réalisé dans un cadre pédagogique strictement contrôlé, sur un serveur de lab fourni par Jedha, avec un mandat signé. Toutes les techniques documentées ici ont été utilisées sur un système autorisé uniquement. La reproduction de ces techniques sur un système tiers sans autorisation explicite est illégale.

Voir [LEGAL.md](./LEGAL.md) pour le périmètre complet.
