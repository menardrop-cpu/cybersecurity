# Méthodologie

## Cadre : PTES (Penetration Testing Execution Standard)

L'audit a suivi une approche structurée en 5 phases, inspirée du [PTES](http://www.pentest-standard.org/).

```
Phase 1   Reconnaissance
Phase 2   Analyse des vulnérabilités
Phase 3   Exploitation
Phase 4   Post-exploitation
Phase 5   Reporting
```

---

## Phase 1 — Reconnaissance

**Objectif :** cartographier la surface d'attaque sans connaissance préalable (black-box).

Outils utilisés :

```bash
# Scan SYN rapide, ports courants
nmap -sS 10.10.10.83

# Détection de versions et OS
nmap -O -sV -v 10.10.10.83

# Scan complet tous les ports + scripts de base
nmap -sV -sC -p- -oN scan_full.txt 10.10.10.83

# Scripts de détection de vulnérabilités (non-intrusifs)
nmap --script "vuln,safe" 10.10.10.83
```

Résultats :

| Port | État | Service | Version |
|------|------|---------|---------|
| 21/tcp | ouvert | FTP | vsftpd 3.0.5 |
| 22/tcp | ouvert | SSH | OpenSSH 8.2p1 Ubuntu |
| 80/tcp | ouvert | HTTP | (Pingozaurus) |
| 8081/tcp | ouvert | HTTP | (application secondaire) |

OS détecté : Linux 4.15 à 5.19 (Ubuntu 20.04.6 LTS)

---

## Phase 2 — Analyse des vulnérabilités

**Objectif :** identifier les vecteurs exploitables sur chaque service découvert.

Approche par service :

* FTP (port 21) : test de connexion anonyme, énumération des fichiers accessibles
* SSH (port 22) : test d'authentification par clé si disponible
* HTTP (port 80) : analyse manuelle du formulaire Pingozaurus, test d'injection
* HTTP (port 8081) : analyse de l'application secondaire

Référentiels consultés :
* [OWASP Testing Guide v4.2](https://owasp.org/www-project-web-security-testing-guide/)
* [GTFOBins](https://gtfobins.github.io/) pour les binaires Linux
* [NVD](https://nvd.nist.gov/) pour les CVE identifiées par Nmap

---

## Phase 3 — Exploitation

**Objectif :** valider chaque vulnérabilité par une preuve de concept (PoC) démontrant l'impact réel.

Principe : aucune vulnérabilité n'est reportée sans validation manuelle. Chaque PoC est documenté avec la commande exacte et le résultat obtenu.

Voir les writeups dans `/writeups/` pour le détail de chaque exploitation.

---

## Phase 4 — Post-exploitation

**Objectif :** mesurer la profondeur de la compromission après un premier accès.

Actions conduites après accès initial :

```bash
# Identification du contexte utilisateur
id
whoami

# Énumération des utilisateurs du système
cat /etc/passwd

# Vérification des droits sudo
sudo -l

# Recherche des binaires SUID
find / -perm -4000 -type f 2>/dev/null

# Lecture des tâches planifiées
cat /etc/crontab

# Lecture des groupes
cat /etc/group
```

---

## Phase 5 — Reporting

**Objectif :** documenter les findings de façon accessible pour deux audiences distinctes.

| Audience | Format | Contenu |
|----------|--------|---------|
| Direction / RH | Rapport Word (synthèse managériale) | Risque business, impact, priorités |
| Équipe technique | Ce repository GitHub | Commandes, PoC, remédiation technique |
| Jury Jedha | Présentation PowerPoint | Démonstration vulnérabilité 6 |

Notation des vulnérabilités : **CVSS v3.1** via [FIRST.org Calculator](https://www.first.org/cvss/calculator/3.1).

Mapping référentiels : OWASP Top 10 2021, CWE, ISO 27001:2022 Annexe A, NIS2 Article 21.

---

## Règles d'engagement (Rules of Engagement)

Mandat signé par H. Quinn, CISO EvilCorp SAS :

* Cible : 10.10.10.83 uniquement
* DoS actif : exclu du scope
* Ingénierie sociale : exclue du scope
* Actions destructives : exclues du scope
* Données sensibles : non conservées au-delà de la preuve de concept

La vulnérabilité CVE-2007-6750 (Slowloris) a été **détectée par script Nmap non-intrusif** et non testée activement, conformément à cette exclusion.
