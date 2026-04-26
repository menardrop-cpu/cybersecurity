# 🌐 Understand Networking

**Formation** : Jedha Cybersécurité — Essentials  
**Module** : How Computers Work  
**Statut** : ✅ Complété

---

## 1. Reminder : What Is a Network?

Un réseau informatique est un ensemble de systèmes connectés entre eux capables d'échanger des données.

La définition est simple. Ce qui est moins simple, c'est de comprendre **comment** cette communication est rendue possible, **qui** peut parler à **qui**, et **comment** on contrôle ce qui circule.

### Pourquoi les réseaux existent

Avant les réseaux, chaque ordinateur fonctionnait en silo. Pour transférer un fichier d'une machine à une autre, il fallait une disquette ou une clé USB physique (on appelait ça le "sneakernet"). Les réseaux ont résolu ce problème en permettant :

* Le partage de ressources (fichiers, imprimantes, connexion internet)
* La communication en temps réel (email, messagerie, voix sur IP)
* L'accès centralisé aux données (serveurs, bases de données)
* La collaboration à distance

### Le modèle fondamental

Tout réseau repose sur le même modèle vu dans le module précédent :

```
Client (initie la requête)
        ↕
Réseau (transporte les données)
        ↕
Serveur (répond à la requête)
```

La différence avec ce qu'on a vu jusqu'ici : le "réseau" entre les deux n'est plus juste un câble. C'est une infrastructure complexe avec ses propres composants, protocoles et règles.

---

## 2. Network Components

Un réseau est constitué de plusieurs types de composants. Chacun joue un rôle précis dans le transport et l'organisation du trafic.

### Les équipements physiques (Hardware)

**Carte réseau (NIC : Network Interface Card)**

C'est l'interface entre un équipement et le réseau. Chaque NIC possède une adresse **MAC (Media Access Control)** gravée en usine, unique au monde sur 48 bits, représentée en hexadécimal :

```
00:1A:2B:3C:4D:5E
```

Les 3 premiers octets identifient le fabricant (OUI : Organizationally Unique Identifier). Les 3 derniers identifient l'interface spécifique.

> **Sécurité** : l'adresse MAC peut être **usurpée** (MAC spoofing) logiciellement. Un attaquant peut se faire passer pour un équipement légitime sur le réseau local. Les systèmes de contrôle d'accès basés uniquement sur l'adresse MAC (MAC filtering) sont donc insuffisants comme mesure de sécurité.

**Switch (Commutateur)**

Équipement qui interconnecte plusieurs appareils au sein du même réseau local. Il fonctionne au niveau 2 du modèle OSI (couche liaison) et utilise les adresses MAC pour diriger le trafic.

Comportement d'un switch :

1. Un appareil A envoie une trame à l'appareil B
2. Le switch reçoit la trame et lit l'adresse MAC de destination
3. Il consulte sa **table CAM** (Content Addressable Memory) qui associe chaque adresse MAC au port physique correspondant
4. Il envoie la trame uniquement sur le port de B, pas sur tous les ports

Cela évite les collisions et améliore les performances comparé à l'ancien **hub** qui diffusait tout sur tous les ports.

> **Sécurité** : une attaque classique sur les switches est le **MAC flooding**. Un attaquant inonde le switch de trames avec des adresses MAC fausses jusqu'à saturer la table CAM. Le switch, ne sachant plus où envoyer le trafic, bascule en mode "hub" et diffuse tout à tout le monde. L'attaquant peut alors intercepter le trafic qui ne lui est pas destiné.

**Routeur (Router)**

Équipement qui interconnecte plusieurs réseaux différents. Il fonctionne au niveau 3 du modèle OSI (couche réseau) et utilise les adresses **IP** pour prendre ses décisions de routage.

Le routeur maintient une **table de routage** qui indique par quel chemin envoyer les paquets selon leur destination.

| Réseau destination | Masque | Passerelle | Interface |
|---|---|---|---|
| 192.168.1.0 | /24 | Direct | eth0 |
| 10.0.0.0 | /8 | 192.168.1.1 | eth0 |
| 0.0.0.0 | /0 | 203.0.113.1 | eth1 |

La dernière ligne (`0.0.0.0/0`) est la **route par défaut** : tout ce qui ne correspond à aucune règle spécifique est envoyé vers cette passerelle (souvent l'accès internet).

> **Sécurité** : les routeurs sont des cibles prioritaires. Compromettre un routeur donne un accès total au trafic transitant par lui. Les attaques incluent la modification des tables de routage (route poisoning), l'exploitation de firmwares non patchés, et les attaques sur les protocoles de routage (RIP, BGP hijacking).

**Point d'accès Wi-Fi (Access Point)**

Équipement qui permet aux appareils sans fil de se connecter au réseau filaire. Il traduit entre les trames Wi-Fi (802.11) et les trames Ethernet.

> **Sécurité** : les réseaux Wi-Fi publics non chiffrés permettent l'interception du trafic. Les attaques courantes incluent le **Evil Twin** (faux point d'accès qui imite un réseau légitime), le **KRACK** (attaque sur WPA2) et le **PMKID attack** sur WPA3. Toujours utiliser HTTPS même sur un réseau de confiance.

**Pare-feu (Firewall)**

Équipement ou logiciel qui filtre le trafic réseau selon des règles définies. Il décide quelles connexions sont autorisées ou bloquées.

Types de filtrage :

| Type | Fonctionnement |
|---|---|
| Stateless | Analyse chaque paquet individuellement selon des règles fixes |
| Stateful | Suit l'état des connexions, comprend le contexte |
| Applicatif (WAF) | Analyse le contenu applicatif, détecte les attaques web |

**Câbles et supports physiques**

| Support | Débit max | Distance max | Usage |
|---|---|---|---|
| Ethernet Cat5e | 1 Gbps | 100 m | Bureaux |
| Ethernet Cat6 | 10 Gbps | 55 m | Datacenters |
| Fibre optique | 100+ Gbps | Plusieurs km | Backbone, FAI |
| Wi-Fi 6 (802.11ax) | 9.6 Gbps théorique | ~50 m indoor | Sans fil |

### Les adresses réseau

**Adresse IP (Internet Protocol)**

Identifiant logique d'un équipement sur un réseau. Contrairement à l'adresse MAC (physique et permanente), l'adresse IP est logique et peut changer.

**IPv4** : 32 bits, notée en 4 octets décimaux séparés par des points.

```
192.168.1.42
```

Chaque octet vaut entre 0 et 255. Soit environ 4,3 milliards d'adresses IPv4 possibles, épuisées depuis 2019.

**IPv6** : 128 bits, notée en 8 groupes de 4 chiffres hexadécimaux.

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

Soit 340 sextillions d'adresses, ce qui résout l'épuisement des adresses IPv4.

**Masque de sous-réseau**

Définit quelle partie de l'adresse IP identifie le réseau et quelle partie identifie l'hôte.

```
IP      : 192.168.1.42
Masque  : 255.255.255.0  (ou /24 en notation CIDR)

Réseau  : 192.168.1.0
Hôte    : .42
```

En notation CIDR, `/24` signifie que les 24 premiers bits définissent le réseau. Un `/24` contient 254 adresses d'hôtes utilisables (256 moins l'adresse réseau et l'adresse de diffusion).

Exemples courants :

| Notation CIDR | Masque | Hôtes utilisables |
|---|---|---|
| /8 | 255.0.0.0 | 16 777 214 |
| /16 | 255.255.0.0 | 65 534 |
| /24 | 255.255.255.0 | 254 |
| /30 | 255.255.255.252 | 2 |

**Plages d'adresses privées**

Ces plages sont réservées aux réseaux locaux et ne sont pas routées sur internet.

| Plage | Notation CIDR | Usage typique |
|---|---|---|
| 10.0.0.0 à 10.255.255.255 | /8 | Grandes entreprises |
| 172.16.0.0 à 172.31.255.255 | /12 | Entreprises moyennes |
| 192.168.0.0 à 192.168.255.255 | /16 | Réseaux domestiques |

> **Sécurité** : identifier si une adresse est privée ou publique est fondamental en pentest. Les adresses RFC 1918 (privées) ne sont pas directement accessibles depuis internet sans NAT ou VPN.

**Ports**

Un port identifie un service spécifique sur une machine. Il complète l'adresse IP pour former ce qu'on appelle un **socket**.

```
192.168.1.42:443    →  HTTPS sur la machine 192.168.1.42
```

Plages de ports :

| Plage | Nom | Description |
|---|---|---|
| 0 à 1023 | Well-known ports | Réservés aux services standards |
| 1024 à 49151 | Registered ports | Assignés à des applications spécifiques |
| 49152 à 65535 | Dynamic / Ephemeral ports | Utilisés temporairement par les clients |

Ports standards à connaître par cœur :

| Port | Protocole | Service |
|---|---|---|
| 20 / 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet (non chiffré, obsolète) |
| 25 | TCP | SMTP (email) |
| 53 | TCP / UDP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP (Remote Desktop Windows) |
| 8080 | TCP | HTTP alternatif |

> **Sécurité** : un scan de ports (nmap) est la première étape d'une reconnaissance réseau. Identifier les ports ouverts sur une cible révèle les services exposés et leurs versions potentiellement vulnérables. Tout port ouvert est une surface d'attaque potentielle.

---

## 3. Local Area Network (LAN)

### Définition

Un LAN (Local Area Network) est un réseau limité à une zone géographique restreinte : un bureau, un bâtiment, un campus. Les équipements y sont connectés par des câbles Ethernet ou du Wi-Fi.

Caractéristiques d'un LAN :

* Haute vitesse (1 Gbps à 10 Gbps en filaire)
* Faible latence
* Géré par une seule organisation
* Privé et contrôlé

### Composants typiques d'un LAN

```
[Internet]
    |
[Modem / Box FAI]
    |
[Routeur]
    |
[Switch principal]
    |
├── [Switch bureau 1]
│       ├── Poste de travail A
│       ├── Poste de travail B
│       └── Imprimante réseau
├── [Switch bureau 2]
│       ├── Poste de travail C
│       └── Serveur fichiers
└── [Access Point Wi-Fi]
        ├── Laptop D
        └── Smartphone E
```

### VLAN (Virtual LAN)

Un VLAN permet de segmenter logiquement un réseau physique en plusieurs réseaux virtuels isolés. Deux équipements sur le même switch peuvent être dans des VLANs différents et ne pas pouvoir communiquer directement.

```
Switch physique unique
├── VLAN 10 : Ressources humaines
├── VLAN 20 : Finance
├── VLAN 30 : IT
└── VLAN 40 : IoT / Appareils connectés
```

> **Sécurité** : les VLANs sont un contrôle de segmentation réseau fondamental. Ils empêchent la propagation latérale d'une attaque : si un poste du VLAN RH est compromis, l'attaquant ne peut pas directement atteindre les serveurs Finance sur un autre VLAN. C'est l'application du principe du moindre privilège au niveau réseau. ISO 27001 A.8.22.

### NAT (Network Address Translation)

Le NAT permet à tous les équipements d'un réseau privé de partager une seule adresse IP publique pour accéder à internet.

```
Réseau privé (192.168.1.0/24)       Internet
Poste A : 192.168.1.10 ──┐
Poste B : 192.168.1.20 ──┤ Routeur NAT ──► IP publique : 203.0.113.5
Poste C : 192.168.1.30 ──┘
```

Le routeur maintient une table NAT qui associe chaque connexion interne à un port temporaire côté public. Quand la réponse arrive, il sait à quel équipement interne la renvoyer.

---

## 4. Network Geography

La géographie réseau classe les réseaux selon leur étendue physique et leur usage. Chaque catégorie a ses propres technologies, contraintes et enjeux de sécurité.

### PAN (Personal Area Network)

Réseau personnel, portée de quelques mètres autour d'un individu.

| Caractéristique | Valeur |
|---|---|
| Portée | Quelques mètres |
| Technologies | Bluetooth, NFC, USB |
| Exemples | Casque Bluetooth, montre connectée, partage mobile |

> **Sécurité** : le Bluetooth est régulièrement ciblé par des attaques. BlueSnarfing (accès non autorisé aux données), Bluebugging (prise de contrôle), et BIAS (Bluetooth Impersonation Attack). Désactiver le Bluetooth quand il n'est pas utilisé est une bonne pratique.

### LAN (Local Area Network)

Réseau local, portée d'un bâtiment ou d'un campus.

| Caractéristique | Valeur |
|---|---|
| Portée | Bâtiment / campus |
| Technologies | Ethernet, Wi-Fi |
| Débit typique | 1 Gbps à 10 Gbps |
| Gestion | Une seule organisation |
| Exemples | Réseau d'entreprise, réseau domestique |

### MAN (Metropolitan Area Network)

Réseau métropolitain, couvre une ville ou une région. Souvent opéré par un fournisseur de télécommunications.

| Caractéristique | Valeur |
|---|---|
| Portée | Ville / région |
| Technologies | Fibre optique, WiMAX |
| Exemples | Réseau d'une université multi-campus, réseau d'une ville |

### WAN (Wide Area Network)

Réseau étendu, couvre de grandes distances géographiques. Internet est le plus grand WAN du monde.

| Caractéristique | Valeur |
|---|---|
| Portée | Pays / continents / mondial |
| Technologies | Fibre longue distance, satellite, MPLS |
| Débit typique | Variable selon l'opérateur |
| Exemples | Internet, réseau privé multi-sites d'une entreprise |

Les entreprises multi-sites utilisent souvent un WAN privé (MPLS ou SD-WAN) pour interconnecter leurs agences, plutôt que de faire transiter les données sensibles sur l'internet public.

### CAN (Campus Area Network)

Réseau de campus, relie plusieurs bâtiments sur un même site (université, hôpital, parc industriel). Intermédiaire entre LAN et MAN.

### SAN (Storage Area Network)

Réseau dédié au stockage. Relie les serveurs aux équipements de stockage (baies de disques, NAS, etc.) via un réseau haute performance séparé du réseau de données classique.

> **Sécurité** : les SANs sont des cibles prioritaires lors d'attaques ransomware car ils concentrent toutes les données de l'organisation. Leur segmentation stricte du reste du réseau est critique.

### Comparatif géographique

| Type | Portée | Débit typique | Gestion | Exemple |
|---|---|---|---|---|
| PAN | Quelques mètres | Faible | Individu | Bluetooth entre téléphone et casque |
| LAN | Bâtiment | 1 à 10 Gbps | Organisation unique | Réseau de bureau |
| CAN | Campus | 10 Gbps+ | Organisation unique | Réseau universitaire |
| MAN | Ville | Variable | Opérateur télco | Réseau métropolitain |
| WAN | Mondial | Variable | Multiple | Internet |
| SAN | Datacenter | 8 à 32 Gbps | Organisation | Réseau de stockage |

---

## 🛡️ Synthèse sécurité réseau

| Composant / Concept | Vecteur d'attaque | Contrôle recommandé |
|---|---|---|
| Adresse MAC | MAC spoofing | 802.1X (authentification port par port) |
| Switch | MAC flooding, VLAN hopping | Port security, VLAN natif dédié |
| Routeur | Route poisoning, firmware vulnérable | MàJ firmware, authentification des protocoles de routage |
| Wi-Fi | Evil Twin, KRACK, PMKID | WPA3, réseau invité isolé, VPN |
| Ports ouverts | Exploitation de services vulnérables | Fermer les ports inutiles, firewall strict |
| LAN non segmenté | Propagation latérale | VLANs, micro-segmentation |
| IP privée exposée | Reconnaissance interne | Filtrage sortant, IDS/IPS |
| Telnet (port 23) | Interception en clair | Remplacer par SSH (port 22) |
| RDP (port 3389) | Brute force, BlueKeep | MFA, exposition VPN uniquement |

---

## 📋 Références

* [RFC 1918 : Adresses privées](https://www.rfc-editor.org/rfc/rfc1918)
* [Cisco : VLAN Configuration Guide](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960/software/release/12-2_55_se/configuration/guide/scg2960/swvlan.html)
* [ANSSI : Recommandations de sécurité pour les réseaux](https://www.ssi.gouv.fr)
* [NIST SP 800-153 : Wi-Fi Security](https://csrc.nist.gov/publications/detail/sp/800-153/final)
* [ISO 27001:2022 A.8.20 : Sécurité des réseaux](https://www.iso.org/standard/27001)
