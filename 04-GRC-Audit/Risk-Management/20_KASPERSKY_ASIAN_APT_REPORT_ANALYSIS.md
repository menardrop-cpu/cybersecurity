# Kaspersky modern Asian APT groups — threat intelligence analysis

**Report**: Modern Asian APT groups TTPs report  
**Source**: Kaspersky 2023  
**Régions analysées**: Russia & Belarus, Indonesia  
**Statut**: ✓ analysé et documenté

---

## Ce que j'ai appris

Ce rapport Kaspersky m'a montré comment analyser les TTPs (tactics, techniques, procedures) d'APTs asiatiques réels en utilisant:
- Le framework MITRE ATT&CK Navigator
- L'analyse des malwares et backdoors
- La cartographie des incidents à travers plusieurs régions
- L'OSINT pour identifier les groupes APT

---

# Section 1: Russia & Belarus Incident

## Vue d'ensemble

Un incident sophistiqué impliquant l'APT **CoughingDown** (connecté au backdoor **EAGERBEE**). L'attaque a ciblé l'infrastructure Windows en exploitant des services Internet Information Services (IIS).

## Questions et réponses

### Q1: Nombre de techniques MITRE ATT&CK reportées

**Réponse**: 21 techniques

**Source**: Utilisation de MITRE ATT&CK Navigator pour analyser le rapport

### Q2: Nom du malware principal

**Réponse**: WebDav-O

**Signification**: WebDav-O est le malware utilisé pour établir une communication persistante et exfiltrer des données via Yandex Disk.

### Q3: Système exploité pour l'accès initial

**Réponse**: IIS Windows Server

**Exploitation**: L'attaquant a exploité une vulnérabilité dans Internet Information Services (IIS) pour gagner l'accès initial au serveur Windows.

### Q4: Fichier créant communication avec Yandex Disk (MD5 Hash)

**Réponse**: 69B99401A0BBBF7BEC1B27DCE12C8B3A

**Signification**: Ce hash identifie le fichier malveillant responsable de la communication avec Yandex Disk pour exfiltrer les données.

### Q5: Nom du groupe APT

**Réponse**: CoughingDown

**Type**: APT groupe associé aux menaces asiatiques

### Q6: Backdoor spécifique associé (OSINT)

**Réponse**: EAGERBEE

**Méthode OSINT**: 
- Rechercher "CoughingDown APT"
- Consulter les bases de données de threat intelligence (Mandiant, Crowdstrike, Kaspersky)
- Corréler avec les rapports de malware analysis
- EAGERBEE est le backdoor utilisé par CoughingDown

### Q7: Fichier utilisé pour reconnaissance sur hôtes distants

**Réponse**: wmic.exe

**Technique MITRE**: T1047 (Windows Management Instrumentation)

**Usage**: wmic.exe est utilisé pour interroger les systèmes distants et rassembler des informations.

### Q8: Commande pour découvrir logiciels installés

**Réponse**: net use

**Technique MITRE**: T1049 (System network discovery)

**Usage**: net use établit des connexions réseau et permet de découvrir les ressources partagées.

### Q9: Outil pour dump mémoire de processus

**Réponse**: procdump.exe

**Technique MITRE**: T1003 (Credential access: OS Credential dumping)

**Usage**: Dump la mémoire de processus pour extraire les credentials.

### Q10: Outil pour envoyer data au C2

**Réponse**: HTran

**Technique MITRE**: T1071 (Command and Control: Application layer protocol)

**Usage**: HTran est un outil de tunneling utilisé pour communiquer avec le serveur C2.

---

## Synthèse Russia & Belarus Incident

### La chaîne d'attaque

```
Exploitation IIS Windows Server (Initial Access)
    ↓
WebDav-O malware deployment (Execution)
    ↓
Yandex Disk communication (Command & Control)
    ↓
Reconnaissance avec wmic.exe (Discovery)
    ↓
Credential dumping via procdump.exe (Credential Access)
    ↓
HTran exfiltration vers C2 (Exfiltration)
```

### Techniques utilisées (21 total)

Les 21 techniques MITRE ATT&CK mappées incluent:
- Exploitation initiale (IIS)
- Execution (WebDav-O)
- Persistence (via scheduled tasks ou registry)
- Privilege escalation
- Discovery (wmic.exe, net use)
- Credential access (procdump.exe)
- Command & Control (HTTPran vers Yandex Disk)
- Exfiltration

---

# Section 2: Indonesia Incident

## Vue d'ensemble

Un incident impliquant l'APT **GhostEmperor**. L'attaque a utilisé des techniques sophistiquées de DLL Side-Loading, injection de processus, et credential dumping.

## Questions et réponses

### Q1: Nombre de techniques MITRE reportées

**Réponse**: 21 techniques

**Source**: MITRE ATT&CK Navigator analysis

### Q2: Nombre de techniques qui chevauchent Russia & Belarus

**Réponse**: 11 techniques

**Signification**: 11 techniques sont communes entre les deux incidents. Cela suggère une évolution TTPs similaire ou une infrastructure partagée.

### Q3: Groupe APT identifié

**Réponse**: GhostEmperor

**Type**: APT groupe asiatique

### Q4: Méthode principale de persistance

**Réponse**: T1574.002 (Hijack execution flow: DLL Side-Loading)

**Explanation**: GhostEmperor a utilisé le DLL side-loading pour maintenir la persistance. Un processus légitime charge une DLL malveillante au lieu de la DLL attendue.

### Q5: Logiciel légitime abusé pour DLL Side-Loading

**Réponse**: MEUpdate.exe

**Technique MITRE**: T1574.002 (DLL Side-Loading)

**Explication**: MEUpdate.exe (processus légitime) a été exploité pour charger une DLL malveillante.

### Q6: MD5 hash de la DLL malveillante

**Réponse**: 6D72C024B804CF690C7E7E8A7135EDB0

**Signification**: Ce hash identifie précisément la DLL malveillante utilisée dans l'attaque.

### Q7: Utilitaire Windows pour télécharger l'exécutable malveillant

**Réponse**: certutil.exe

**Technique MITRE**: T1105 (Ingress tool transfer)

**Usage**: certutil.exe est utilisé pour télécharger les outils malveillants depuis le C2.

### Q8: Trois adresses IP C2 utilisées

**Réponse**: 47.96.167.205, 8.210.141.104, 23.224.91.98

**Signification**: Ces trois IPs sont les serveurs Command & Control utilisés par GhostEmperor pour la communication.

### Q9: Technique d'injection de payload

**Réponse**: T1055.012 (Process injection: Process hollowing)

**Explication**: Process hollowing est utilisé pour injecter le payload malveillant dans un processus légitime.

### Q10: Processus Windows créé et injecté

**Réponse**: svchost.exe

**Explication**: svchost.exe (processus système légitime) a été créé et injecté avec du code malveillant pour des opérations discrètes.

### Q11: Outil PowerShell pour credential dumping

**Réponse**: DumpMinitool.exe

**Technique MITRE**: T1003.001 (OS Credential dumping: LSASS memory)

**Usage**: Dump les credentials depuis la mémoire LSASS.

### Q12: Commande de reconnaissance réseau

**Réponse**: ipconfig.exe /all

**Technique MITRE**: T1016 (System network discovery)

**Usage**: Discover la configuration réseau complète de la victime.

### Q13: Outil de credential dumping LSASS

**Réponse**: mimikat_ssp

**Technique MITRE**: T1003 (Credential access: OS Credential dumping)

**Usage**: Variante de Mimikatz utilisée pour extraire les credentials.

### Q14: Technique de credential dumping

**Réponse**: T1003 (OS Credential dumping)

**Sous-techniques associées**:
- T1003.001 (LSASS memory)
- T1003.002 (Security account manager)
- T1003.004 (LSA secrets)

### Q15: Technique de collection d'informations de sécurité

**Réponse**: T1518.001 (Software discovery: Security software discovery)

**Usage**: Découvrir les logiciels de sécurité installés sur le système victime.

### Q16: Utilitaire pour exfiltrer les données

**Réponse**: curl

**Technique MITRE**: T1041 (Exfiltration over C2 channel)

**Usage**: curl est utilisé pour envoyer les données volées vers le serveur C2.

### Q17: Service cloud pour exfiltrer les fichiers

**Réponse**: file.io

**Technique MITRE**: T1537 (Transfer data to cloud account)

**Usage**: Les données sont exfiltrées vers file.io, un service cloud d'hébergement de fichiers.

---

## Synthèse Indonesia Incident

### La chaîne d'attaque

```
MEUpdate.exe DLL Side-Loading (Initial Access)
    ↓
Malicious DLL injection (Execution & Persistence)
    ↓
certutil.exe téléchargement payload (Execution)
    ↓
Process hollowing dans svchost.exe (Execution)
    ↓
ipconfig reconnaissance (Discovery)
    ↓
DumpMinitool.exe credential dumping (Credential Access)
    ↓
curl exfiltration vers C2 (Exfiltration)
    ↓
file.io cloud storage (Data staging)
```

### Techniques utilisées (21 total)

Les 21 techniques incluent:
- DLL side-loading (T1574.002)
- Process injection (T1055.012)
- Credential dumping (T1003)
- Discovery (T1016, T1518.001)
- Exfiltration (T1041, T1537)
- Command & Control

---

# Comparaison: Russia & Belarus vs Indonesia

## Techniques communes (11)

Les 11 techniques communes indiquent une stratégie d'attaque similaire:
- Discovery (reconnaissance systèmes)
- Credential access (dumping)
- Command & Control
- Exfiltration

## Techniques différentes

**Russia & Belarus spécifiques**:
- Exploitation IIS
- WebDav-O malware
- Yandex Disk C2

**Indonesia spécifiques**:
- DLL side-loading
- Process hollowing
- file.io cloud storage
- DumpMinitool.exe

## Implications

La présence de 11 techniques communes suggère:
- Une TTP partagée (même groupe ou inspiration)
- Une évolution de tactiques entre les incidents
- Une adaptation aux défenses locales

---

# Partie 3: MITRE ATT&CK Mapping Complet

## Russia & Belarus - 21 Techniques

Les 21 techniques couvrent:
- **Initial access**: Exploitation IIS
- **Execution**: WebDav-O malware
- **Persistence**: Registry/scheduled tasks
- **Privilege escalation**: (selon configuration)
- **Defense evasion**: Obfuscation
- **Credential access**: procdump.exe
- **Discovery**: wmic.exe, net use
- **Collection**: (data gathering)
- **Command & Control**: HTTPran + Yandex Disk
- **Exfiltration**: HTTPran vers C2

## Indonesia - 21 Techniques

Les 21 techniques couvrent:
- **Initial access**: DLL side-loading
- **Execution**: MEUpdate.exe, certutil.exe
- **Persistence**: DLL side-loading (T1574.002)
- **Privilege escalation**: Process hollowing
- **Defense evasion**: (technique masquage)
- **Credential access**: DumpMinitool.exe, mimikat_ssp (T1003)
- **Discovery**: ipconfig.exe, wmic.exe
- **Collection**: T1518.001 (security software)
- **Command & Control**: IPs C2
- **Exfiltration**: curl (T1041), file.io (T1537)

---

# Partie 4: Pyramid of Pain - Application

## Ce que les APTs peuvent changer facilement

- **IPs C2**: Changer rapidement (nouvelles adresses VPS)
- **Domaines**: Enregistrer de nouveaux domaines
- **Hashes malware**: Modifier le code = nouveau hash

**Exemple**: Les 3 IPs C2 de GhostEmperor peuvent être changées en heures.

## Ce que les APTs ne peuvent pas changer facilement

- **TTPs**: DLL side-loading, process hollowing, credential dumping
- **Infrastructure**: Processus légitime exploités (MEUpdate.exe, svchost.exe)
- **Tools**: certutil.exe, curl, procdump.exe

**Implication**: Détecter sur les **TTPs** plutôt que les IoCs.

---

# Partie 5: OSINT pour identifier les APTs

## CoughingDown (Russia & Belarus)

**Méthode OSINT**:
1. Chercher "CoughingDown APT"
2. Vérifier threat intelligence databases
3. Correlate avec EAGERBEE backdoor
4. Consulter les rapports Kaspersky, Mandiant, etc.

**Résultat**: APT groupe russe/biélorusse

## GhostEmperor (Indonesia)

**Méthode OSINT**:
1. Chercher "GhostEmperor APT"
2. Vérifier les hashes malware sur VirusTotal
3. Analyser les C2 IPs (geolocation, reputation)
4. Correlate avec les tactiques de DLL side-loading

**Résultat**: APT groupe asiatique (probablement chinois ou coréen)

---

# Partie 6: Ce que j'ai appris

## 1. Les APTs asiatiques partagent des TTPs

Les 11 techniques communes entre Russia & Belarus et Indonesia suggèrent:
- Une évolution similaire des stratégies
- Une partage possible de knowledge
- Une adaptation aux défenses régionales

## 2. DLL Side-Loading est une technique populaire

GhostEmperor utilise massivement DLL side-loading (T1574.002). C'est une technique difficile à détecter car elle abuse de processus légitimes.

## 3. Credential dumping est la cible finale

Les deux incidents utilisent le credential dumping (T1003) pour accéder aux comptes administratifs. C'est toujours l'objectif principal.

## 4. Cloud storage est utilisé pour exfiltration

GhostEmperor exfiltre les données vers **file.io**, un service cloud public. C'est discret car file.io est un service légitime.

## 5. Reconnaissance est la phase la plus longue

Les deux incidents commencent par discovery (wmic.exe, ipconfig.exe) pour mapper l'environnement cible. Cette phase est silencieuse et dure semaines.

---

# Partie 7: Application pratique

## Si je détectais Russia & Belarus Incident

**Étapes immédiates**:
1. Chercher tous les fichiers avec hash 69B99401A0BBBF7BEC1B27DCE12C8B3A
2. Bloquer les connexions vers Yandex Disk
3. Auditer l'historique wmic.exe et procdump.exe
4. Dumper la mémoire pour trouver WebDav-O
5. Révoquer les credentials compromis

**Défenses**:
- Bloquer IIS exploitation (patcher, WAF)
- Monitorer les connexions Yandex Disk
- Alerter sur procdump.exe execution
- MFA sur tous les comptes

## Si je détectais Indonesia Incident

**Étapes immédiates**:
1. Chercher MEUpdate.exe anormal
2. Auditer certutil.exe downloads
3. Chercher process hollowing (svchost.exe avec code malveillant)
4. Dumper la mémoire LSASS
5. Bloquer les 3 IPs C2

**Défenses**:
- Bloquer DLL side-loading (application whitelisting)
- Monitorer certutil.exe usage
- Détecter process hollowing (EDR)
- Dumper LSASS en boucle
- Alerter sur curl exfiltration

---

# Résumé des incidents

## Russia & Belarus

**APT**: CoughingDown (EAGERBEE backdoor)  
**Malware principal**: WebDav-O  
**Accès initial**: IIS Windows Server  
**Persistence**: Yandex Disk C2  
**Techniques**: 21 (exploitation, discovery, credential dumping, exfiltration)  
**Indicateurs**: Hash 69B99401A0BBBF7BEC1B27DCE12C8B3A, HTTPran tool  

## Indonesia

**APT**: GhostEmperor  
**Malware principal**: Malicious DLL (hash 6D72C024B804CF690C7E7E8A7135EDB0)  
**Accès initial**: DLL side-loading MEUpdate.exe  
**Persistence**: Process hollowing svchost.exe  
**Techniques**: 21 (DLL side-loading, process injection, credential dumping, cloud exfil)  
**Indicateurs**: IPs C2 (47.96.167.205, 8.210.141.104, 23.224.91.98), certutil usage  

---

**Status**: ✓ Rapport Kaspersky analysé et documenté

**Ce que j'ai appris**: Comment analyser les TTPs d'APTs réels, comparer les incidents, identifier les patterns

**Utilité**: 5/5

**Application**: Incident response, threat hunting, APT tracking, defense strategy

---

Les APTs asiatiques utilisent des techniques sophistiquées mais prévisibles. La détection dépend de la surveillance des **TTPs** (DLL side-loading, process injection, credential dumping) plutôt que des IoCs changeants.
