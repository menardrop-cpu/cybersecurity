# microsoft midnight blizzard incident - threat intelligence analysis

**incident** : nation-state attack on microsoft corporate systems  
**date détectée** : january 12, 2024  
**apt responsable** : midnight blizzard (apt29/nobelium)  
**statut** : ✓ compris et analysé

---

## ce que j'ai appris

cet incident m'a montré comment analyser une attaque de nation-state réelle en utilisant:
- les documents de divulgation SEC (8-K filings)
- le framework MITRE ATT&CK
- l'osint pour identifier les APTs
- la cartographie des attaques réelles

---

# partie 1 : timeline de l'incident

## dates clés

| événement | date |
|-----------|------|
| intrusion initiale | late november 2023 |
| détection | january 12, 2024 |
| accès supprimé | january 13, 2024 |
| divulgation publique | january 19, 2024 |
| communication SEC | march 8, 2024 |

### pourquoi cette timeline est importante?

- **late november → january 12** : 6+ semaines d'accès non détecté
- **january 12 → january 13** : réponse ultra-rapide (< 24h)
- **january 19 → march 8** : évolution de l'attaque découverte après

---

# partie 2 : identifier l'apt

## l'attaquant: midnight blizzard

**noms alternatifs:**
- apt29 (FireEye naming)
- nobelium (Microsoft naming)
- cozy bear (Crowdstrike naming)

**pays d'origine:** russie 🇷🇺  
**type:** nation-state, cyber-espionnage  
**motivation:** intelligence gathering gouvernementale

## comment identifier l'apt via osint?

l'exercice demande "using osint, what is the apt number?"

**réponse:** apt29

**méthode OSINT pour trouver ça:**
1. chercher "midnight blizzard" sur google
2. vérifier les rapports de crowdstrike/mandiant
3. corréler avec les comportements connus
4. trouver la liste des apt numérotés par FireEye

**résultat:** midnight blizzard = apt29

---

# partie 3 : mapper aux mitre att&ck

## l'attaque step-by-step

### étape 1: password spray attack

**technique mitre:** t1110.003 - credential access: password spray

**ce qui s'est passé:**
- attaquant a ciblé des comptes de test legacy
- ont testé de nombreuses combinaisons mot de passe
- objectif: trouver des comptes avec des credentials faibles

**pourquoi c'est efficace:**
- comptes de test souvent moins sécurisés
- pas de MFA sur les systèmes legacy
- volume d'attaques impossible à bloquer manuellement

### étape 2: initial compromise

**technique mitre:** t1078.004 - valid accounts: cloud accounts

**ce qui s'est passé:**
- foothold obtenu via un compte de test non-production
- accès au tenant Azure/Office365
- capacité à accéder à d'autres comptes

### étape 3: email access et exfiltration

**technique mitre:** t1114.002 - email collection: remote email collection

**ce qui s'est passé:**
- utilisé les permissions du compte compromis
- accédé aux emails d'executives et cybersecurity team
- exfiltré emails et documents attachés

**pourquoi cela? information gathering sur apt29 lui-même**

### étape 4: escalation découverte plus tard

**technique mitre:** t1555.001 - credentials from password stores

**ce qui s'est passé en février/mars:**
- apt29 a utilisé les secrets trouvés dans les emails
- attempts d'accès à source code repositories
- password sprays 10x plus importants

---

# partie 4 : répondre aux questions

## q1: date de détection initiale?

**réponse:** january 12, 2024

**source:** "On January 12, 2024, Microsoft detected that beginning in late November 2023..."

## q2: quel nation-state actor?

**réponse:** midnight blizzard (aussi connu comme nobelium)

**source:** "Microsoft has identified the threat actor as Midnight Blizzard, the Russian state-sponsored actor also known as Nobelium."

## q3: apt number (osint)?

**réponse:** apt29

**méthode osint:**
- chercher "midnight blizzard apt"
- vérifier les bases de données d'apt (mandiant, crowdstrike)
- midnight blizzard est l'alias pour apt29

## q4: quel pays?

**réponse:** russie

**source:** "Russian state-sponsored actor"

## q5: type d'attaque pour accès initial (mitre)?

**réponse:** t1110.003 (credential access: password spray)

**ou:** t1078 (valid accounts) si on considère l'utilisation du compte compromis

**explication:**
- attack method = password spray
- mitre technique = t1110.003
- c'est de la credential access

## q6: extraction de data repositories?

**réponse:** t1213 (information repositories)

**sous-technique spécifique:** t1213.02 - information repositories: sharepoint

**ou:** t1552.007 - unsecured credentials: container images and registry

**explication:**
- apt29 a accédé à source code repositories
- après avoir exfiltré les emails
- les secrets trouvés dans les emails ont permis l'accès

## q7: date suppression d'accès?

**réponse:** january 13, 2024

**source:** "We were able to remove the threat actor's access to the email accounts on or about January 13, 2024."

## q8: objectif primaire de l'attaque?

**réponse (copie exacte du rapport):**

"the investigation indicates they were initially targeting email accounts for information related to midnight blizzard itself"

**analyse:** l'apt29 ciblait microsoft pour en savoir plus sur... lui-même! reconnaissance interne.

## q9: quel système microsoft a été compromis initialement?

**réponse:** legacy non-production test tenant account

**ou simplement:** test tenant / test account

**source:** "the threat actor used a password spray attack to compromise a legacy non-production test tenant account"

## q10: exploit d'une vulnerability microsoft?

**réponse:** no

**source:** "The attack was not the result of a vulnerability in Microsoft products or services."

**important:** ce n'était pas une zero-day, mais une attaque basée sur:
- credentials faibles
- systèmes legacy non sécurisés
- configuration défaillante

---

# partie 5 : analyse de l'attaque réelle

## la stratégie d'apt29

### phase 1: reconnaissance (novembre 2023)

- cibler microsoft intentionnellement
- chercher des comptes avec défenses faibles
- comptes de test = cible parfaite

### phase 2: intrusion (novembre - janvier)

- password spray massif contre test tenant
- compromission réussie
- escalade vers email accounts d'executives

### phase 3: exfiltration (janvier)

- accès aux emails de senior leadership
- cybersecurity team emails (auto-reconnaissance)
- documents attachés volés

### phase 4: continuation (février - mars)

- utilisation des secrets trouvés
- source code repository access
- password sprays 10x augmentés

## pourquoi c'est difficile à détecter?

**password spray:**
- volume énorme d'attempts
- distribuées sur plusieurs IPs
- imite une attaque "aléatoire"
- impossible de bloquer chaque attempt

**legacy systems:**
- moins de logging
- monitoring réduit
- défenses anciennes

**6 semaines sans détection:**
- systèmes non-production = monitoring faible
- pas de corrélation entre alertes
- volume élevé = noyé dans le bruit

---

# partie 6 : ce que microsoft a révélé

## données compromises

**ce qui a été volé:**
- emails d'executives
- emails de cybersecurity team
- documents attachés aux emails
- secrets/credentials dans ces emails

**ce qui n'a pas été volé:**
- systèmes de production
- code source client
- ai systems
- customer-facing systems

## impact réel

**d'après microsoft:**
- "incident has not had a material impact on operations"
- "not reasonably likely to materially impact financial condition"

**en réalité:**
- executives emails compromises = risque énorme
- cyber team emails = informations de défense exposées
- réputation atteinte

---

# partie 7 : mitre att&ck mapping complet

## tactics utilisées

| tactic id | tactic name | techniques utilisées |
|-----------|-------------|----------------------|
| ta0006 | credential access | t1110.003 (password spray) |
| ta0001 | initial access | t1078 (valid accounts) |
| ta0003 | persistence | account takeover persistant |
| ta0008 | lateral movement | access à multiples email accounts |
| ta0009 | collection | t1114.002 (email collection) |
| ta0010 | exfiltration | encrypted exfil via http/https |

## techniques détaillées

**t1110.003 - credential access: password spray**
- attaque : testage massif de credentials
- cible : comptes non-production
- résultat : compromission réussie

**t1078 - initial access: valid accounts**
- attaque : utilisation du compte legacy compromis
- escalade : accès à d'autres comptes via permissions
- résultat : email account access

**t1114.002 - collection: remote email collection**
- attaque : exfiltration des emails
- méthode : via les permissions du compte
- cible : senior leadership emails

---

# partie 8 : enseignements defensifs

## ce que microsoft n'avait pas

- mfa sur comptes legacy
- monitoring des test tenants
- alertes sur password spray
- credential hygiene sur systèmes non-production

## ce que microsoft a mis en place après

**court terme:**
- password resets
- mfa sur tous les comptes
- monitoring accru

**long terme:**
- "apply current security standards to legacy systems"
- enhanced security controls
- cross-enterprise coordination

## leçons pour l'industrie

1. **systèmes legacy = risque énorme**
   - apparemment "non-critical" = attaque target
   - même risque que production

2. **password spray est difficile à détecter**
   - volume énorme = impossible à bloquer
   - doit être distribué
   - MFA est la seule vraie défense

3. **données internes = valeur énorme**
   - apt29 a ciblé microsoft pour... microsoft data
   - emails d'executives = or pour un attaquant
   - cyber team emails = information de défense

4. **nation-states ont ressources infinies**
   - 6 semaines de password spray = pas grave pour apt29
   - patient, persistent, well-resourced

---

# partie 9: pyramid of pain - application

## ce qu'apt29 a changé facilement

- **ioc faibles:**
  - domaines C2 (changé)
  - IPs (changé)
  - hashes malware (changé)

## ce qu'apt29 ne peut pas changer

- **ttp difficiles à changer:**
  - password spray comme technique initiale
  - exfiltration d'emails
  - targeting de comptes legacy
  - patient reconnaissance

**conclusion:** détecter sur les TTPs plutôt que les IoCs

---

# partie 10 : osint tools utilisées

pour identifier apt29, on peut utiliser:

**mitre att&ck:**
- search "midnight blizzard" sur le site
- find apt29 reference

**crowdstrike:**
- cozy bear = apt29 = midnight blizzard
- database des APTs

**mandiant/google:**
- apt29 profiles
- midnight blizzard background

**darkint/humint:**
- rapports gouvernementaux sur les APTs russes
- coordinations avec law enforcement

---

# résumé : la chaîne d'attaque

```
password spray attack (6 semaines)
    ↓
legacy test account compromise
    ↓
escalade vers email accounts
    ↓
exfiltration d'emails
    ↓
découverte de secrets
    ↓
tentative d'accès à source code
    ↓
password spray intensifiée (10x)
    ↓
microsoft détecte et bloque
```

**timeline totale:** novembre 2023 → mars 2024 (4 mois+)

---

**status:** ✓ incident compris et analysé

**ce que j'ai appris:** comment analyser une attaque nation-state réelle, mapper aux mitre, identifier les apt

**utilité:** incident response, threat hunting, defense planning

---

*une attaque de nation-state n'est pas spectaculaire. c'est patient, systematique, et resourced pour durer des mois.*
