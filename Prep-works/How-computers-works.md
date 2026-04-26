# 💻 Module 01 — How Computers Work

**Formation** : Jedha Cybersécurité — Essentials  
**Statut** : ✅ En cours  

---

## 🗺️ Vue d'ensemble

Avant de sécuriser un système, il faut comprendre comment il fonctionne. Ce module couvre l'architecture physique d'un ordinateur, le processus de démarrage, la communication entre composants, et les fondements logiciels qui s'appuient sur ce matériel. C'est la base sur laquelle toute la sécurité informatique repose.

---

## 🧩 1. Les composants matériels

### Carte mère (Motherboard)

Squelette et système nerveux de l'ordinateur. Relie physiquement tous les composants et assure leur communication. Tout passe par elle.

Connecteurs clés : socket CPU, slots RAM, slots PCIe, connecteurs SATA, ports USB/réseau.

---

### CPU (Central Processing Unit)

Le processeur exécute en permanence des instructions : calculs arithmétiques, comparaisons logiques, gestion mémoire. Les CPU modernes ont plusieurs **cœurs** qui traitent des instructions en parallèle.

Se connecte à la carte mère via le **socket CPU**.

> **Sécurité** : la puissance de calcul du CPU conditionne directement les performances des outils de chiffrement et d'analyse. Un serveur sous-dimensionné peut créer des goulots sur les contrôles de sécurité.

---

### RAM (Random Access Memory)

Mémoire de travail à court terme. Stocke temporairement les données dont le CPU a besoin pendant l'exécution.

Caractéristiques clés :

| Propriété | Détail |
|---|---|
| **Volatile** | Contenu perdu à la coupure d'alimentation |
| **Rapide** | Accès en nanosecondes |
| **Technologies** | DDR4, DDR5, DDR6 |

> **Sécurité — Cold Boot Attack** : la RAM est censée être volatile, mais les données persistent quelques secondes à quelques minutes après extinction. Refroidie rapidement (azote liquide), elles peuvent rester plusieurs heures. Un attaquant avec accès physique peut extraire les clés de chiffrement en mémoire. En forensics : toujours acquérir la RAM AVANT d'éteindre une machine compromise.

---

### Stockage — HDD / SSD

Mémoire à long terme. Les données persistent sans alimentation.

| Caractéristique | HDD | SSD |
|---|---|---|
| Technologie | Plateaux magnétiques rotatifs | Puces mémoire flash |
| Vitesse | Lente | Très rapide |
| Coût | Faible par Go | Plus élevé par Go |
| Résistance chocs | Faible | Bonne |

Interfaces : câbles SATA ou slots PCIe (NVMe).

> **Sécurité** : le stockage est la cible principale en cas d'accès physique non autorisé. Chiffrement au repos obligatoire sur tout équipement mobile (BitLocker / LUKS / FileVault).

---

### Carte réseau (Network Adapter)

Permet la communication avec d'autres systèmes. Peut être intégrée à la carte mère ou ajoutée via slot PCIe.

* **Filaire (Ethernet)** : plus stable et sécurisée
* **Sans fil (Wi-Fi)** : plus exposée aux interceptions

---

### Alimentation (PSU)

Convertit le courant alternatif (AC) en courant continu (DC). Distribue l'énergie à tous les composants. Si sous-dimensionnée → système instable.

---

### GPU (Graphics Processing Unit)

Traite les données visuelles. Connecté via slot PCIe. Ses milliers de cœurs parallèles le rendent aussi très efficace pour les calculs massifs.

> **Sécurité** : les outils comme Hashcat exploitent le GPU pour casser des mots de passe. Un hash MD5 peut être testé à des milliards de tentatives par seconde. C'est pourquoi les algorithmes modernes (bcrypt, Argon2) sont délibérément lents à calculer.

---

### I/O (Entrées / Sorties)

| Catégorie | Exemples | Connecteurs |
|---|---|---|
| Entrée | Clavier, souris, microphone | USB |
| Sortie | Écran, imprimante, enceintes | HDMI, DisplayPort, USB |

> **Sécurité** : les clés USB sont un vecteur d'attaque physique classique (BadUSB, Rubber Ducky). Une politique de contrôle des ports USB est une mesure de sécurité physique standard.

---

## 🔄 2. Le processus de démarrage (Boot Process)

### Les 5 étapes

**Étape 1 — Bouton d'alimentation**  
Signal envoyé au PSU → alimentation de tous les composants.

**Étape 2 — Firmware UEFI**  
L'UEFI (Unified Extensible Firmware Interface) s'initialise et démarre le hardware. Remplace l'ancien BIOS.

| | BIOS | UEFI |
|---|---|---|
| Interface | Texte uniquement | Graphique |
| Support disques | Jusqu'à 2 To (MBR) | Au-delà de 2 To (GPT) |
| Secure Boot | Non | Oui |

**Étape 3 — POST (Power-On Self Test)**  
L'UEFI vérifie que chaque composant requis est présent et fonctionnel. En cas d'échec → signal d'erreur (bips ou message).

**Étape 4 — Sélection du périphérique de boot**  
L'UEFI consulte sa liste ordonnée de périphériques (SSD, HDD, USB, réseau) et cherche un bootloader valide.

**Étape 5 — Chargement du bootloader**  
Le bootloader charge l'OS depuis le stockage vers la RAM. L'UEFI cède ensuite le contrôle à l'OS.

```
Bouton alimentation
    ↓
PSU alimente tous les composants
    ↓
UEFI initialise le hardware
    ↓
POST — vérifie tous les composants
    ↓
UEFI sélectionne le périphérique de boot
    ↓
Bootloader charge l'OS en RAM
    ↓
L'OS prend le contrôle
```

> **Sécurité** : le process de boot est une cible des attaques avancées. Les **bootkits** s'installent dans le firmware UEFI avant le chargement de l'OS, rendant leur détection très difficile. Le **Secure Boot** vérifie la signature cryptographique du bootloader pour bloquer ce type d'attaque.

---

## 🖥️ 3. Le système d'exploitation (OS)

### Rôle de l'OS

L'OS est la couche logicielle entre le matériel et les applications. Il coordonne tout ce qui se passe sur la machine.

```
Utilisateur
    ↕
Applications
    ↕
Système d'exploitation (OS)
    ↕
Matériel physique
```

### Séparation des privilèges

| Espace | Accès | Contenu |
|---|---|---|
| **Kernel space** | Illimité au matériel | Le noyau — gère directement CPU, RAM, stockage |
| **User space** | Restreint | Toutes les applications — doivent faire des system calls |

Un **system call** est une requête d'une application au kernel pour accéder à une ressource matérielle.

> **Sécurité** : une vulnérabilité permettant à un processus user space d'atteindre le kernel space = **privilege escalation**. C'est l'un des vecteurs d'attaque les plus critiques. CVE-2022-0847 (Dirty Pipe), CVE-2023-0179, etc.

### Responsabilités de l'OS

| Responsabilité | Rôle |
|---|---|
| Gestion des processus | Crée, planifie, priorise, termine les programmes |
| Gestion de la mémoire | Alloue la RAM, protège les espaces mémoire, gère la mémoire virtuelle |
| Gestion du système de fichiers | Organise fichiers, répertoires, permissions, métadonnées |
| Gestion des utilisateurs | Comptes, authentification, permissions |
| Gestion des périphériques | Drivers, couche d'abstraction matérielle |

---

## 🌐 4. Réseaux et communication

### Le modèle client-serveur

```
Client (initie la requête)  →  Serveur (répond)
```

**Règle fondamentale** : c'est toujours le client qui initie.

### Concepts clés

| Concept | Définition |
|---|---|
| **Protocole** | Règles de communication (HTTP, SSH, DNS...) |
| **Port** | Identifie un service spécifique sur un serveur (HTTP=80, HTTPS=443, SSH=22) |
| **IP Address** | Adresse réseau d'un système |
| **DNS** | Traduit un nom de domaine en adresse IP |

### HTTP / HTTPS

HTTP est le protocole du web. **Stateless** : chaque requête est indépendante.

| Méthode | Usage |
|---|---|
| GET | Récupérer une ressource |
| POST | Envoyer des données |
| PUT | Créer/remplacer une ressource |
| DELETE | Supprimer une ressource |

Codes de statut clés :

| Code | Signification |
|---|---|
| 200 | Succès |
| 403 | Accès refusé |
| 404 | Ressource non trouvée |
| 500 | Erreur serveur |

---

## ☁️ 5. Virtualisation et Cloud

### Virtualisation

Un **hyperviseur** divise un serveur physique en plusieurs machines virtuelles (VMs) indépendantes.

| Type | Tourne sur | Usage |
|---|---|---|
| Type 1 (bare-metal) | Matériel physique | Production, datacenters |
| Type 2 (hosted) | OS existant | Labs, tests, apprentissage |

**Containers** : environnement léger qui partage le kernel de l'hôte. Plus rapide à démarrer qu'une VM, moins isolé.

> **Sécurité** : un container partage le kernel de l'hôte. Une vulnérabilité kernel affecte potentiellement tous les containers du nœud. Les workloads sensibles restent dans des VMs.

### Cloud

| Modèle | Responsabilité |
|---|---|
| IaaS | Fournisseur : matériel. Client : OS + apps |
| PaaS | Fournisseur : matériel + OS. Client : apps |
| SaaS | Fournisseur : tout. Client : utilisation |

> **Point critique** : le modèle de responsabilité partagée. AWS sécurise l'infrastructure. La configuration de tes ressources reste ta responsabilité. Une misconfiguration S3 en accès public = incident de données, et AWS n'interviendra pas.

---

## 🛡️ Synthèse sécurité

| Composant | Vecteur d'attaque | Contrôle |
|---|---|---|
| UEFI | Bootkit firmware | Secure Boot, MàJ firmware |
| RAM | Cold boot attack | Chiffrement mémoire, extinction sécurisée |
| Stockage | Accès physique | Chiffrement au repos (BitLocker/LUKS) |
| USB / I/O | BadUSB, exfiltration | Politique USB, contrôle des ports |
| Kernel space | Privilege escalation | Patching noyau, hardening |
| HTTP | Interception, injection | HTTPS obligatoire, CSP |
| Containers | Kernel exploit | Isolation renforcée, MàJ kernel |
| Cloud | Misconfiguration | Posture management, IAM strict |

---

## 📋 Références

* [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
* [ANSSI — Recommandations GNU/Linux](https://www.ssi.gouv.fr)
* [NIST SP 800-125 — Virtualisation Security](https://csrc.nist.gov/publications/detail/sp/800-125/final)
* [ISO 27001:2022](https://www.iso.org/standard/27001)
