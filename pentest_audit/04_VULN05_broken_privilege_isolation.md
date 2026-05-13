# VULN-05 — Broken Privilege Isolation

**Sévérité :** MOYEN (CVSS 5.1)
**OWASP :** A01:2021 Broken Access Control
**CWE :** CWE-269 Improper Privilege Management
**Vecteur :** Groupe Unix secretgroup partagé entre www-data et john

---

## Description

Le service web (www-data) et l'utilisateur John appartiennent tous deux au groupe `secretgroup`. Cette configuration crée un accès croisé non intentionnel : www-data peut lire les fichiers accessibles à ce groupe, dont le script contenant les credentials de John.

---

## Découverte

Vérification de l'identité depuis le contexte www-data (via VULN-01) :

```bash
uid=33(www-data) gid=33(www-data) groups=33(www-data),1003(secretgroup)
```

Depuis le compte alice, confirmation dans /etc/group :

```bash
cat /etc/group | grep secretgroup
# secretgroup:x:1003:john,www-data
```

---

## Exploitation

Le groupe secretgroup donne à www-data un accès au fichier `/run/john-script.sh` appartenant à john. Ce pont entre le service web et le compte utilisateur est exploité dans la chaine VULN-01 → VULN-02 → VULN-06.

---

## Impact

Sans ce groupe partagé, la compromission du service web resterait isolée à www-data. Avec ce pont, un attaquant peut passer de l'application web aux droits root via une seule chaîne d'exploitation.

**Impact GRC :** Violation du principe de moindre privilège. Contrôle A.8.2 ISO 27001:2022 (droits d'accès privilégiés) non respecté.

---

## Remédiation

```bash
# Suppression du groupe secretgroup
sudo groupdel secretgroup

# Vérification
grep secretgroup /etc/group  # doit retourner vide
```

Revoir l'architecture des permissions : www-data ne doit appartenir qu'à son propre groupe.

---

## Références

* [CWE-269](https://cwe.mitre.org/data/definitions/269.html)
* [OWASP A01:2021](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)

**MITRE ATT&CK :** T1134 Access Token Manipulation, TA0004 Privilege Escalation
