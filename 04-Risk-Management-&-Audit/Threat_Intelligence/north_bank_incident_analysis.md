# Threat intelligence & APT analysis 

**Module**: Introduction to threat intelligence  
**Exercice**: Northbank financial group incident analysis  

---

## Ce que j'ai appris

Cet exercice m'a enseigné comment analyser une **attaque réelle** en utilisant:
- Les frameworks MITRE ATT&CK
- Les techniques de threat intelligence
- L'identification des APTs
- La cartographie des comportements aux TTPs

---

# Partie 1: Comprendre l'incident

## Résumé de l'attaque

Une attaque sophistiquée contre **Northbank financial group** attribuée à **APT29 (Cozy Bear)**, un groupe d'espionnage russe. L'objectif: accéder aux données financières sensibles via une compromission multi-étapes.

## La chaîne d'exploitation

```
Spear-phishing email
    ↓
Malicious attachment (security_patch_update.exe)
    ↓
PowerShell execution (T1059.001)
    ↓
Remote payload download
    ↓
Scheduled task persistence (T1053.005)
    ↓
Credential harvesting via Kerberoasting (T1003.003)
    ↓
Lateral movement (T1047 - WMI)
    ↓
C2 communication (T1071.001)
    ↓
Data exfiltration (T1041)
```

---

# Partie 2: Identifier les indicateurs de compromission (IoCs)

## Qu'est-ce qu'un IoC?

Un **Indicator of compromise** est une pièce de preuve qu'un attaquant a été actif sur ton réseau. C'est comme une empreinte digitale laissée sur une scène de crime.

## Les IoCs trouvés dans l'exercice

| Type | Valeur | Signification |
|------|--------|---------------|
| **Fichier malveillant** | security_patch_update.exe | Attachment de phishing |
| **SHA-1 hash** | d4411f70e0dcc2f88d74ae7251d51c6676075f6f | Identification unique du malware |
| **Domaine malveillant** | hxxp://update-secure.com | Source du payload |
| **Serveur C2** | 185.220.101.3 | Communication attaquant-victime |
| **IP exfiltration** | 185.220.101.3 (port 443) | Données volées envoyées ici |
| **Tâche planifiée** | SystemUpdateChecker | Persistence mechanism |
| **Compte compromis** | svc-finance-admin | Credentials escaladées |
| **Domaine C2** | secure-finance-check.com | Alternative masquée pour C2 |

### Pourquoi ces IoCs sont critiques?

Les hashes identifient précisément le malware (impossible à changer). Les domaines et IPs permettent de bloquer la communication C2. Les tâches planifiées révèlent les mécanismes de persistance. Les comptes compromis montrent où l'attaquant a escaladé ses privilèges.

---

# Partie 3: Mapper les comportements aux MITRE ATT&CK TTPs

## Étape 1: Identifier le comportement

L'attaquant a exécuté une commande PowerShell encodée en Base64:
```
powershell.exe -enc JABsAG8AYwBhAGwA...
```

**Question**: Qu'est-ce que le système essaie de faire?
→ **Réponse**: Exécuter du code en cachant son contenu via l'encoding.

## Étape 2: Rechercher la tactic

**Questions à poser**:
- Est-ce que l'attaquant essaie de RUN du code? → **Oui**
- Est-ce que c'est dans la phase d'exploitation initiale? → **Oui**

**Tactic mapped**: TA0002 (Execution)

## Étape 3: Identifier la technique exacte

Dans le framework MITRE, il existe une technique spécifique:
**T1059.001 — Command and scripting interpreter: PowerShell**

Pourquoi cette technique? L'attaquant utilise PowerShell (interpréteur de commandes). L'encoding (-enc) cache le contenu pour éviter la détection.

---

# Partie 4: Identifier l'APT

## Qui est APT29 (Cozy Bear)?

**Pays d'origine**: 🇷🇺 Russie  
**Type**: Nation-state APT (espionnage gouvernemental)  
**Noms alternatifs**: Cozy Bear (Crowdstrike), APT29 (FireEye), Nobelium (Microsoft)

**Motivations**:
- Cyber espionnage (intelligence gathering)
- Accès aux données sensibles gouvernementales et privées

**TTPs caractéristiques d'APT29**:
- PowerShell obfusqué
- Persistence via scheduled tasks
- Lateral movement sophistiqué
- Exfiltration sur C2 chiffré

---

# Partie 5: MITRE ATT&CK framework — les tactics

## Les 14 tactics du framework enterprise

| ID | Tactic | Description | Utilisée ici? |
|----|---------|-----------|----|
| TA0043 | Reconnaissance | Gather info pour planifier | ✗ Non |
| TA0042 | Resource development | Créer infrastructure | ✓ Oui (domaine enregistré) |
| TA0001 | Initial access | Entrer le réseau | ✓ Oui (phishing) |
| TA0002 | Execution | Exécuter du code | ✓ Oui (PowerShell) |
| TA0003 | Persistence | Maintenir l'accès | ✓ Oui (scheduled task) |
| TA0004 | Privilege escalation | Gagner plus de droits | ✓ Oui (Kerberoasting) |
| TA0005 | Defense evasion | Éviter la détection | ✓ Oui (encoding) |
| TA0006 | Credential access | Voler des credentials | ✓ Oui (credential harvest) |
| TA0007 | Discovery | Scanner l'environnement | ✗ Non mentionné |
| TA0008 | Lateral movement | Bouger dans le réseau | ✓ Oui (WMI) |
| TA0009 | Collection | Collecter les données | ✓ Oui (données financières) |
| TA0011 | Command & control | Communiquer avec C2 | ✓ Oui (HTTPS C2) |
| TA0010 | Exfiltration | Voler les données | ✓ Oui (SSH over 443) |
| TA0040 | Impact | Détruire/disrupter | ✗ Non (c'est du vol) |

---

# Partie 6: Comprendre la pyramid of pain

## Qu'est-ce que la pyramid of pain?

Un concept montrant à quel point il est difficile pour un attaquant de CHANGER ses indicateurs:

```
          HARD TO CHANGE
                 ▲
                 │
          ┌──────────────┐
          │   TTPs       │  ← Tactics, techniques, procedures
          │ (Hardest)    │     (Prennent des années à développer)
          └──────────────┘
                 │
          ┌──────────────┐
          │   Tools      │  ← Infrastructure, malware
          │              │     (Jours à semaines pour adapter)
          └──────────────┘
                 │
          ┌──────────────┐
          │   Network    │  ← IPs, domains
          │ artifacts    │     (Heures à jours pour changer)
          └──────────────┘
                 │
          ┌──────────────┐
          │   File       │  ← Hashes, file names
          │  hashes      │     (Quelques secondes pour changer)
          └──────────────┘
                 ▼
           EASY TO CHANGE
```

### Application à notre exercice

**Faciles à changer**:
- Hash du fichier (d4411f70...) → Changer 1 byte = nouveau hash
- Domaine (update-secure.com) → Enregistrer un nouveau domaine en quelques minutes

**Difficiles à changer**:
- TTPs (PowerShell obfusqué, Kerberoasting, persistence via scheduled task)
- Ces techniques prennent **des années** à développer

**Conclusion**: En se concentrant sur les **TTPs** plutôt que les IoCs, on construit une défense beaucoup plus robuste.

---

# Partie 7: Les questions de l'exercice répondues

## Q1: Quel est le nom du fichier malveillant?
**Réponse**: security_patch_update.exe  
**Pourquoi**: Spoofé comme une mise à jour système pour leurrer l'utilisateur.

## Q2: Quel APT est responsable?
**Réponse**: APT29 (Cozy Bear)  
**Pourquoi**: Mêmes TTPs que les attaques précédentes d'APT29 (nation-state russe).

## Q3: Type d'attaque pour l'accès initial (MITRE ID)?
**Réponse**: T1566.001 (Spear-phishing attachment)  
**Pourquoi**: Email phishing + pièce jointe malveillante = accès initial.

## Q4: Hash SHA-1 du malware?
**Réponse**: d4411f70e0dcc2f88d74ae7251d51c6676075f6f  
**Pourquoi**: Identifiant unique du fichier malveillant pour le tracker.

## Q5: Autre nom d'exécutable associé au hash (VirusTotal)?
**Réponse**: 620d2bf14fe345eef618fdd1dac242b3a0bb65ccb75699fe00f7c671f2c1d869.exe  
**Pourquoi**: Le malware a circulé sous plusieurs noms (même payload, noms différents).

## Q6: Type de machines ciblées?
**Réponse**: x64  
**Pourquoi**: Le malware est compilé pour architecture 64-bit (systèmes modernes).

## Q7: MITRE technique pour vol de credentials?
**Réponse**: T1003.003 (Kerberoasting)  
**Pourquoi**: Extraction de hashes de comptes de service depuis active directory.

## Q8: Nom de la tâche de persistance?
**Réponse**: SystemUpdateChecker  
**Pourquoi**: Scheduled task qui s'exécute tous les 6h pour maintenir l'accès.

## Q9: Port pour l'exfiltration chiffrée?
**Réponse**: 443  
**Pourquoi**: SSH tunnel sur HTTPS (port 443) pour masquer le trafic comme du web normal.

## Q10: Domaine complet du serveur C2?
**Réponse**: secure-finance-check.com  
**Pourquoi**: Utilisé pour communiquer avec les systèmes compromis.

---

# Partie 8: Ce que j'ai appris

## 1. Les IoCs sont des empreintes digitales

Chaque attaque laisse des traces: hashes (identification du malware), IPs/domaines (infrastructure), tâches planifiées (persistance), comptes compromis (escalade de privilèges).

Rassembler ces IoCs permet de bloquer les attaques futures, partager l'intelligence avec d'autres organisations, et documenter la menace.

## 2. MITRE ATT&CK = langage commun

Au lieu de dire "l'attaquant a utilisé du PowerShell obfusqué", on dit **T1059.001**. Les équipes techniques et non-techniques comprennent. C'est compatible avec les outils SIEM et EDR. C'est une référence mondiale pour les TTPs.

## 3. Les TTPs > les IoCs

Un hash change en 1 seconde. Une TTP (technique) prend **des années** à développer. Donc: ne bloque pas juste les hashes, détecte les COMPORTEMENTS (TTPs), construit des défenses génériques contre les techniques.

## 4. APT29 = espionnage d'état

APT29 n'est pas un cyber-criminel cherchant du profit. C'est un **groupe d'espionnage gouvernemental russe**. Ressources illimitées, compétences très avancées, objectif: voler de l'intelligence gouvernementale et financière, patience: peut rester des **mois** sans être détecté.

## 5. La chaîne d'exploitation est progressive

L'attaquant ne devient pas admin au jour 1. C'est une progression:
1. Phishing → Foothold
2. PowerShell → Exécution
3. Scheduled task → Persistance
4. Kerberoasting → Escalade
5. WMI → Lateral movement
6. C2 → Communication
7. SSH tunnel → Exfiltration

Chaque étape ajoute de la complexité mais aussi réduit la détection.

---
