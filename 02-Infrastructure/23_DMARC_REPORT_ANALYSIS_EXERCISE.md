# DMARC report analysis exercise — email authentification et deliverability

**Exercice**: Analyse d'un rapport XML DMARC  
**Contexte**: Marketing team investigating email deliverability  
**Organisation**: Nuvotec.io (domaine analysé)  
**Durée**: ~45 minutes  
**Objectif**: Comprendre comment lire et interpréter les rapports DMARC

---

## Contexte de l'exercice

Une équipe marketing observe que ses campagnes email ont un taux de livraison très bas par rapport aux standards du marché. Après avoir implémenté SPF, DKIM et DMARC sur leur domaine, l'équipe reçoit des rapports DMARC mais ne sait pas comment les lire.

**Mission**: Analyser le rapport XML DMARC fourni par Microsoft (Enterprise Outlook) et répondre à des questions pour comprendre les problèmes de deliverability.

---

# Understanding DMARC Reports

## Qu'est-ce qu'un rapport DMARC?

Un rapport DMARC est un document XML généré par les serveurs de messagerie (Gmail, Outlook, Yahoo, etc.) qui reçoivent vos emails. Il documente:
- Qui a reçu vos emails
- D'où ils venaient vraiment (source IP)
- Si SPF et DKIM ont passé
- Si l'email s'aligne avec votre politique DMARC
- Quelle action a été prise (livré, quarantaine, rejeté)

**Importance**: C'est le seul feedback que vous avez sur l'authentification réelle de vos emails auprès des principaux fournisseurs.

---

# Analyse détaillée du rapport

## Structure XML d'un rapport DMARC

Un rapport DMARC contient trois sections principales:

```
<?xml version="1.0"?>
<feedback>
  ├─ <report_metadata>        # Infos sur le rapport
  ├─ <policy_published>       # Politique DMARC du domaine
  └─ <record>                 # Enregistrement d'email (peut y en avoir plusieurs)
      ├─ <row>                # Métadonnées du record
      ├─ <identifiers>        # Identifiants de l'email
      └─ <auth_results>       # Résultats SPF/DKIM
```

---

# Questions et réponses

## Q1: Who is the organization that generated and sent the DMARC report?

**Réponse**: `Enterprise Outlook`

**Explication**: 
- Cette information se trouve dans `<report_metadata>` → `<org_name>`
- Enterprise Outlook est l'infrastructure de Microsoft
- Microsoft a intercepté les emails et généré ce rapport pour avertir le propriétaire du domaine nuvotec.io
- C'est un signal important: vos emails passent par les serveurs Microsoft

**Implication**: Microsoft valide ou rejette vos emails en fonction de votre configuration SPF/DKIM/DMARC.

---

## Q2: What is the email address of the report sender?

**Réponse**: `dmarcreport@microsoft.com`

**Explication**:
- Location: `<report_metadata>` → `<email>`
- Cette adresse envoie les rapports DMARC régulièrement
- Vous pouvez recevoir des centaines de ces rapports par jour si vous avez du volume

**Action recommandée**: Configurer une adresse email dédiée pour recevoir ces rapports (ex: `dmarc-reports@nuvotec.io`).

---

## Q3: What is the domain this report is about?

**Réponse**: `nuvotec.io`

**Explication**:
- Location: `<policy_published>` → `<domain>`
- C'est le domaine de l'organisation marketing qu'on analyse
- C'est le domaine que nuvotec.io a configuré avec SPF, DKIM et DMARC

---

## Q4: What is the total number of email records included in this DMARC report?

**Réponse**: `4`

**Explication**:
- Comptez simplement les blocs `<record>` distincts dans le XML
- Chaque `<record>` représente un groupe d'emails avec les mêmes caractéristiques
- Le rapport agrège les données par: source IP, domaine en-tête, résultats SPF/DKIM

**Analyse**:
- Record 1: 1 email depuis Google vers linwoodgroup.com
- Record 2: 1 email depuis HubEngage vers oska.tech
- Record 3: 1 email depuis Google vers quantaxa.net
- Record 4: 1 email depuis Google vers quantaxa.net

Total: 4 emails analysés dans ce rapport.

---

## Q5: What is the DKIM alignment mode for nuvotec.io?

**Réponse**: `r` (Relaxed)

**Explication**:
- Location: `<policy_published>` → `<adkim>`
- Les deux modes possibles sont:
  - `r` = Relaxed mode (moins strict)
  - `s` = Strict mode (plus strict)

**Qu'est-ce que cela signifie?**

En **Relaxed mode**:
- Le domaine DKIM de la signature peut être un sous-domaine du domaine principal
- Exemple: Email signé par `mailer.nuvotec.io` passe avec `nuvotec.io` en Relaxed

En **Strict mode**:
- Le domaine DKIM doit correspondre exactement
- Exemple: Email signé par `mailer.nuvotec.io` échouerait contre `nuvotec.io` en Strict

**Configuration actuelle**: Nuvotec utilise Relaxed, ce qui est standard et permissif.

---

## Q6: Which domain was used as envelope_from in the email sent to oska.tech?

**Réponse**: `mailer.hubengage.net`

**Explication**:
- Location: Deuxième `<record>` (celui avec `<envelope_to>oska.tech</envelope_to>`)
- Dans `<identifiers>` → `<envelope_from>`
- `envelope_from` est l'adresse SMTP (Return-Path), pas l'adresse visible

**Pourquoi c'est important?**

Nuvotec n'envoie pas directement ses emails. Ils utilisent un prestataire externe: **HubEngage**. L'enveloppe technique provient de `mailer.hubengage.net` alors que l'email affiche `From: nuvotec.io`.

Cela explique pourquoi il y a plusieurs domaines dans le rapport:
- `envelope_from`: mailer.hubengage.net (serveur réel d'envoi)
- `header_from`: nuvotec.io (ce que voit le destinataire)

---

## Q7: What is the DKIM selector used by nuvotec.io for Google-based signing?

**Réponse**: `20230601`

**Explication**:
- Location: Troisième `<record>` (celui avec l'IP Google `2a00:1450:4864:20::34a`)
- Dans `<auth_results>` → `<dkim>` où `<domain>google.com`
- Le champ `<selector>` contient la clé

**Qu'est-ce qu'un sélecteur DKIM?**

Le sélecteur est une partie de la clé publique stockée dans le DNS:
```
selector1._domainkey.example.com  TXT  v=DKIM1; p=<clé-publique>
```

Dans ce cas:
```
20230601._domainkey.nuvotec.io  TXT  v=DKIM1; p=<clé-publique>
```

**Signification**: Nuvotec utilise un sélecteur basé sur la date (`20230601`), probablement pour rotationner les clés régulièrement (bonne pratique de sécurité).

---

## Q8: Which domain passed SPF but failed DKIM in the report?

**Réponse**: `nuvotec.io`

**Explication**:
- Regardez la colonne `<policy_evaluated>` pour tous les records
- Trois records affichent `<dkim>fail</dkim>` et `<spf>pass</spf>`:
  - Record 1: IP Google, SPF pass, DKIM fail
  - Record 3: IP Google, SPF pass, DKIM fail
  - Record 4: IP Google, SPF pass, DKIM fail

- Dans `<identifiers>` → `<header_from>`, le domaine est toujours `nuvotec.io`

**Analyse du problème**:

SPF fonctionne parce que:
- Google est listé dans le SPF de nuvotec.io
- Google peut envoyer des emails au nom de nuvotec.io

DKIM échoue pour certains records parce que:
- La signature DKIM ne correspond pas
- Possible raison: Google signe avec un sélecteur différent

---

## Q9: In the record where envelope_to is linwoodgroup.com, which mechanism caused DMARC to pass?

**Réponse**: `spf`

**Explication**:
- Location: Premier `<record>` (celui avec `<envelope_to>linwoodgroup.com</envelope_to>`)
- Dans `<policy_evaluated>`:
  - `<dkim>fail</dkim>`
  - `<spf>pass</spf>`
  - `<disposition>none</disposition>` (email livré)

**Important**: Pour que DMARC passe, il suffit qu'**UN SEUL** des deux mécanismes soit valide:
- SPF OU DKIM (pas besoin des deux)

Dans ce cas:
- DKIM a échoué
- SPF a réussi
- Email a été livré normalement

**Logic DMARC**:
```
Si (SPF valide OU DKIM valide) {
    appliquer disposition "none" (livrer l'email)
}
```

---

## Q10: What is the policy disposition applied to all records in the report?

**Réponse**: `none`

**Explication**:
- Location: Chaque `<record>` → `<row>` → `<policy_evaluated>` → `<disposition>`
- Tous les records ont `<disposition>none</disposition>`

**Qu'est-ce que cela signifie?**

La disposition peut être:
- `none`: Aucune action, email livré normalement
- `quarantine`: Email mis en quarantaine (dossier spam/suspicious)
- `reject`: Email rejeté (non livré)

Ici: **Tous les emails ont été livrés normalement** (`none`).

**Comparaison avec la politique publiée**:
```xml
<policy_published>
  <p>quarantine</p>  ← Politique: quarantine
  ...
</policy_published>
```

Même si la politique DMARC publiée était `quarantine`, la disposition appliquée est `none` parce que:
- Au moins un mécanisme (SPF ou DKIM) a réussi
- Les emails sont authentifiés
- Aucune action restrictive n'a été nécessaire

---

# Analyse globale du rapport

## Résumé des résultats

| Record | Destinataire | Source IP | SPF | DKIM | Disposition | Problème |
|--------|--------------|-----------|-----|------|-------------|---------|
| 1 | linwoodgroup.com | Google | ✓ | ✗ | none | DKIM échoue |
| 2 | oska.tech | HubEngage | ✗ | ✓ | none | SPF échoue |
| 3 | quantaxa.net | Google | ✓ | ✗ | none | DKIM échoue |
| 4 | quantaxa.net | Google | ✓ | ✗ | none | DKIM échoue |

## Problèmes identifiés

### Problème 1: DKIM échoue pour Google (3 records sur 4)

Les emails signés par Google échouent la validation DKIM. Raisons possibles:
- Le sélecteur DKIM de Google ne correspond pas à ce qui est en DNS
- La clé publique dans le DNS est incorrecte ou expirée
- Configuration d'alignement relâchée accepte les erreurs

**Impact sur deliverability**: Moyen (SPF passe, donc emails sont livrés)

### Problème 2: SPF échoue pour HubEngage (1 record)

Email envoyé depuis `mailer.hubengage.net` mais SPF n'est pas passé. Raison:
- HubEngage n'est peut-être pas inclus dans le SPF de nuvotec.io
- Ou HubEngage n'est pas configuré correctement

**Impact sur deliverability**: Critique (si DKIM échoue aussi, email serait rejeté)

**Solution**: Ajouter HubEngage dans le SPF:
```
v=spf1 include:hubsengage.com -all
```

---

# Recommandations pour améliorer la deliverability

## Court terme

1. **Corriger le SPF pour HubEngage**
   - Vérifier que l'authentification de HubEngage est complète
   - S'assurer que `mailer.hubengage.net` est dans le SPF

2. **Vérifier les sélecteurs DKIM**
   - S'assurer que les sélecteurs DKIM publiés en DNS correspondent à ceux utilisés réellement
   - Pour Google: `20230601._domainkey.nuvotec.io` doit exister en DNS

3. **Monitorer les rapports DMARC**
   - Mettre en place une pipeline automatisée pour traiter les rapports
   - Définir des alertes si le taux d'échec dépasse un seuil

## Long terme

1. **Rationaliser les serveurs d'envoi**
   - Nuvotec utilise plusieurs prestataires (Google, HubEngage)
   - Considérer la consolidation pour simplifier la configuration

2. **Passer en Strict alignment**
   - Actuellement: Relaxed (`r`)
   - Futur: Strict (`s`) pour plus de sécurité
   - Demande une configuration plus rigoureuse mais meilleure défense contre le spoofing

3. **Implémenter le monitoring DMARC**
   - Utiliser des outils comme dmarcian, Valimail ou Google Postmaster Tools
   - Automatiser la génération de rapports et d'alertes

---

# Points clés à retenir

## Pour comprendre les rapports DMARC

1. **Source XML**: Les rapports sont générés par les fournisseurs email (Microsoft, Google, Yahoo)
2. **Granularité**: Chaque `<record>` est une agrégation d'emails similaires
3. **Deux mécanismes**: SPF et DKIM (il suffit qu'un seul passe)
4. **Disposition vs Politique**: La politique est ce que vous publiez; la disposition est ce qui a été appliqué

## Pour améliorer la deliverability

1. **SPF**: Maintenir une liste à jour des serveurs d'envoi autorisés
2. **DKIM**: Publier les sélecteurs corrects en DNS et les rotationner régulièrement
3. **DMARC**: Commencer en `p=none`, passer à `p=quarantine`, puis `p=reject`
4. **Monitoring**: Analyser régulièrement les rapports DMARC pour identifier les problèmes

---

**Status**: ✓ Exercice compris et analysé

**Ce que j'ai appris**: Comment lire et interpréter un rapport DMARC XML, identifier les problèmes d'authentification email, recommander des solutions

**Utilité**: 5/5

**Application**: Diagnostiquer les problèmes de deliverability, configurer SPF/DKIM/DMARC correctement, améliorer la réputation du domaine
