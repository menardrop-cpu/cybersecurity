# Threat Intelligence & APT Analysis — Mon Apprentissage

**Module** : Introduction to Threat Intelligence  
**Exercice** : NorthBank Financial Group Incident Analysis  
**Date** : Mai 2026  
**Statut** : ✅ Complété

---

## Ce Que J'Ai Appris

Cet exercice m'a enseigné comment analyser une **attaque réelle** en utilisant :
- Les frameworks MITRE ATT&CK
- Les techniques de Threat Intelligence
- L'identification des APTs
- La cartographie des comportements aux TTPs

---

# PARTIE 1 : COMPRENDRE L'INCIDENT

## Résumé De L'Attaque

Une attaque sophistiquée contre **NorthBank Financial Group** attribuée à **APT29 (Cozy Bear)**, un groupe d'espionnage russe. L'objectif : accéder aux données financières sensibles via une compromission multi-étapes.

## La Chaîne D'Exploitation

```
Spear-Phishing Email
    ↓
Malicious Attachment (security_patch_update.exe)
    ↓
PowerShell Execution (T1059.001)
    ↓
Remote Payload Download
    ↓
Scheduled Task Persistence (T1053.005)
    ↓
Credential Harvesting via Kerberoasting (T1003.003)
    ↓
Lateral Movement (T1047 - WMI)
    ↓
C2 Communication (T1071.001)
    ↓
Data Exfiltration (T1041)
```

---

# PARTIE 2 : IDENTIFIER LES INDICATEURS DE COMPROMISSION (IoCs)

## Qu'Est-Ce Qu'Un IoC?

Un **Indicator of Compromise** est une pièce de preuve qu'un attaquant a été actif sur ton réseau. C'est comme une empreinte digitale laissée sur une scène de crime.

## Les IoCs Trouvés Dans L'Exercice

| Type | Valeur | Signification |
|------|--------|---------------|
| **Fichier Malveillant** | security_patch_update.exe | Attachment de phishing |
| **SHA-1 Hash** | d4411f70e0dcc2f88d74ae7251d51c6676075f6f | Identification unique du malware |
| **Domaine Malveillant** | hxxp://update-secure.com | Source du payload |
| **Serveur C2** | 185.220.101.3 | Communication attaquant-victime |
| **IP Exfiltration** | 185.220.101.3 (port 443) | Données volées envoyées ici |
| **Tâche Planifiée** | SystemUpdateChecker | Persistence mechanism |
| **Compte Compromis** | svc-finance-admin | Credentials escaladées |
| **Domaine C2** | secure-finance-check.com | Alternative masquée pour C2 |

### Pourquoi Ces IoCs Sont Critiques?

- **Hashes** : Identifient précisément le malware (impossible à changer)
- **Domaines & IPs** : Permettent de bloquer la communication C2
- **Tâches Planifiées** : Révèlent les mécanismes de persistance
- **Comptes Compromis** : Montrent où l'attaquant a escaladé ses privilèges

---

# PARTIE 3 : MAPPER LES COMPORTEMENTS AUX MITRE ATT&CK TTPs

## Étape 1 : Identifier Le Comportement

L'attaquant a exécuté une commande PowerShell encodée en Base64 :
```
powershell.exe -enc JABsAG8AYwBhAGwA...
```

**Question** : Qu'est-ce que le système essaie de faire?
→ **Réponse** : Exécuter du code en cachant son contenu via l'encoding.

## Étape 2 : Rechercher La Tactic

**Questions à poser :**
- Est-ce que l'attaquant essaie de RUN du code? → **OUI**
- Est-ce que c'est dans la phase d'exploitation initiale? → **OUI**

**Tactic Mapped** : TA0002 (Execution)

## Étape 3 : Identifier La Technique Exacte

Dans le framework MITRE, il existe une technique spécifique :
**T1059.001 — Command and Scripting Interpreter: PowerShell**

Pourquoi cette technique?
- L'attaquant utilise PowerShell (interpréteur de commandes)
- L'encoding (-enc) cache le contenu pour éviter la détection

## Étape 4 : Résultat Du Mapping

```
Comportement Observé              Tactic         Technique
═════════════════════════════════════════════════════════════
PowerShell encodé                Execution      T1059.001
Téléchargement de payload        Execution      T1105
Création de scheduled task       Persistence    T1053.005
Kerberoasting                    Credential Acc T1003.003
WMI remote execution             Lateral Movem  T1047
Encrypted C2 connection          C&C            T1071.001
Exfiltration SSH over port 443   Exfiltration   T1041
```

---

# PARTIE 4 : IDENTIFIER L'APT

## Qui Est APT29 (Cozy Bear)?

**Pays d'Origine** : 🇷🇺 Russie  
**Type** : Nation-State APT (espionnage gouvernemental)  
**Naming Convention** : 
- Crowdstrike : Cozy Bear
- FireEye : APT29

**Motivations** : 
- Cyber espionnage (intelligence gathering)
- Accès aux données sensibles gouvernementales et privées

**TTPs Caractéristiques D'APT29** :
- PowerShell obfusqué
- Persistence via scheduled tasks
- Lateral movement sophistiqué
- Exfiltration sur C2 chiffré

## Pourquoi C'Est APT29?

L'exercice dit directement : "similarities with previous attacks linked to APT29"

Les similarités incluent :
1. **Technique de phishing** : Spear-phishing sophistiquée
2. **Payload staging** : Téléchargement de payload secondaire
3. **PowerShell usage** : Encodage pour éviter la détection
4. **C2 communication** : HTTPS chiffré pour masquer la comm
5. **Data exfiltration** : Volume important de données sensibles

---

# PARTIE 5 : MITRE ATT&CK FRAMEWORK — LES TACITCS

## Les 14 Tactics Du Framework Enterprise

| ID | Tactic | Description | Utilisée Ici? |
|----|---------|-----------|----|
| TA0043 | Reconnaissance | Gather info pour planifier | ❌ Non |
| TA0042 | Resource Development | Créer infrastructure | ✅ Oui (domaine enregistré) |
| TA0001 | Initial Access | Entrer le réseau | ✅ Oui (phishing) |
| TA0002 | Execution | Exécuter du code | ✅ Oui (PowerShell) |
| TA0003 | Persistence | Maintenir l'accès | ✅ Oui (scheduled task) |
| TA0004 | Privilege Escalation | Gagner plus de droits | ✅ Oui (Kerberoasting) |
| TA0005 | Defense Evasion | Éviter la détection | ✅ Oui (encoding) |
| TA0006 | Credential Access | Voler des credentials | ✅ Oui (credential harvest) |
| TA0007 | Discovery | Scanner l'environnement | ❌ Non mentionné |
| TA0008 | Lateral Movement | Bouger dans le réseau | ✅ Oui (WMI) |
| TA0009 | Collection | Collecter les données | ✅ Oui (données financières) |
| TA0011 | Command & Control | Communiquer avec C2 | ✅ Oui (HTTPS C2) |
| TA0010 | Exfiltration | Voler les données | ✅ Oui (SSH over 443) |
| TA0040 | Impact | Détruire/disrupter | ❌ Non (c'est du vol) |

---

# PARTIE 6 : COMPRENDRE LA PYRAMID OF PAIN

## Qu'Est-Ce Que La Pyramid Of Pain?

Un concept montrant à quel point il est difficile pour un attaquant de CHANGER ses indicateurs :

```
          HARD TO CHANGE
                 ▲
                 │
          ┌──────────────┐
          │   TTPs       │  ← Tactics, Techniques, Procedures
          │ (Hardest)    │     (Prennent des années à développer)
          └──────────────┘
                 │
          ┌──────────────┐
          │   Tools      │  ← Infrastructure, malware
          │              │     (Jours à semaines pour adapter)
          └──────────────┘
                 │
          ┌──────────────┐
          │   Network    │  ← IPs, Domains
          │ Artifacts    │     (Heures à jours pour changer)
          └──────────────┘
                 │
          ┌──────────────┐
          │   File       │  ← Hashes, File names
          │  Hashes      │     (Quelques secondes pour changer)
          └──────────────┘
                 ▼
           EASY TO CHANGE
```

### Application À Notre Exercice

**Faciles à changer :**
- Hash du fichier (d4411f70...) → Changer 1 byte = nouveau hash
- Domaine (update-secure.com) → Enregistrer un nouveau domaine en quelques minutes

**Difficiles à changer :**
- TTPs (PowerShell obfusqué, Kerberoasting, persistence via scheduled task)
- Ces techniques prennent **des années** à développer

**Conclusion** : En se concentrant sur les **TTPs** plutôt que les IoCs, on construit une défense beaucoup plus robuste.

---

# PARTIE 7 : LES QUESTIONS DE L'EXERCICE RÉPONDUES

## Q1: Quel Est Le Nom Du Fichier Malveillant?
**Réponse** : security_patch_update.exe  
**Pourquoi** : Spoofé comme une mise à jour système pour leurrer l'utilisateur.

## Q2: Quel APT Est Responsable?
**Réponse** : APT29 (Cozy Bear)  
**Pourquoi** : Mêmes TTPs que les attaques précédentes d'APT29 (nation-state russe).

## Q3: Type D'Attaque Pour L'Accès Initial (MITRE ID)?
**Réponse** : T1566.001 (Spear-Phishing Attachment)  
**Pourquoi** : Email phishing + pièce jointe malveillante = accès initial.

## Q4: Hash SHA-1 Du Malware?
**Réponse** : d4411f70e0dcc2f88d74ae7251d51c6676075f6f  
**Pourquoi** : Identifiant unique du fichier malveillant pour le tracker.

## Q5: Autre Nom D'Exécutable Associé au Hash (VirusTotal)?
**Réponse** : 620d2bf14fe345eef618fdd1dac242b3a0bb65ccb75699fe00f7c671f2c1d869.exe  
**Pourquoi** : Le malware a circulé sous plusieurs noms (même payload, noms différents).

## Q6: Type De Machines Ciblées?
**Réponse** : x64  
**Pourquoi** : Le malware est compilé pour architecture 64-bit (systèmes modernes).

## Q7: MITRE Technique Pour Vol De Credentials?
**Réponse** : T1003.003 (Kerberoasting)  
**Pourquoi** : Extraction de hashes de comptes de service depuis Active Directory.

## Q8: Nom De La Tâche De Persistance?
**Réponse** : SystemUpdateChecker  
**Pourquoi** : Scheduled task qui s'exécute tous les 6h pour maintenir l'accès.

## Q9: Port Pour L'Exfiltration Chiffrée?
**Réponse** : 443  
**Pourquoi** : SSH tunnel sur HTTPS (port 443) pour masquer le trafic comme du web normal.

## Q10: Domaine Complet Du Serveur C2?
**Réponse** : secure-finance-check.com  
**Pourquoi** : Utilisé pour communiquer avec les systèmes compromis.

---

# PARTIE 8 : CE QUE J'AI APPRIS

## 1. Les IoCs Sont Des Empreintes Digitales

Chaque attaque laisse des traces :
- **Hashes** : Identification du malware
- **IPs/Domaines** : Infrastructure de l'attaquant
- **Tâches planifiées** : Mécanismes de persistance
- **Comptes compromis** : Escalade de privilèges

Rassembler ces IoCs permet de :
- Bloquer les attaques futures
- Partager l'intelligence avec d'autres organisations
- Documenter la menace

## 2. MITRE ATT&CK = Langage Commun

Au lieu de dire "l'attaquant a utilisé du PowerShell obfusqué", on dit **T1059.001**.

Avantage :
- Les équipes techniques et non-techniques comprennent
- Compatible avec les outils SIEM et EDR
- Référence mondiale pour les TTPs

## 3. Les TTPs > Les IoCs

Un hash change en 1 seconde.
Une TTP (technique) prend **des années** à développer.

Donc :
- Ne bloque pas juste les hashes
- Détecte les COMPORTEMENTS (TTPs)
- Construit des défenses génériques contre les techniques

## 4. APT29 = Espionnage D'État

APT29 n'est pas un cyber-criminel cherchant du profit.
C'est un **groupe d'espionnage gouvernemental russe**.

Implications :
- Ressources illimitées
- Compétences très avancées
- Objectif : voler de l'intelligence (données gouvernementales, financières)
- Patience : peut rester des **mois** sans être détecté

## 5. La Chaîne D'Exploitation Est Progressive

L'attaquant ne devient pas admin au jour 1.
C'est une progression :
1. Phishing → Foothold
2. PowerShell → Exécution
3. Scheduled task → Persistance
4. Kerberoasting → Escalade
5. WMI → Lateral movement
6. C2 → Communication
7. SSH tunnel → Exfiltration

Chaque étape ajoute de la complexité mais aussi réduit la détection.

---

# PARTIE 9 : APPLICATION PRATIQUE

## Si Je Trouvais Cette Attaque...

### Step 1 : Collecter Les IoCs
```
✅ Chercher tous les fichiers avec le hash d4411f70...
✅ Bloquer les IPs 185.220.101.3
✅ Bloquer les domaines malveillants
✅ Trouver toutes les instances de "SystemUpdateChecker"
```

### Step 2 : Mapper Aux TTPs
```
✅ T1059.001 → Auditer l'utilisation de PowerShell (logs)
✅ T1003.003 → Surveiller les Kerberoasting attempts
✅ T1053.005 → Chercher les scheduled tasks suspectes
✅ T1047 → Détecter les WMI calls distantes
```

### Step 3 : Défendre
```
✅ MFA sur tous les comptes de service
✅ Logging PowerShell avancé
✅ EDR pour détecter les comportements
✅ Restreindre WMI à distance
✅ Monitorer les connexions C2
```

---

# RÉSUMÉ : La Méthode Complète

```
1. ANALYSER L'INCIDENT
   ├─ Identifier les IoCs
   ├─ Tracer la progression de l'attaque
   └─ Documenter la timeline

2. MAPPER AUX MITRE ATT&CK
   ├─ Identifier les Tactics (pourquoi)
   ├─ Identifier les Techniques (comment)
   └─ Identifier les Procedures (avec quoi)

3. IDENTIFIER L'APT
   ├─ Chercher les similarités TTPs
   ├─ Vérifier les motivations
   └─ Corréler avec d'autres attaques connues

4. CRÉER L'INTELLIGENCE
   ├─ Documenter les TTPs découvertes
   ├─ Partager avec l'industrie
   └─ Mettre à jour les défenses
```

---

**Status** : ✅ EXERCICE COMPRIS ET MAÎTRISÉ

**Ce Que J'Ai Appris** : Comment analyser une attaque réelle, mapper aux TTPs, identifier les APTs

**Utilité** : 5/5 ⭐⭐⭐⭐⭐

**Application** : Incident Response, Threat Hunting, Detection Engineering

---

*La vraie victoire ce n'est pas de bloquer un hash. C'est de comprendre les COMPORTEMENTS d'un attaquant pour construire une défense qui fonctionne contre tous ses variants.* 🎯
