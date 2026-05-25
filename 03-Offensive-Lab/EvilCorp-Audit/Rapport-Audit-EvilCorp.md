# Final Project Jedha: Infrastructure Design & Security

## Vue d'ensemble du projet

**Durée**: 10 jours  
**Objectif**: Concevoir, déployer, sécuriser et documenter une infrastructure IT complète pour une PME fictive  
**Type**: Projet de groupe (avec phases de testing et présentation)

---

# Context & Missions

## Situation

Tu es embauché comme **technical lead** par une PME en transformation digitale. L'entreprise a besoin d'une infrastructure fiable, sécurisée et scalable.

## Deux missions

### Mission I: Conception et déploiement (Jours 1-5)

En tant que leader technique, tu dois:
1. **Concevoir** une infrastructure complète
2. **Déployer** tous les composants
3. **Sécuriser** le système
4. **Documenter** chaque décision
5. **Préparer** la présentation

### Mission II: Testing de sécurité (Jours 6-8)

Tester l'infrastructure d'une autre équipe et:
1. Identifier les vulnérabilités
2. Écrire un pentest report
3. Aider l'autre équipe à s'améliorer

---

# Choisir ton profil d'entreprise

Tu dois choisir **UN** profil parmi les 4. Chaque a ses spécificités.

## 1. Medical Office (Cabinet médical) 🏥

**Focus principal**: Confidentialité, Conformité, Backups

**Contexte**:
- Gestion de dossiers patients (données sensibles)
- Conformité RGPD, HIPAA (si US)
- Audit trails critiques
- Chiffrement obligatoire

**Infrastructure type**:
- Database patient (chiffrée, HIPAA-compliant)
- Secure File Server (dossiers patients)
- Backup régulier + archivage long-terme
- Access control strict par rôle
- Audit logging complet

**Défi principal**: Protéger les données patient = sanction si breached

---

## 2. Remote-first Startup 🌐

**Focus principal**: VPN, Cloud-based, Remote Access

**Contexte**:
- Équipe distribuée (100% remote)
- Besoin d'accès depuis n'importe où
- Scalabilité cloud
- Collaboration temps-réel

**Infrastructure type**:
- VPN centralizado (OpenVPN, WireGuard)
- Cloud infrastructure (AWS/Azure)
- SSO/MFA pour accès
- Collaboration tools (Nextcloud, Slack)
- Network segmentation

**Défi principal**: Sécuriser les accès remote = risque VPN

---

## 3. E-Commerce Site 🛒

**Focus principal**: Availability, Redundancy, Scalability

**Contexte**:
- Besoin 24/7 uptime
- Gestion de paiements (PCI-DSS)
- Trafic variable (pics ventes)
- Données clients sensibles

**Infrastructure type**:
- Load balancing + haute disponibilité
- Database replication
- Cache layer (Redis)
- Auto-scaling
- Monitoring 24/7

**Défi principal**: Downtime = perte d'argent immédiate

---

## 4. Architecture Firm 🏢

**Focus principal**: Storage, Collaboration Tools

**Contexte**:
- Fichiers énormes (plans, rendus 3D, vidéos)
- Besoin de partage fichiers sécurisé
- Versioning et archivage
- Collaboration sur projets

**Infrastructure type**:
- NAS/Storage serveur haute capacité
- Samba/NFS pour partage
- Nextcloud pour collaboration
- Backup de la taille énorme
- Version control

**Défi principal**: Gérer les volumes de données

---

## 5. NGO (ONG) 🤝

**Focus principal**: Low budget, Open-source stack

**Contexte**:
- Budget très limité
- Stack 100% open-source
- Flexibilité importante
- Sustainability

**Infrastructure type**:
- Server minimaliste (même un RPi)
- Linux seulement
- Nextcloud pour tout
- Self-hosted
- Monitoring simple

**Défi principal**: Faire maximum avec minimum

---

# Composants techniques obligatoires

**TOUS les projets doivent inclure**:

✅ **Web Server**  
   - Apache, Nginx, ou Node.js  
   - Portal interne ou site public  
   - HTTPS obligatoire

✅ **Database**  
   - SQL (PostgreSQL, MySQL) ou NoSQL (MongoDB)  
   - Connexion sécurisée à l'app  
   - Backup régulier

✅ **File Server/Drive**  
   - Samba, NFS, Nextcloud, OwnCloud  
   - Accès contrôlé par rôle

✅ **Active Directory**  
   - Gestion centralisée des identités  
   - Windows Server AD ou Samba AD  
   - SSO si possible

✅ **Backup & Monitoring**  
   - Wazuh, Zabbix, Prometheus  
   - S3, ou storage local  
   - Alertes en temps-réel

✅ **Security**  
   - Firewall (UFW, pfSense)  
   - Policies d'accès (least privilege)  
   - IDS/IPS optionnel

---

# Décisions architecturales à prendre

## Infrastructure: On-premise, Cloud, ou Hybrid?

| Type | Avantages | Désavantages | Quand? |
|------|-----------|--------------|--------|
| **On-premise** | Contrôle total, coût fixe | Maintenance, scaling difficile | Medical, privacy-critical |
| **Cloud** (AWS/Azure) | Scalabilité, managed services | Coûts variables, vendor lock-in | Startup, E-commerce |
| **Hybrid** | Flexibilité, meilleur des deux | Complexité, multi-management | Architecture firm |

## Containers ou VMs?

| Option | Avantages | Désavantages | Quand? |
|--------|-----------|--------------|--------|
| **VMs** | Isolation forte, OS complet | Lourd, lent | On-premise, legacy |
| **Docker** | Léger, rapide, portable | Apprentissage, orchestration | Startup, DevOps |
| **Kubernetes** | Orchestration avancée | Très complexe | E-commerce, scalabilité |

## Automatisation?

| Option | Avantages | Désavantages | Quand? |
|--------|-----------|--------------|--------|
| **Scripts** | Simple, flexible | Non-scalable, error-prone | Petite infra |
| **Ansible** | Idempotent, simple | Agents requis parfois | Medium infra |
| **Terraform** | Infrastructure-as-Code | Courbe apprentissage | Cloud, IaC |
| **Kubernetes** | Orchestration complète | Très complexe | Scalabilité |

## Threat modeling?

Toujours faire:
1. Identifier les assets (données critiques)
2. Identifier les menaces (attack vectors)
3. Évaluer les risques
4. Implémenter des contrôles

---

# Livrables requis

## 1. Technical Specification Document 📄

**Contenu minimum**:
- Analyse des besoins (par profil d'entreprise)
- Network diagrams (topologie, flux)
- Tech choices justifiées
- Risk analysis (menaces identifiées)
- Security controls (comment on mitigue)
- Compliance (RGPD, HIPAA, etc.)
- Backup strategy + RTO/RPO

**Format**:
- Document professionnel (Word, PDF, ou Markdown)
- 15-30 pages
- Diagrammes clairs

## 2. Infrastructure Deployment 🛠️

**Requis**:
- ✓ AD (Windows Server ou Samba) opérationnel
- ✓ Database connectée et sécurisée
- ✓ Web app fonctionnelle
- ✓ File share accessible
- ✓ Monitoring actif
- ✓ Firewalls configurés

**Bonus**:
- Containers (Docker)
- Automation (Ansible, Terraform)
- Replication/HA
- Encryption complète

## 3. Project Management File 🗂️

**Contenu**:
- Kanban board (Trello, GitHub Projects, Notion)
- Planning détaillé (timeline)
- Budget estimate
- Distribution des rôles
- Agile practices (sprints, standups)

**Exemple Notion**:
```
Tâches:
- [ ] Network design (2j)
- [ ] AD setup (3j)
- [ ] Database config (2j)
- [ ] Web server (2j)
- [ ] Backup config (1j)
- [ ] Security hardening (2j)
- [ ] Testing (1j)
- [ ] Documentation (2j)
```

## 4. Technology Watch Report 📋

**Format**: 2-3 pages en anglais

**Objectif**: Identifier 1-2 technologies/standards clés du projet

**Exemples**:
- "Kubernetes for container orchestration in e-commerce"
- "HIPAA compliance in healthcare infrastructure"
- "Zero-trust architecture in remote-first environments"

**Structure**:
1. Technologie/standard
2. Pourquoi c'est important
3. Comment tu l'as utilisé
4. Références

## 5. Pentest Report ✍️

**Scope**: Infrastructure de l'autre groupe

**Contenu**:
- Executive summary
- Methodology (NIST, OWASP)
- Findings par sévérité (Critical, High, Medium, Low)
- Proof-of-concept (PoC)
- Recommendations

**Format**:
- Professionnel
- 10-20 pages
- Preuves des vulnérabilités

## 6. Final Presentation 🗣️

**Durée**: 10-15 minutes

**Slides à couvrir**:
1. Contexte et objectifs (1 slide)
2. Architecture overview (2 slides)
3. Décisions techniques (2 slides)
4. Security measures (2 slides)
5. Challenges & solutions (1 slide)
6. Lessons learned (1 slide)
7. Demo si possible (2 slides)
8. Q&A

**Tips**:
- Pratiquer le timing
- Parler distinctement
- Pas de slides trop denses
- Photos de l'infrastructure

---

# Timeline détaillée

## Phase I: Building (Jours 1-5)

### Jour 1: Planning & Design
- [ ] Choisir le profil entreprise
- [ ] Brainstorm architecture
- [ ] Créer Kanban board
- [ ] Commencer Tech Spec Document

### Jour 2-3: Network & AD
- [ ] Design topologie réseau
- [ ] Setup Active Directory
- [ ] Configurer users/groups
- [ ] Test authentification

### Jour 4: Database & Web
- [ ] Setup database
- [ ] Configurer backup
- [ ] Deploy web app
- [ ] Test connectivity

### Jour 5: Security & Monitoring
- [ ] Setup firewall
- [ ] Install monitoring (Wazuh)
- [ ] Configure alertes
- [ ] Initial security testing

### Jour 5 (fin): Final touches
- [ ] Finir Tech Spec Document
- [ ] Finir Project Management file
- [ ] Créer diagrams finaux
- [ ] Tester tout de bout en bout

---

## Phase II: Testing (Jours 6-8)

### Jours 6-8: Security Assessment
- [ ] Recevoir infrastructure autre équipe
- [ ] Faire reconnaissance (Nmap)
- [ ] Tester vulnérabilités
- [ ] Écrire pentest report
- [ ] Partager findings

---

## Phase III: Presentation (Jours 9-10)

### Jour 9: Prep
- [ ] Finaliser slides
- [ ] Pratiquer présentation
- [ ] Préparer demo
- [ ] Vérifier équipement

### Jour 10: Go Live!
- [ ] Présentation (10-15 min)
- [ ] Q&A
- [ ] Feedback

---

# Critères d'évaluation

| Axe | Critères | Weight |
|-----|----------|--------|
| **Qualité technique** | Infrastructure fonctionnelle, sécurisée, bien intégrée | 30% |
| **Documentation** | Clés, complètes, standards professionnels | 20% |
| **Sécurité** | Firewall, access controls, backup, encryption | 25% |
| **Agile & PM** | Kanban, planning réaliste, progress tracking | 10% |
| **Communication** | Présentation claire, documents pro | 10% |
| **Innovation** | Containers, cloud, scripting, monitoring avancé | 5% |

---

# Outils recommandés par domaine

## Infrastructure

- **GNS3**: Network simulation
- **Proxmox**: Hypervisor (on-premise)
- **AWS/Azure**: Cloud
- **Docker**: Containers
- **Kubernetes**: Orchestration

## Monitoring & Security

- **Wazuh**: SIEM + monitoring
- **Zabbix**: Infrastructure monitoring
- **Prometheus**: Metrics
- **Suricata**: IDS/IPS
- **pfSense**: Firewall

## Web & Database

- **Nginx/Apache**: Web server
- **PostgreSQL/MySQL**: SQL DB
- **MongoDB**: NoSQL
- **Node.js/Django**: App framework

## File Server

- **Samba**: SMB/CIFS
- **NFS**: Network File System
- **Nextcloud**: Cloud storage
- **Minio**: Object storage (S3)

## Directory Services

- **Windows Server AD**: Enterprise
- **Samba AD**: Open-source alternative

## DevOps & IaC

- **Git**: Version control
- **Ansible**: Configuration management
- **Terraform**: Infrastructure as Code
- **Jenkins**: CI/CD

## Project Management

- **Trello**: Kanban simple
- **GitHub Projects**: Intégré à GitHub
- **Notion**: Flexible + docs
- **Jira**: Agile professionnel (overkill?)

---

# Tips & Best Practices

## Avant de commencer

- [ ] Tous les membres comprennent le projet
- [ ] Roles clairs (qui fait quoi?)
- [ ] Environment de test + production séparé
- [ ] Backup de ton backup (2x backup!)

## Pendant le build

- [ ] Documenter au fur et à mesure (pas à la fin!)
- [ ] Tester chaque composant isolé
- [ ] Puis tester l'intégration
- [ ] Security mindset depuis le départ
- [ ] Committer le code/config régulièrement

## Documentation

- [ ] Screenshots de chaque config
- [ ] Explain the "why", pas juste le "how"
- [ ] Diagrammes professionnels (Lucidchart, Draw.io)
- [ ] Checklist de déploiement

## Présentation

- [ ] Pratiquer le timing (10-15 min, c'est court!)
- [ ] Pas de slide dense
- [ ] Démo live si possible (mais avec fallback = vidéo)
- [ ] Story-telling: contexte → problem → solution → results

---

# Ressources disponibles

## Lab GNS3

**Config**: 40GB RAM, 8vCPU, 230GB storage

**Important**:
- Utiliser **NAT node** (pas Cloud node!)
- SSH tunnel pour accéder aux VMs
- Garder le lab up during project

## Documentation Jedha

- Tous les modules couverts
- Best practices
- Security guidelines

## Externe

- NIST Framework
- OWASP Top 10
- CIS Benchmarks
- RFC (DNS, TLS, etc.)

---

# Checklist avant D-Day

### 48h avant présentation

- [ ] Infrastructure fully tested
- [ ] Tech Spec Document finalisé
- [ ] Pentest report prêt
- [ ] Slides de présentation OK
- [ ] Avoir un backup du lab (snapshot)
- [ ] Tester la projection/audio

### 24h avant

- [ ] Full run-through (timing + tech)
- [ ] Internet de présentation testé
- [ ] Toutes les slides finales
- [ ] Notes de présentation imprimées

### Day of

- [ ] Arrive 30 min early
- [ ] Test projector/audio/network
- [ ] Calm breathing (c'est un projet cool!)
- [ ] Enjoy the moment! 🎉

---

# Demoday vs Certification

## Demoday (Phase I)

**Objectif**: Showcase your work to audience

**Durée présentation**: 10-15 min

**Prérequis**:
- Pas besoin d'être 100% complet
- Infrastructure fonctionnelle suffisante
- Peut être enregistré (pour recruiters!)

## Certification (Phase II - si applicable)

**Objectif**: Évaluation officielle par jury

**Durée**: Peut être plus longue

**Prérequis**:
- TOUS les livrables complets
- Infrastructure fully secured
- Questions spécifiques possibles
- Evaluation formelle

---

# Note finale 🎯

Ce projet est ton opportunity de:
- Montrer tes skills réels
- Démontrer ton leadership technique
- Créer quelque chose de concret
- Te préparer pour les vrais jobs

**Make it count!** 🚀

---

## Questions à te poser maintenant

1. **Quel profil d'entreprise tu choisirais?** (et pourquoi?)
2. **On-premise ou Cloud?** (analyse coûts-bénéfices)
3. **Quels sont tes points forts à mettre en avant?**
4. **Quels tools tu maîtrises déjà?**
5. **Où tu veux apprendre le plus?**

Start thinking about it! Vous avez 3 mois pour vous préparer. 💪
