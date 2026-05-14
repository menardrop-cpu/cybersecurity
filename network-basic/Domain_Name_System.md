# DNS: Domain Name System

## 1. DNS Basics 😙

### Qu'est-ce que le DNS?

Le DNS (Domain Name System) est le service de résolution de noms d'Internet. Il traduit les noms de domaine lisibles par l'homme (exemple.com) en adresses IP (192.0.2.1) que les machines comprennent.

**Pourquoi c'est critique:**
* Chaque requête web passe par une résolution DNS
* Un DNS lent ralentit l'expérience utilisateur
* Un DNS compromis peut rediriger le trafic vers des serveurs malveillants
* Référence ISO 27001: contrôle de la disponibilité et de l'intégrité des services critiques

### Architecture simple

```
Utilisateur
    ↓ (je veux accéder à google.com)
Résolveur local (client/routeur)
    ↓ (résous google.com)
Serveur DNS récursif (FAI)
    ↓ (demande au serveur racine)
Serveur racine (.)
    ↓ (demande au serveur TLD)
Serveur TLD (.com)
    ↓ (demande au serveur autoritaire)
Serveur autoritaire (google.com)
    ↓ (réponse: 142.250.185.46)
Utilisateur (affichage du site)
```

### Le modèle de résolution DNS

1. **Recursive Query**: Le client demande au résolveur de faire toute la résolution
2. **Iterative Query**: Le serveur DNS renvoie soit la réponse, soit l'adresse d'un autre serveur

### Hiérarchie DNS

* **Root nameserver (.)**: Niveau 0, connaît les serveurs TLD
* **TLD nameserver (.com, .fr, .org)**: Niveau 1, connaît les serveurs autoritaires
* **Authoritative nameserver**: Niveau 2, détient les vrais enregistrements DNS

### Caching et TTL

* **TTL (Time To Live)**: Durée en secondes pendant laquelle un enregistrement DNS peut être mis en cache
* **TTL court** (300s): Idéal pour les domaines changeants ou en migration
* **TTL long** (3600s ou +): Idéal pour la stabilité et la réduction de la charge DNS

Exemple:
```
$ dig google.com +noall +answer
google.com.		300	IN	A	142.250.185.46
```
Ici, TTL = 300 secondes. Chaque résolveur peut cacher cette réponse pendant 5 minutes.

---

## 2. DNS Servers 🤖

### Types de serveurs DNS

#### Serveur DNS récursif (Resolver)

* Répond aux requêtes des clients
* Effectue toute la résolution (demandes aux serveurs racine, TLD, autoritaires)
* Met en cache les réponses
* Utilisé par les FAI, les entreprises, les services publics (8.8.8.8 de Google, 1.1.1.1 de Cloudflare)

**Exemple de configuration (bind9):**
```
options {
    listen-on { 127.0.0.1; };
    recursion yes;
    forwarders { 8.8.8.8; 8.8.4.4; };
};
```

#### Serveur DNS autoritaire (Nameserver)

* Détient les enregistrements DNS réels d'un domaine
* Ne cache pas (stockage définitif)
* Répond aux requêtes itératives
* Configuré par le propriétaire du domaine

**Exemple:**
* ns1.example.com
* ns2.example.com
* Registre du domaine pointe vers ces serveurs

#### Serveur DNS secondaire

* Réplique des serveurs primaires
* Zone transfers (AXFR) depuis le serveur primaire
* Augmente la résilience et distribue la charge
* Recommandé pour la continuité de service (ISO 27001: A.12.3.1)

### Configuration de base d'un nameserver

**Fichier de configuration simplifié:**
```
zone "example.com" {
    type master;
    file "/etc/bind/zones/example.com.db";
    allow-transfer { 192.0.2.2; };
};
```

**Fichier de zone (example.com.db):**
```
$TTL 3600
@   IN  SOA ns1.example.com. admin.example.com. (
            2024051401  ; Serial
            3600        ; Refresh (1h)
            1800        ; Retry (30m)
            604800      ; Expire (7d)
            86400 )     ; Minimum TTL (1d)
    IN  NS  ns1.example.com.
    IN  NS  ns2.example.com.
    IN  A   192.0.2.1
www IN  A   192.0.2.10
mail IN A   192.0.2.20
```

### Architecture recommandée pour une organisation

```
Internet
    ↓
[Serveur DNS récursif interne]
    ↓ (cache, sécurité)
    ├→ [Serveur autoritaire primaire] (example.com)
    ├→ [Serveur autoritaire secondaire] (réplication)
    └→ [Forwarders externes] (8.8.8.8, etc.)
```

**Référence GRC:**
* OWASP: DNS amplification attacks (amplifier attack)
* ISO 27001 A.12.3.1: Disponibilité des services essentiels
* NIS2: Essentialité du DNS pour les opérateurs de services critiques

---

## 3. DNS Records 📝

### Types d'enregistrements principaux

#### A (Address) - IPv4

Relie un nom de domaine à une adresse IPv4.

```
example.com.    IN  A   192.0.2.1
www.example.com. IN  A   192.0.2.10
```

* 1 domaine = 1 seule adresse IPv4 par enregistrement
* Supporté par tous les clients

#### AAAA - IPv6

Relie un nom de domaine à une adresse IPv6.

```
example.com.    IN  AAAA    2001:db8::1
www.example.com. IN  AAAA    2001:db8::10
```

#### CNAME (Canonical Name)

Alias vers un autre nom de domaine. Utilisé pour rediriger sans changer l'enregistrement principal.

```
blog.example.com.   IN  CNAME   example.com.
ftp.example.com.    IN  CNAME   example.com.
```

**Important:**
* Jamais de CNAME vers un autre domaine si celui-ci a d'autres enregistrements (A, MX, etc.)
* Le CNAME ne doit pas être un sous-domaine d'une zone avec MX ou NS

#### MX (Mail Exchange)

Indique le serveur de messagerie responsable du domaine, avec une priorité.

```
example.com.    IN  MX  10  mail1.example.com.
example.com.    IN  MX  20  mail2.example.com.
```

Priorité basse = préféré d'abord. Ici, mail1 reçoit les emails en premier, mail2 en second.

#### NS (Nameserver)

Désigne les serveurs DNS autoritaires du domaine.

```
example.com.    IN  NS  ns1.example.com.
example.com.    IN  NS  ns2.example.com.
```

Configuré chez le registrar (GoDaddy, OVH, etc.). C'est la passerelle vers vos serveurs DNS.

#### SOA (Start Of Authority)

Enregistrement de contrôle de la zone. Contient les paramètres de gestion et de réplication.

```
example.com.    IN  SOA ns1.example.com. admin.example.com. (
                2024051401  ; Serial
                3600        ; Refresh
                1800        ; Retry
                604800      ; Expire
                86400 )     ; Minimum TTL
```

* **Serial**: Numéro de version de la zone (incrémenté à chaque modification)
* **Refresh**: Délai avant que le serveur secondaire demande une mise à jour (3600s = 1h)
* **Retry**: Délai avant nouvelle tentative en cas d'échec (1800s = 30m)
* **Expire**: Durée max avant considération comme obsolète (604800s = 7j)
* **Minimum TTL**: TTL par défaut pour tous les enregistrements

#### TXT (Text)

Enregistrements texte pour configurations spéciales.

```
example.com.    IN  TXT "v=spf1 include:_spf.google.com ~all"
_dmarc.example.com. IN  TXT "v=DMARC1; p=reject; rua=mailto:dmarc@example.com"
_acme-challenge.example.com. IN TXT "validation-string-pour-ssl"
```

**Utilisations courantes:**
* **SPF** (Sender Policy Framework): Autorise certains serveurs à envoyer des emails du domaine
* **DMARC** (Domain-based Message Authentication): Politique d'authentification et rapport
* **DKIM** (DomainKeys Identified Mail): Signature numérique des emails
* **Let's Encrypt validation**: Preuve de propriété du domaine

#### PTR (Pointer)

Résolution inverse: traduit une IP vers un nom de domaine. Critère clé pour les emails.

```
1.2.0.192.in-addr.arpa.    IN  PTR mail.example.com.
```

* Configuré dans la zone inverse du fournisseur d'IP
* Important pour l'authentification des serveurs de messagerie (SPF, DMARC check)
* Référence ISO 27001: Logging et audit des services réseau

### Table récapitulative

| Type | Utilité | Exemple |
|------|---------|---------|
| A | IPv4 → Nom | example.com → 192.0.2.1 |
| AAAA | IPv6 → Nom | example.com → 2001:db8::1 |
| CNAME | Alias | www → example.com |
| MX | Serveur de mail | example.com → mail.example.com (priorité 10) |
| NS | Serveur DNS | example.com → ns1.example.com |
| SOA | Contrôle de zone | Paramètres de gestion de la zone |
| TXT | Texte libre | SPF, DMARC, DKIM, validation |
| PTR | IP → Nom | 192.0.2.1 → mail.example.com |

---

## 4. NS Lookup and DIG 🕵️‍♂️

### nslookup - Outil simple de résolution

**Syntaxe:**
```bash
nslookup [domaine] [serveur DNS]
```

**Exemples:**

Requête simple:
```bash
$ nslookup google.com
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
Name:	google.com
Address: 142.250.185.46
```

Requête vers un serveur DNS spécifique:
```bash
$ nslookup google.com 1.1.1.1
```

Requête sur un type spécifique (MX, NS, TXT):
```bash
$ nslookup -type=MX example.com
```

Requête en reverse (IP → Nom):
```bash
$ nslookup 192.0.2.1
```

**Limitations:**
* Interface moins flexible que dig
* Pas idéal pour les scripts
* Moins de contrôle sur les paramètres de requête

### dig - Outil professionnel de requête DNS

**Syntaxe:**
```bash
dig [@serveur] domaine [type]
```

**Exemples essentiels:**

Requête A simple:
```bash
$ dig google.com
; <<>> DiG 9.16.33 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		300	IN	A	142.250.185.46

;; Query time: 45 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
```

Requête vers un serveur DNS spécifique:
```bash
$ dig @ns1.example.com example.com
```

Requête MX (serveurs de mail):
```bash
$ dig example.com MX

;; ANSWER SECTION:
example.com.	3600	IN	MX	10 mail1.example.com.
example.com.	3600	IN	MX	20 mail2.example.com.
```

Requête NS (serveurs DNS):
```bash
$ dig example.com NS
```

Requête TXT (SPF, DMARC):
```bash
$ dig example.com TXT
$ dig _dmarc.example.com TXT
```

Requête reverse (PTR):
```bash
$ dig -x 142.250.185.46
```

### Options utiles de dig

| Option | Effet |
|--------|-------|
| `+short` | Affiche uniquement la réponse |
| `+noall +answer` | Affiche uniquement l'enregistrement |
| `+trace` | Trace la requête du root au serveur autoritaire |
| `+stats` | Affiche les statistiques |
| `+nocmd` | Masque la ligne de commande |
| `@serveur` | Interroge un serveur DNS spécifique |
| `+tcp` | Force la requête en TCP (au lieu d'UDP) |
| `+dnssec` | Valide la signature DNSSEC |

**Exemples avancés:**

Réponse simplifiée:
```bash
$ dig google.com +short
142.250.185.46
```

Afficher uniquement l'enregistrement (idéal pour les scripts):
```bash
$ dig google.com +noall +answer
google.com.		300	IN	A	142.250.185.46
```

Tracer la résolution complète:
```bash
$ dig google.com +trace

; <<>> DiG 9.16.33 <<>> google.com +trace
; (1 server found)
;; global options: +cmd
.			518400	IN	NS	a.root-servers.net.
.			518400	IN	NS	b.root-servers.net.
...
com.		172800	IN	NS	a.gtld-servers.net.
com.		172800	IN	NS	b.gtld-servers.net.
...
google.com.		900	IN	NS	ns1.google.com.
google.com.		900	IN	NS	ns2.google.com.
...
google.com.		300	IN	A	142.250.185.46
```

### Interprétation des réponses

**Section HEADER:**
```
;; flags: qr rd ra;
```
* `qr`: Query Response (réponse = oui)
* `rd`: Recursion Desired (récursion demandée = oui)
* `ra`: Recursion Available (récursion disponible = oui)
* `aa`: Authoritative Answer (réponse autoritaire = oui)

**Codes de statut:**
* `NOERROR`: Succès
* `NXDOMAIN`: Domaine inexistant
* `SERVFAIL`: Erreur serveur
* `REFUSED`: Serveur refuse la requête (DNSSEC invalide, restriction ACL)
* `TIMEOUT`: Pas de réponse

**Exemple diagnostic:**
```bash
$ dig nonexistent.com
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN
```
Cela signifie que le domaine n'existe pas.

### Cas d'usage pratiques

**1. Vérifier que le DNS a bien été changé:**
```bash
$ dig example.com +noall +answer
example.com.		3600	IN	A	192.0.2.1
```

**2. Identifier les serveurs DNS d'un domaine:**
```bash
$ dig example.com NS +short
ns1.example.com.
ns2.example.com.
```

**3. Vérifier les enregistrements MX:**
```bash
$ dig example.com MX +short
10 mail1.example.com.
20 mail2.example.com.
```

**4. Valider une configuration SPF/DMARC:**
```bash
$ dig example.com TXT +short
v=spf1 include:_spf.google.com ~all
```

**5. Reverse DNS (vérifier que l'IP correspond):**
```bash
$ dig -x 192.0.2.1 +short
mail.example.com.
```

**6. Diagnostiquer une propagation DNS lente:**
```bash
$ dig @8.8.8.8 example.com +stats
Query time: 145 msec  (trop lent, check TTL / cache)

$ dig @1.1.1.1 example.com +stats
Query time: 12 msec   (bien)
```

### Scripting avec dig

**Boucle sur plusieurs domaines:**
```bash
for domain in example.com test.com mysite.fr; do
  echo "=== $domain ==="
  dig $domain +short
done
```

**Extraction de l'IP pour un script:**
```bash
IP=$(dig example.com +short)
echo "IP of example.com: $IP"
```

**Vérification de santé DNS:**
```bash
#!/bin/bash
DOMAIN="example.com"
EXPECTED_IP="192.0.2.1"
ACTUAL_IP=$(dig $DOMAIN +short)

if [ "$ACTUAL_IP" = "$EXPECTED_IP" ]; then
  echo "DNS OK"
else
  echo "DNS ERREUR: attendu $EXPECTED_IP, obtenu $ACTUAL_IP"
  exit 1
fi
```

---

## Synthèse GRC

### ISO 27001 et DNS

* **A.12.3.1 - Disponibilité des services**: Déployer des serveurs DNS secondaires, monitorer la santé DNS
* **A.12.3.2 - Sécurité des services réseau**: Configurer DNSSEC, bloquer les requêtes DNS non autorisées
* **A.13.1.3 - Logging et monitoring**: Activer les logs DNS pour audit
* **A.14.2.1 - Continuité de service**: DNS redondant, failover automatique

### NIS2 et DNS

* DNS est service essentiel pour les opérateurs critiques (télécoms, électricité, services publics)
* Obligation de résilience: minimum 2 serveurs DNS
* Obligation de monitoring et d'alertes sur les défaillances DNS

### DORA (Digital Operational Resilience Act)

* DNS fait partie de la chaîne critique d'infrastructure numérique
* Audit et test de continuité de service DNS obligatoires

### OWASP DNS

* DNS Amplification Attacks: limiter les requêtes récursives vers des IPs externes
* DNS Spoofing: utiliser DNSSEC, valider les certificats TLS
* DNS Exfiltration: monitorer les requêtes DNS sortantes anormales

---

## Commandes de référence rapide

```bash
# Requête simple
dig example.com

# Réponse courte
dig example.com +short

# Serveur DNS spécifique
dig @8.8.8.8 example.com

# Type spécifique
dig example.com MX
dig example.com NS
dig example.com TXT

# Reverse DNS
dig -x 192.0.2.1

# Trace complet
dig example.com +trace

# Statistiques
dig example.com +stats

# Ancienne syntaxe (nslookup)
nslookup example.com
nslookup -type=MX example.com
```

---

**Auteur:** Pierre | **Formation:** Jedha Essentials + Fullstack (2026)  
**Dernière mise à jour:** Mai 2026  
**Niveau:** Débutant / Intermédiaire
