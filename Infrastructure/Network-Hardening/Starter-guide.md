# Network Hardening — Guide Complet — Mon Apprentissage

**Module** : Network Hardening  
**Objectif** : Sécuriser un réseau en profondeur  

---

## Aperçu Du Cours

Ce module couvre la **défense réseau** :
1. **Firewalls** (contrôle du trafic)
2. **Encryption & VPN** (chiffrement des données)
3. **Monitoring** (surveillance continue)
4. **Threat Intelligence** (analyse des menaces)

---

# PARTIE 1 : Firewalls

## Concept

**Firewall** = barrière de sécurité entre réseau interne et internet.

Fonctionne sur :
* **Layer 3** (réseau) : filtre par IP
* **Layer 4** (transport) : filtre par port/protocole
* **Layer 7** (application) : analyse le contenu

## Règles De Base

### Syntaxe D'Une Règle

```
Source IP: 192.168.10.0/24
Destination IP: 192.168.1.10
Source Port: Any
Destination Port: 3306
Protocol: TCP
Action: ALLOW
```

**Traduction** :
* Autoriser les paquets
* De 192.168.10.0/24
* Vers 192.168.1.10
* Port 3306 (MySQL)
* Protocole TCP

### Ordre Des Règles

⚠️ **IMPORTANT** :
* La PREMIÈRE règle qui correspond = appliquée
* Les autres règles sont ignorées
* Si aucune règle correspond = BLOCKED par défaut

**Exemple** :
```
Règle 1 : ALLOW TCP 192.168.10.0/24 → 192.168.1.10:3306
Règle 2 : DENY TCP 192.168.0.0/16 → 192.168.1.10:3306
```

Un paquet de 192.168.10.1 → 192.168.1.10:3306 :
→ Correspond à la règle 1 = ALLOW (règle 2 ignorée)

## Best Practices Firewall

### 1. Principle Of Least Privilege

* Autoriser seulement ce qui est nécessaire
* Bloquer tout par défaut
* Ajouter les exceptions explicitement

**Exemple** :
```
Mauvais :
ALLOW all traffic
DENY 192.168.50.0/24

Bon :
DENY all (défaut)
ALLOW 192.168.10.0/24 → 192.168.1.10:3306
ALLOW 192.168.20.0/24 → 192.168.1.11:5432
```

### 2. Segmentation Réseau

Diviser le réseau en **zones** :

```
INTERNET
   ↓
┌──────────┐
│ DMZ      │  (serveurs web publics)
└──────────┘
   ↓
┌──────────┐
│ INTERNAL │  (serveurs internes)
└──────────┘
   ↓
┌──────────┐
│ CRITICAL │  (données sensibles)
└──────────┘
```

Règles entre zones :
```
Internet → DMZ : ALLOW (port 80, 443)
DMZ → Internal : ALLOW (port 3306)
Internal → Critical : ALLOW (admin seulement)
```

### 3. Ordre Des Règles

Placer les règles **spécifiques avant générales** :

```
GOOD :
Règle 1 : ALLOW 192.168.10.5 → 10.0.0.1:22 (spécifique)
Règle 2 : DENY 192.168.10.0/24 → 10.0.0.1:22 (général)

BAD :
Règle 1 : DENY 192.168.10.0/24 → 10.0.0.1:22 (bloquerait aussi 192.168.10.5)
Règle 2 : ALLOW 192.168.10.5 → 10.0.0.1:22 (jamais atteinte)
```

### 4. Logging & Monitoring

Activer les logs pour :
* Tous les paquets refusés
* Les connexions suspectes
* Les violations de règles

### 5. Maintenance Régulière

* Revoir les règles chaque trimestre
* Supprimer les règles obsolètes
* Documenter chaque règle

---

# PARTIE 2 : Encryption & VPN

## Cryptographie Symétrique (AES)

### Concept

Même clé pour chiffrer ET déchiffrer.

```
Message : "Bonjour"
Clé : "secret123"

Chiffrement :
Message + Clé → Ciphertext "X7fG9kL2mN"

Déchiffrement :
Ciphertext + Clé → "Bonjour"
```

**Usage** :
* Chiffrer des fichiers locaux
* Protéger les données stockées

**Problème** :
* Si quelqu'un a la clé → données compromises
* Comment partager la clé de façon sécurisée?

## Cryptographie Asymétrique (RSA)

### Concept

**Deux clés différentes** :
* **Public Key** (partagée avec tout le monde)
* **Private Key** (gardée secrète)

```
Message : "Bonjour"
Public Key de Bob : "ABC123"

Chiffrement :
Message + Public Key → Ciphertext "X7fG9kL2mN"

Bob déchiffre :
Ciphertext + Private Key → "Bonjour"
```

### Comment Ça Marche (Simplifié)

1. Choisir deux nombres premiers : P=61, Q=53
2. Calculer n = P × Q = 3233
3. Calculer T = (P-1) × (Q-1) = 3120
4. Choisir e (public exponent) = 17
5. Calculer d (private exponent) où d×e mod T = 1 → d=2753

**Chiffrement** :
```
c = m^e mod n
```

**Déchiffrement** :
```
m = c^d mod n
```

**Usage** :
* HTTPS (chiffrement web)
* Transmission de données sécurisée

## VPN (Virtual Private Network)

### Concept

Crée un tunnel chiffré sur internet.

```
Mon Ordinateur
      ↓
[VPN Client] ← → [VPN Server]
      ↓
Internet (chiffré)
      ↓
Destination finale
```

**Résultat** :
* Connexion chiffrée
* IP masquée
* Données protégées

### Types De VPN

**Remote Access VPN** :
* Utilisateur seul se connecte au réseau privé
* Exemple : employé de maison accédant au réseau de l'entreprise

**Site-to-Site VPN** :
* Deux réseaux complets se connectent
* Exemple : bureau Paris + bureau Londres

### Protocoles VPN

| Protocole | Sécurité | Performance | Compatibilité |
|-----------|----------|-------------|---------------|
| IPsec | Excellente | Bonne | Ancienne |
| OpenVPN | Très bonne | Bonne | Large |
| WireGuard | Excellente | Excellente | Moderne |
| L2TP/IPsec | Bonne | Bonne | Large |
| PPTP | Faible | Bonne | Très large |

### WireGuard (Modern VPN)

**Avantages** :
* Code simple (5000 lignes vs 400000 pour OpenVPN)
* Très rapide
* Cryptographie moderne (Curve25519, ChaCha20)
* Facile à configurer

**Comment Ça Marche** :
1. Exchange public keys
2. Établir tunnel chiffré
3. Transmettre données

---

# PARTIE 3 : Monitoring

## Concept

Surveiller le réseau et les machines pour détecter les attaques.

## Network Security Monitoring (SIEM)

### Qu'est-ce Qu'Un SIEM?

**SIEM** = Security Information and Event Management

Collecte les logs de TOUT :
* Firewalls
* Serveurs web
* Bases de données
* Active Directory
* Etc.

### Workflow SIEM

```
1. COLLECT (Collecte)
   ↓
   Logs from: firewall, servers, DB, AD
   
2. NORMALIZE (Normaliser)
   ↓
   Convert à format standard
   
3. AGGREGATE (Agréger)
   ↓
   Group similar events
   
4. ANALYZE (Analyser)
   ↓
   Find patterns, detect anomalies
   
5. ALERT (Alerter)
   ↓
   Notify security team
```

### Exemples De Détection

```
Brute Force SSH :
- 100 tentatives de login échouées
- De la même IP
- En 5 minutes
→ ALERT : Possible SSH brute force!

SQL Injection :
- Requête HTTP avec "UNION SELECT"
- Vers une base de données
→ ALERT : Possible SQL injection!

Déni de Service :
- 10,000 paquets/seconde
- Vers le même port
- De différentes IPs
→ ALERT : Possible DDoS!
```

## Endpoint Security Monitoring (EDR)

### Qu'est-ce Qu'Un EDR?

**EDR** = Endpoint Detection and Response

Agent sur chaque machine qui :
* Enregistre TOUTES les activités
* Détecte les malwares
* Peut bloquer les processus malveillants

### Menaces Détectées Par EDR

* **Malwares** (viruses, trojans, keyloggers)
* **Ransomware** (blocage de fichiers)
* **APTs** (Advanced Persistent Threats)
* **0-days** (exploits inconnus)

### Workflow EDR

```
Agent sur machine
    ↓
Collect ALL activities (processus, fichiers, réseau)
    ↓
Send to EDR server
    ↓
Analyze contre les règles
    ↓
Si violation:
   - Log l'activité
   - Alert analyst
   - Bloquer le processus (optionnel)
```

---

# PARTIE 4 : Threat Intelligence

## Concept

**Threat Intelligence** = Analyser les menaces pour les prévenir.

Workflow :
```
Collect IOCs → Process → Analyze → Disseminate → Prevent
```

## IOCs (Indicators Of Compromise)

Les "preuves" qu'un attaque a eu lieu :

* **File Hashes** : MD5/SHA256 d'un malware
* **IPs** : Serveurs C&C (Command & Control)
* **Domains** : Noms de domaine malveillants
* **URLs** : Pages de phishing

**Exemple** :
```
IOC : 192.168.1.100
Signification : C&C server du malware Ratankba
Action : Bloquer tout trafic vers cette IP
```

## TTPs (Tactics, Techniques, Procedures)

Les **méthodes d'attaque** d'un groupe.

**Exemple - Lazarus Group** :
```
Tactic : Reconnaissance
Technique : Social engineering via email
Procedure : Envoyer un email avec lien malveillant
           vers des sites financiers
Result : Infection du système
```

## Threat Intelligence Lifecycle

### 1. Collection

Collecter les IOCs et TTPs de :
* Malware samples
* Logs des attaques
* Threat feeds publiques
* Partenaires gouvernementaux

### 2. Processing

Normaliser et organiser avec des **tags** :

```
File Hash: X7fG9kL2mN
Tags: malware, ransomware, windows
Family: Ratankba
Variants: 5
First seen: 2024-01-15
```

### 3. Analysis

Chercher les patterns :

```
Attaque 1 : Phishing email → Excel malveillant
Attaque 2 : Phishing email → Word malveillant
Attaque 3 : Phishing email → PDF malveillant

Pattern : Toutes les attaques utilisent phishing
TTP : Phishing is the initial access method
```

### 4. Dissemination

Partager avec les équipes :
* **SIEM** : intégrer les IOCs pour détection
* **EDR** : signaler les fichiers malveillants
* **Firewall** : bloquer les IPs malveillantes

### 5. Feedback

Améliorer la détection basée sur les résultats.

## Threat Intel Tools

### IOCs Aggregators

* **VirusTotal** : Analyser les fichiers/URLs
* **AlienVault OTX** : Open threat exchange
* **YETI** : Manage IOCs with tagging
* **URLhaus** : Track phishing URLs

### IP Intelligence

* **Shodan** : Trouver les serveurs exposés
* **GreyNoise** : Distinguer scanners vs attaquants
* **Censys** : Inventory internet-wide

### Threat Actor Tracking

Identifier les groupes d'attaque :

```
Lazarus Group (Corée du Nord)
- TTPs : Phishing + malware custom
- Targets : Secteur financier
- Tools : MATA framework

APT28 (Russie)
- TTPs : Spear phishing + 0-days
- Targets : Gouvernements
- Tools : Fancy Bear tools
```

---

# RÉSUMÉ DES 4 PILIERS

| Pilier | Outil | Fonction |
|--------|-------|----------|
| **Firewall** | Pfsense | Contrôle du trafic réseau |
| **Encryption** | WireGuard VPN | Chiffrement des données |
| **Monitoring** | SIEM + EDR | Détection des menaces |
| **Intelligence** | YETI + VirusTotal | Analyse des menaces |

---

# BEST PRACTICES GLOBALES

1. **Defense In Depth**
   * Plusieurs couches de sécurité
   * Si une échoue, les autres restent

2. **Least Privilege**
   * Donner le minimum d'accès
   * Bloquer tout par défaut

3. **Segmentation**
   * Diviser le réseau en zones
   * Isoler les systèmes critiques

4. **Monitoring Continu**
   * SIEM + EDR actifs 24/7
   * Alertes immédiates

5. **Incident Response**
   * Plan d'action en cas d'attaque
   * Équipe CSIRT dédiée

6. **Threat Intelligence**
   * Rester informé des menaces
   * Adapter la défense

7. **Regular Updates**
   * Patcher tous les systèmes
   * Mettre à jour les signatures

8. **Training & Awareness**
   * Former les employés
   * Sensibiliser aux risques

---

# Compliance & Standards

### GDPR (Europe)

Obligation de :
* Chiffrer les données
* Alerter en cas de fuite
* Avoir une DPO (Data Protection Officer)

### PCI-DSS (Cartes Bancaires)

Obligation de :
* Firewall pour chaque accès
* Chiffrement des cartes
* Scans de vulnérabilités réguliers

### NIST 800-53 (USA - Gouvernement)

Framework de contrôles :
* AC : Access Control
* AU : Audit et accountability
* SC : System and Communications Protection

---

# Points Clés À Retenir

✓ **Firewall** = première ligne de défense
✓ **VPN/Encryption** = protègent les données
✓ **SIEM** = détecte les intrusions réseau
✓ **EDR** = détecte les menaces sur les machines
✓ **Threat Intel** = prévient les attaques futures
✓ **Defense In Depth** = plusieurs couches

---

# Commandes & Outils Essentiels

```bash
# VPN WireGuard
wg-quick up wg0          # Start VPN
wg show                  # Check status
wg-quick down wg0        # Stop VPN

# SIEM (Splunk example)
./splunk start           # Start Splunk
# Access http://localhost:8000

# EDR (exemple)
# Installer agent sur machine
# Vérifier dans console

# Threat Intel (VirusTotal)
curl https://www.virustotal.com/api/v3/files/{hash}
# Check file reputation

# Firewall (pfSense)
# Web UI : https://192.168.1.1
# Configure rules there
```

---

# Structure De Sécurité Complète

```
┌─────────────────────────────────────┐
│         INTERNET                    │
└──────────────┬──────────────────────┘
               │
        ┌──────▼──────┐
        │  FIREWALL   │  ← Contrôle trafic
        │ (pfSense)   │
        └──────┬──────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼──┐   ┌───▼──┐  ┌───▼──┐
│ DMZ  │   │ZONE 1│  │ZONE 2│  ← Segmentation
└───┬──┘   └───┬──┘  └───┬──┘
    │          │         │
    │    ┌─────▼─────┐   │
    │    │   SIEM    │   │      ← Monitoring
    │    │ (Splunk)  │   │        réseau
    │    └───────────┘   │
    │                    │
    └──────────┬─────────┘
               │
         ┌─────▼─────┐
         │ MACHINES  │
         │ (Agents   │       ← Monitoring
         │  EDR)     │         endpoint
         └───────────┘
```

---

## Ce Que J'ai Appris

* Firewalls = contrôle du trafic par règles
* VPN/Encryption = protègent les données en transit
* SIEM = détection réseau centralisée
* EDR = protection endpoint
* Threat Intel = prévention proactive
* Defense in Depth = plusieurs couches nécessaires
