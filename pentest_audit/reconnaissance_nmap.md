# Reconnaissance — Scan Nmap

**Phase :** Reconnaissance
**Cible :** 10.10.10.83
**Outil :** Nmap 7.95

---

## Objectif

Cartographier la surface d'attaque sans aucune connaissance préalable de l'infrastructure (approche black-box). Identifier les ports ouverts, les services actifs, les versions, et les premières pistes de vulnérabilités.

---

## Étape 1 — Scan SYN rapide

```bash
nmap -sS 10.10.10.83
```

Résultat :

```
Starting Nmap 7.95 at 2026-05-09 03:08 EDT
Nmap scan report for 10.10.10.83
Host is up (0.0000090s latency).
Not shown: 996 closed tcp ports (reset)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
8081/tcp  open  blackice-icecap
MAC Address: A2:43:5F:EC:B3:5C (Unknown)
```

4 ports ouverts identifiés.

---

## Étape 2 — Détection de versions et OS

```bash
nmap -O -sV -v 10.10.10.83
```

Résultat :

```
PORT      STATE SERVICE  VERSION
21/tcp    open  ftp      vsftpd 3.0.5
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13
80/tcp    open  http
8081/tcp  open  blackice-icecap?

Running: Linux 4.X|5.X
OS details: Linux 4.15 - 5.19
Uptime guess: 48.211 days (since Sat Mar 21 22:29:07 2026)
```

Informations clés :
* Système : Ubuntu Linux (kernel 4.15 à 5.19)
* FTP : vsftpd 3.0.5 (version à vérifier pour failles connues)
* SSH : OpenSSH 8.2p1 (version récente, moins de CVE actifs)

---

## Étape 3 — Scan complet tous les ports

```bash
nmap -sV -sC -p- -oN scan_full.txt 10.10.10.83
```

Confirmé : uniquement 4 ports TCP ouverts sur 65535.

Points notables détectés :

```
ftp-anon: Anonymous FTP login allowed (FTP code 230)
```

**Signal fort :** le FTP accepte les connexions anonymes. Premier vecteur d'investigation.

---

## Étape 4 — Scripts de détection de vulnérabilités

```bash
nmap --script "vuln,safe" 10.10.10.83
```

Résultat notable :

```
http-slowloris-check:
  VULNERABLE:
  Slowloris DOS attack
    State: LIKELY VULNERABLE
    IDs:  CVE:CVE-2007-6750
```

Détection de la CVE-2007-6750 (Slowloris) sur le port 80. Notée mais non testée activement (DoS exclu des RoE).

---

## Synthèse de la reconnaissance

| Port | Service | Version | Piste |
|------|---------|---------|-------|
| 21 | FTP | vsftpd 3.0.5 | Accès anonyme autorisé |
| 22 | SSH | OpenSSH 8.2p1 | Connexion par clé si disponible |
| 80 | HTTP | inconnu | Application web Pingozaurus |
| 8081 | HTTP | inconnu | Application secondaire |

**Priorités d'investigation :**
1. FTP anonyme (porte d'entrée facile)
2. Application web port 80 (surface d'attaque importante)
3. Application port 8081

---

## Références

* [Nmap Documentation](https://nmap.org/book/)
* [Nmap NSE Scripts](https://nmap.org/nsedoc/)
* [vsftpd 3.0.5 Security](https://security.appspot.com/vsftpd.html)

**MITRE ATT&CK :** TA0043 Reconnaissance, T1046 Network Service Scanning
