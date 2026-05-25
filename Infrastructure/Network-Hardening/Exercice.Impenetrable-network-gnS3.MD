# Network Hardening Exercise — "Impenetrable Network" — Guide Complet

**Exercice** : Construire un réseau sécurisé pour Company XYZ  
**Durée** : 2h  
**Outils** : GNS3  
**Objectif** : 2 sites (Paris + London) sécurisés avec VPN site-to-site

---

## CONTEXTE DE L'EXERCICE

### Le Problème

Company XYZ a eu une **data breach** il y a quelques mois.

**Leur nouveau réseau DOIT être** :
* Sécurisé (firewall)
* Segmenté (VLANs)
* Connecté entre les 2 sites (VPN)

### Les Sites

**Paris** :
* 3 desktop (employés)
* 1 desktop (admin réseau)
* 2 printers (pas internet)
* 2 IoT devices (avec internet)

**London** :
* 2 desktop (employés)
* 1 desktop (admin réseau)
* 1 printer (pas internet)
* **Pas d'IoT**

### Besoin

Paris ↔ London doivent communiquer entre eux.

---

# ÉTAPE 1 : COMPRENDRE LA STRATÉGIE

## Architecture Globale

```
PARIS                          LONDON
┌─────────────────┐           ┌──────────────┐
│ Employees VLAN  │           │ Employees V. │
│ Admin VLAN      │           │ Admin VLAN   │
│ Printers VLAN   │──VPN───────│ Printers V.  │
│ IoT VLAN        │  Tunnel    │              │
│                 │           │              │
│ Firewall + GW   │           │ Firewall+GW  │
└────────┬────────┘           └──────┬───────┘
         │                           │
       Internet                   Internet
```

**Clé** : VPN tunnel entre les deux firewalls = site-to-site VPN.

---

# ÉTAPE 2 : COMPRENDRE LES VLANs

## Pourquoi Des VLANs?

**VLAN** = Virtual LAN = isoler logiquement les appareils.

**Avantage** :
* Les appareils du VLAN 1 ne peuvent pas communiquer avec VLAN 2 par défaut
* Le firewall contrôle quel trafic entre les VLANs

**Exemple sans VLANs** (MAUVAIS) :
```
Tous les appareils sur même réseau
Imprimante peut se connecter à n'importe quel ordinateur
Trojan sur IoT peut scanner tout le réseau
INSÉCURISÉ!
```

**Exemple avec VLANs** (BON) :
```
VLAN 10 : Employees (isolated)
VLAN 20 : Admin (isolated)
VLAN 30 : Printers (isolated)
VLAN 40 : IoT (isolated)

Firewall contrôle qui parle à qui
IOT ne peut pas communiquer avec Admin par défaut
SÉCURISÉ!
```

## Les 4 VLANs De Paris

### VLAN 10 : EMPLOYEES
```
Description : Ordinateurs des employés
Adresse réseau : 10.1.10.0/24 (Paris) ou 10.2.10.0/24 (London)
Besoin internet : OUI
Besoin VPN : OUI (pour accéder à London)
Appareils : 3 desktops (Paris), 2 desktops (London)

Firewall rules :
- ALLOW internet (sortante)
- ALLOW accès printers (pour imprimer)
- ALLOW VPN tunnel vers London employees
```

### VLAN 20 : ADMIN
```
Description : Ordinateurs des administrateurs réseau
Adresse réseau : 10.1.20.0/24 (Paris) ou 10.2.20.0/24 (London)
Besoin internet : OUI (pour updates, support)
Besoin VPN : OUI (accès admin à London)
Appareils : 1 desktop (Paris), 1 desktop (London)

Firewall rules :
- ALLOW ALL (l'admin a tous les droits)
- ALLOW internet
- ALLOW VPN vers London
```

### VLAN 30 : PRINTERS
```
Description : Imprimantes
Adresse réseau : 10.1.30.0/24 (Paris) ou 10.2.30.0/24 (London)
Besoin internet : NON
Besoin VPN : NON
Appareils : 2 printers (Paris), 1 printer (London)

Firewall rules :
- ALLOW reçevoir du trafic d'employees (imprimer)
- ALLOW reçevoir du trafic d'admin (maintenance)
- DENY TOUT le reste
- NO internet access
```

### VLAN 40 : IoT (Paris SEULEMENT)
```
Description : Appareils IoT (caméras, capteurs, etc.)
Adresse réseau : 10.1.40.0/24 (Paris)
Besoin internet : OUI
Besoin VPN : NON (pas sûr de traverser VPN)
Appareils : 2 IoT devices

Firewall rules :
- ALLOW internet (sortante)
- DENY communications avec autres VLANs
- NO access au data de company
```

---

# ÉTAPE 3 : COMPRENDRE LE FIREWALL

## Regles De Firewall Par VLAN

### Règles VLAN 10 (Employees)

```
Règle 1 : ALLOW VLAN 10 → Internet
Raison : Les employés ont besoin d'accéder à internet

Règle 2 : ALLOW VLAN 10 → VLAN 30 (Printers)
Raison : Les employés doivent pouvoir imprimer

Règle 3 : ALLOW VLAN 10 (Paris) → VLAN 10 (London) via VPN
Raison : Employés Paris accèdent à ressources London

Règle 4 : DENY everything else from VLAN 10
Raison : Sécurité (deny all par défaut)
```

### Règles VLAN 20 (Admin)

```
Règle 1 : ALLOW VLAN 20 → VLAN 10
Raison : Admin gère les ordinateurs employés

Règle 2 : ALLOW VLAN 20 → VLAN 30
Raison : Admin maintient les imprimantes

Règle 3 : ALLOW VLAN 20 → VLAN 40 (IoT)
Raison : Admin gère les IoT devices

Règle 4 : ALLOW VLAN 20 → Internet
Raison : Admin accède à support, updates

Règle 5 : ALLOW VLAN 20 (Paris) → VLAN 20 (London) via VPN
Raison : Admin Paris gère London

Règle 6 : ALLOW VLAN 20 (Paris) → toutes VLANs à London via VPN
Raison : Admin full access partout
```

### Règles VLAN 30 (Printers)

```
Règle 1 : ALLOW VLAN 10 → VLAN 30 (incoming)
Raison : Les employés impriment

Règle 2 : ALLOW VLAN 20 → VLAN 30 (incoming)
Raison : Admin maintient les printers

Règle 3 : DENY VLAN 30 → anything (sortant)
Raison : Les printers ne doivent pas initier de connexions

Règle 4 : DENY VLAN 30 → Internet
Raison : Pas d'accès internet pour printers
```

### Règles VLAN 40 (IoT) - Paris seulement

```
Règle 1 : ALLOW VLAN 40 → Internet
Raison : Les IoT devices ont besoin d'envoyer data

Règle 2 : DENY VLAN 40 → VLAN 10
Raison : IoT ne peut pas accéder aux données des employés

Règle 3 : DENY VLAN 40 → VLAN 20
Raison : IoT ne peut pas accéder aux données d'admin

Règle 4 : DENY VLAN 40 → VLAN 30
Raison : IoT ne peut pas commander les printers

Règle 5 : DENY VLAN 40 → VPN (London)
Raison : IoT ne doit pas accéder à l'autre site
```

---

# ÉTAPE 4 : COMPRENDRE LE VPN SITE-TO-SITE

## Concept

Deux firewalls établissent un **tunnel chiffré** entre les deux sites.

```
Paris Firewall ←─── VPN Tunnel (chiffré) ───→ London Firewall
        ↓                                           ↓
    VLAN 10-40 (encrypted)                   VLAN 10-30 (encrypted)
```

## Flux De Données Via VPN

**Employé Paris veut accéder à imprimante London** :

```
Employé (VLAN 10 Paris)
    ↓
Paquet : src=10.1.10.5 dst=10.2.30.1
    ↓
Firewall Paris :
  - Vérifie règle : ALLOW VLAN 10 → VLAN 30 via VPN? OUI
  - Chiffre le paquet
    ↓
VPN Tunnel (encrypted over internet)
    ↓
Firewall London :
  - Déchiffre le paquet
  - Vérifie règle : ALLOW VLAN 10 Paris → VLAN 30 London? OUI
    ↓
Imprimante London (10.2.30.1)
    ↓
Réponse revient via tunnel, déchiffrée
    ↓
Employé Paris reçoit réponse
```

## Pourquoi VPN Site-to-Site?

**Sans VPN** :
```
Trafic entre Paris et London = en clair sur internet
N'importe qui peut sniffer
Données sensibles exposées
INSÉCURISÉ!
```

**Avec VPN** :
```
Trafic entre Paris et London = chiffré
Même si sniffed, illegible
Données protégées
SÉCURISÉ!
```

---

# ÉTAPE 5 : ARCHITECTURE COMPLÈTE

## Topologie Paris

```
┌──────────────────────────────────────┐
│          PARIS OFFICE                │
└──────────────────────────────────────┘

VLAN 10 (Employees)          VLAN 20 (Admin)
┌─────────────────┐          ┌────────────┐
│ Desktop 1 (IP)  │          │ Desktop    │
│ Desktop 2 (IP)  │          │ Admin (IP) │
│ Desktop 3 (IP)  │          └────────────┘
└────────┬────────┘                │
         │                         │
         └──────────┬──────────────┘
                    │
            ┌───────▼────────┐
            │  Switch VLAN   │  (Distribue par VLAN)
            └────────┬───────┘
                     │
          ┌──────────┼──────────┐
          │          │          │
       ┌──▼───┐  ┌───▼──┐  ┌───▼──┐
       │Printer│  │Printer│  │ IoT  │
       │ VLAN  │  │ VLAN  │  │ VLAN │
       │ 30    │  │ 30    │  │ 40   │
       └───────┘  └───────┘  └──────┘

VLAN 30 (Printers)           VLAN 40 (IoT)
┌────────────────┐           ┌──────────┐
│ Printer 1 (IP) │           │ IoT 1(IP)│
│ Printer 2 (IP) │           │ IoT 2(IP)│
└────────────────┘           └──────────┘

         ┌────────────────────────┐
         │  Firewall + Router     │  (pfSense)
         │  - Contrôle VLANs      │
         │  - VPN Gateway         │
         │  - Ruleset appliqué    │
         └──────────┬─────────────┘
                    │
              ┌─────▼─────┐
              │  Internet  │
              │  Gateway   │
              └────────────┘
```

## Topologie London

```
┌──────────────────────────────────────┐
│         LONDON OFFICE                │
└──────────────────────────────────────┘

VLAN 10 (Employees)          VLAN 20 (Admin)
┌─────────────────┐          ┌────────────┐
│ Desktop 1 (IP)  │          │ Desktop    │
│ Desktop 2 (IP)  │          │ Admin (IP) │
└────────┬────────┘          └────┬───────┘
         │                        │
         └──────────┬─────────────┘
                    │
            ┌───────▼────────┐
            │  Switch VLAN   │  (Distribue par VLAN)
            └────────┬───────┘
                     │
                  ┌──▼───┐
                  │Printer│
                  │ VLAN  │
                  │ 30    │
                  └───────┘

VLAN 30 (Printers)
┌────────────────┐
│ Printer (IP)   │
└────────────────┘

         ┌────────────────────────┐
         │  Firewall + Router     │  (pfSense)
         │  - Contrôle VLANs      │
         │  - VPN Gateway         │
         │  - Ruleset appliqué    │
         └──────────┬─────────────┘
                    │
              ┌─────▼─────┐
              │  Internet  │
              │  Gateway   │
              └────────────┘
```

## Connexion VPN Site-to-Site

```
Paris Firewall ←─────── VPN Tunnel (WireGuard) ─────→ London Firewall
       │                                                    │
    VLANs 10-40                                        VLANs 10-30
       │ (trafic chiffré via VPN)                        │
       └──────────────────────┬───────────────────────────┘
                              │
                    Trafic inter-site
```

---

# ÉTAPE 6 : SCHÉMA RÉSUMÉ AVEC IPs

### Plan IP Proposé

```
PARIS :
- VLAN 10 (Employees) : 10.1.10.0/24
  * Desktop 1 : 10.1.10.10
  * Desktop 2 : 10.1.10.11
  * Desktop 3 : 10.1.10.12
  * Gateway : 10.1.10.1 (Firewall)

- VLAN 20 (Admin) : 10.1.20.0/24
  * Desktop Admin : 10.1.20.10
  * Gateway : 10.1.20.1 (Firewall)

- VLAN 30 (Printers) : 10.1.30.0/24
  * Printer 1 : 10.1.30.10
  * Printer 2 : 10.1.30.11
  * Gateway : 10.1.30.1 (Firewall)

- VLAN 40 (IoT) : 10.1.40.0/24
  * IoT 1 : 10.1.40.10
  * IoT 2 : 10.1.40.11
  * Gateway : 10.1.40.1 (Firewall)

LONDON :
- VLAN 10 (Employees) : 10.2.10.0/24
  * Desktop 1 : 10.2.10.10
  * Desktop 2 : 10.2.10.11
  * Gateway : 10.2.10.1 (Firewall)

- VLAN 20 (Admin) : 10.2.20.0/24
  * Desktop Admin : 10.2.20.10
  * Gateway : 10.2.20.1 (Firewall)

- VLAN 30 (Printers) : 10.2.30.0/24
  * Printer : 10.2.30.10
  * Gateway : 10.2.30.1 (Firewall)
```

---

# ÉTAPE 7 : ÉTAPES DE CONFIGURATION GNS3

## Phase 1 : Setup Paris

### 1.1 Créer les VLANs

Dans Switch managé :
```
VLAN 10 : name employees
VLAN 20 : name admin
VLAN 30 : name printers
VLAN 40 : name iot
```

### 1.2 Configurer le Firewall (pfSense)

Interface WAN : vers internet
Interface LAN : vers le switch

Ajouter interfaces virtuelles pour chaque VLAN :
```
- em1.10 (VLAN 10 employees)
- em1.20 (VLAN 20 admin)
- em1.30 (VLAN 30 printers)
- em1.40 (VLAN 40 iot)
```

### 1.3 Configurer les IPs du Firewall

```
VLAN 10 interface : 10.1.10.1/24
VLAN 20 interface : 10.1.20.1/24
VLAN 30 interface : 10.1.30.1/24
VLAN 40 interface : 10.1.40.1/24
```

### 1.4 Ajouter les appareils

Pour chaque VLAN :
- Créer les VMs (Ubuntu, etc.)
- Les brancher au switch VLAN correct
- Configurer les IPs (DHCP ou statique)

## Phase 2 : Setup London

**Même configuration que Paris**, mais avec:
- Adresses 10.2.x.x au lieu de 10.1.x.x
- Pas de VLAN 40 (IoT)

## Phase 3 : Configurer le VPN

Sur Paris Firewall (pfSense) :
```
VPN Type : WireGuard (ou OpenVPN)
- Generate public/private keys pour Paris
- Set peer (London firewall)
- Tunnel address : 10.0.0.1/32 (Paris side)
```

Sur London Firewall (pfSense) :
```
VPN Type : WireGuard (même)
- Generate public/private keys pour London
- Set peer (Paris firewall) avec public key Paris
- Tunnel address : 10.0.0.2/32 (London side)
```

## Phase 4 : Ajouter les Firewall Rules

Sur Paris Firewall :
```
1. ALLOW VLAN 10 → Internet
2. ALLOW VLAN 10 → VLAN 30
3. ALLOW VLAN 10 → VLAN 10 (London) via VPN
4. ALLOW VLAN 20 → ALL (Admin access)
5. ALLOW VLAN 20 (London) → ALL (via VPN)
6. DENY VLAN 30 → outside VLAN
7. DENY VLAN 40 → VLAN 10, 20, 30
8. DENY everything else
```

Sur London Firewall :
```
Same rules but adapted for London VLANs
```

---

# ÉTAPE 8 : TESTS À FAIRE

## Test 1 : Intra-VLAN Communication

```
Test : Desktop employee (VLAN 10) → Printer (VLAN 30)
Expected : SUCCESS (can print)

Test : IoT device (VLAN 40) → Desktop (VLAN 10)
Expected : FAIL (blocked)
```

## Test 2 : Internet Access

```
Test : Desktop employee (VLAN 10) → Internet
Expected : SUCCESS

Test : Printer (VLAN 30) → Internet
Expected : FAIL (no internet)

Test : IoT device (VLAN 40) → Internet
Expected : SUCCESS
```

## Test 3 : VPN Site-to-Site

```
Test : Desktop Paris (VLAN 10) → Desktop London (VLAN 10) via VPN
Expected : SUCCESS (can ping, ssh, etc.)

Test : Admin Paris (VLAN 20) → Admin London (VLAN 20) via VPN
Expected : SUCCESS (full admin access)

Test : Printer Paris (VLAN 30) → Printer London (VLAN 30) via VPN
Expected : SUCCESS (via VLAN 10 or 20)
```

---

# ÉTAPE 9 : LIVRABLES POUR LE RAPPORT

## Slide 1 : Architecture Globale

Montrer :
- Schéma des 2 sites
- VLANs en Paris
- VLANs en London
- VPN tunnel entre les deux

## Slide 2 : Raison De Cette Architecture

Expliquer :
```
"Nous avons choisi une architecture VLAN segmentée parce que:

1. Isolation : Chaque type d'appareil (employees, admin, printers, IoT)
   est isolé dans son propre VLAN

2. Sécurité : Le firewall contrôle strictement quel trafic peut passer
   entre les VLANs

3. Printers non-internet : Les imprimantes n'ont pas internet, donc
   elles ne peuvent pas être hackées depuis internet

4. IoT isolé : Les appareils IoT (moins sûrs) sont isolés et ne peuvent
   pas accéder aux données sensibles

5. Admin full-access : L'admin a accès à tous les VLANs pour gérer
   le réseau
"
```

## Slide 3 : Best Practices Implémentées

Expliquer :
```
"Nous avons implémenté les best practices suivantes:

1. Least Privilege : Chaque VLAN a le minimum d'accès nécessaire

2. Deny by Default : Toutes les connexions sont bloquées par défaut,
   sauf si explicitement autorisées

3. Segmentation : Les VLANs isolent les appareils par fonction

4. VPN Encryption : Le trafic site-to-site est chiffré

5. Firewall Rules : Contrôle strict du trafic inter-VLAN

6. No Internet for Non-Essential Devices : Printers n'ont pas internet
"
```

---

# CHECKLIST FINALE

## Avant De Soumettre

☐ Topology visible en GNS3 (2 sites avec VLANs)
☐ Firewall configuré sur chaque site
☐ VPN tunnel établi entre les 2 firewalls
☐ Tests intra-VLAN réussis
☐ Tests inter-site via VPN réussis
☐ Firewall rules appliquées
☐ Rapport 1-3 slides avec explications
☐ Architecture justifiée
☐ Best practices expliquées

---

# RÉSUMÉ POUR TOI

## Le Core De L'Exercice

```
Problème : Créer un réseau sécurisé multi-site

Solution :
1. Segmenter chaque site en VLANs (isolation)
2. Configurer un firewall (contrôle du trafic)
3. Établir un VPN tunnel (sécuriser la liaison inter-site)
4. Appliquer les règles de sécurité

Résultat : Réseau "impenetrable" qui applique:
- Least Privilege (minimum d'accès)
- Defense in Depth (plusieurs couches)
- Segmentation (isolation par VLAN)
- Encryption (VPN)
```

## Les Clés Du Succès

1. **Comprendre les VLANs** = isolation logique
2. **Comprendre le Firewall** = contrôle du trafic
3. **Comprendre le VPN** = sécurisation de la liaison
4. **Appliquer les règles** = security by default (deny all)
5. **Documenter** = expliquer le pourquoi

---

Bonne chance! C'est un excellent exercice pour comprendre la défense réseau! 🛡️
