# Flask Blog exercise : documentation de réalisation

**Exercice**: Building a Blog with Flask (Part 1)
**Module**: Web Security — Python Scripting for Security

---

## Objectif

Construire un blog minimaliste avec Flask pour comprendre la mécanique interne d'un site web dynamique: comment le serveur traite les requêtes (backend) et génère les pages (frontend).

En cybersécurité, comprendre le fonctionnement d'un backend est indispensable pour pouvoir l'auditer et l'attaquer.

---

## Concepts assimilés

| Concept | Ce que c'est |
|---------|-------------|
| **Virtual Environment** | Bulle isolée pour éviter de polluer le système global |
| **CLI / nano** | Éditer des fichiers en ligne de commande comme sur un serveur distant |
| **Routing** | Lier une URL précise (`/`) à une fonction Python |
| **Jinja2** | Moteur de templates: injecter des données Python dans du HTML statique |

---

## Structure finale du projet

```
flask_blog/
├── venv/                     ← Environnement virtuel (ne pas committer)
├── app.py                    ← Logique principale (routes, données)
├── static/
│   └── css/
│       └── style.css         ← Styles CSS
└── templates/
    ├── index.html            ← Page d'accueil (liste des articles)
    └── post.html             ← Page d'un article individuel
```

---

# Étape 1: Préparation de l'environnement

## Créer et isoler le projet

```bash
# Créer le dossier de travail
mkdir flask_blog
cd flask_blog

# Créer l'environnement virtuel
python -m venv venv

# Activer l'environnement virtuel
source venv/bin/activate
# Le prompt change: (venv) $

# Installer Flask dans la bulle isolée
pip install flask

# Créer la structure de dossiers
mkdir templates
mkdir -p static/css
```

**Pourquoi un environnement virtuel?**

Sans venv, Flask s'installe globalement sur le système. Si deux projets ont besoin de versions différentes de Flask, il y a conflit. Le venv crée une installation Python dédiée à ce seul projet.

---

# Étape 2: Création de app.py

Le fichier `app.py` est le cerveau de l'application. Il contient:
- L'initialisation de Flask
- La base de données fictive (liste Python)
- Les routes (qui répondent à quelle URL)

```bash
nano app.py
```

```python
from flask import Flask, render_template

app = Flask(__name__)

# Base de données simulée: une liste de dictionnaires Python
# Dans une vraie application, ce serait PostgreSQL ou MySQL
posts = [
    {
        'id': 1,
        'title': 'Introduction to Flask',
        'content': 'Flask is a micro web framework written in Python.'
    },
    {
        'id': 2,
        'title': 'Python for System Administrators',
        'content': 'Python is a powerful language for automation and system administration.'
    }
]

if __name__ == '__main__':
    app.run(debug=True)
```

**Points techniques**:

`app = Flask(__name__)`: Crée l'instance Flask. `__name__` indique à Flask où trouver les ressources (templates, static) relativement au fichier actuel.

`debug=True`: Le serveur redémarre automatiquement à chaque modification du code. Ne jamais activer en production.

---

# Étape 3: Création de la route d'accueil

On ajoute la première route dans `app.py`, entre la liste `posts` et le bloc `if __name__`:

```python
from flask import Flask, render_template

app = Flask(__name__)

posts = [
    {
        'id': 1,
        'title': 'Introduction to Flask',
        'content': 'Flask is a micro web framework written in Python.'
    },
    {
        'id': 2,
        'title': 'Python for System Administrators',
        'content': 'Python is a powerful language for automation and system administration.'
    }
]

# Route d'accueil
@app.route('/')
def index():
    return render_template('index.html', posts=posts)

if __name__ == '__main__':
    app.run(debug=True)
```

**Explication ligne par ligne**:

`@app.route('/')`: Un décorateur. Il "écoute" les requêtes HTTP GET sur l'URL `/`. Quand quelqu'un visite la racine du site, Flask exécute la fonction en dessous.

`def index()`: La fonction qui s'exécute. Elle peut s'appeler n'importe comment, mais `index` est la convention pour la page d'accueil.

`render_template('index.html', posts=posts)`: Charge le fichier `templates/index.html` et lui passe la variable `posts` (notre liste Python). Le template peut ensuite utiliser cette variable.

---

# Étape 4: Création du template index.html

Flask exige que les templates soient dans un dossier nommé **exactement** `templates`.

```bash
nano templates/index.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>Flask Blog</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <h1>Flask Blog</h1>

    {% for post in posts %}
        <div class="post-card">
            <h2>
                <a href="{{ url_for('post', post_id=post.id) }}">
                    {{ post.title }}
                </a>
            </h2>
            <p>{{ post.content[:100] }}...</p>
        </div>
    {% endfor %}

</body>
</html>
```

**La syntaxe Jinja2 expliquée**:

| Syntaxe | Rôle | Exemple |
|---------|------|---------|
| `{% %}` | Logique (action) | `{% for post in posts %}` |
| `{{ }}` | Affichage (écriture) | `{{ post.title }}` |
| `{# #}` | Commentaire (non visible) | `{# TODO: ajouter pagination #}` |

`{% for post in posts %}`: Boucle sur la liste `posts` transmise par Flask. Pour chaque article, le bloc HTML entre `{% for %}` et `{% endfor %}` est répété.

`{{ post.title }}`: Affiche la valeur de la clé `title` du dictionnaire `post` courant.

`{{ post.content[:100] }}`: Affiche seulement les 100 premiers caractères du contenu (slicing Python dans Jinja2).

`url_for('post', post_id=post.id)`: Génère dynamiquement l'URL vers la route nommée `post` avec le paramètre `post_id`. Plus robuste que d'écrire `/post/1` en dur: si on change l'URL, `url_for` s'adapte automatiquement.

---

# Étape 5: Création de la route article individuel

On ajoute une deuxième route dans `app.py`:

```python
from flask import Flask, render_template, abort

app = Flask(__name__)

posts = [
    {
        'id': 1,
        'title': 'Introduction to Flask',
        'content': 'Flask is a micro web framework written in Python.'
    },
    {
        'id': 2,
        'title': 'Python for System Administrators',
        'content': 'Python is a powerful language for automation and system administration.'
    }
]

@app.route('/')
def index():
    return render_template('index.html', posts=posts)

# Route dynamique: <int:post_id> capture un entier dans l'URL
@app.route('/post/<int:post_id>')
def post(post_id):
    # Chercher l'article avec l'id correspondant
    post = next((p for p in posts if p['id'] == post_id), None)
    
    # Si aucun article trouvé: retourner une erreur 404
    if post is None:
        abort(404)
    
    return render_template('post.html', post=post)

if __name__ == '__main__':
    app.run(debug=True)
```

**Explication**:

`/post/<int:post_id>`: La partie `<int:post_id>` est un **paramètre dynamique**. Flask capture automatiquement la valeur dans l'URL et la passe à la fonction. `int:` force la conversion en entier (si quelqu'un met `/post/abc`, Flask retourne 404 automatiquement).

`next((p for p in posts if p['id'] == post_id), None)`: Une generator expression qui cherche le premier article dont `id` correspond à `post_id`. Le deuxième argument `None` est retourné si aucun article n'est trouvé.

`abort(404)`: Retourne une réponse HTTP 404 (Not Found) si l'article n'existe pas. Importer `abort` depuis `flask`.

---

# Étape 6: Création du template post.html

```bash
nano templates/post.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ post.title }} - Flask Blog</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <header>
        <a href="{{ url_for('index') }}">← Retour au blog</a>
    </header>

    <article>
        <h1>{{ post.title }}</h1>
        <p>{{ post.content }}</p>
    </article>

</body>
</html>
```

`url_for('index')`: Génère l'URL vers la route `index` (la page d'accueil `/`).

---

# Bonus: Ajout du CSS

```bash
nano static/css/style.css
```

```css
body {
    font-family: Arial, sans-serif;
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
    color: #333;
}

h1 {
    color: #2c3e50;
    border-bottom: 2px solid #3498db;
    padding-bottom: 10px;
}

.post-card {
    background: white;
    padding: 20px;
    margin: 20px 0;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.post-card h2 a {
    color: #3498db;
    text-decoration: none;
}

.post-card h2 a:hover {
    text-decoration: underline;
}

article {
    background: white;
    padding: 30px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

header a {
    color: #3498db;
    text-decoration: none;
}
```

---

# Étape 7: Lancement du serveur

```bash
# S'assurer que le venv est actif
source venv/bin/activate

# Lancer le serveur de développement
python app.py
```

**Résultat attendu dans le terminal**:

```
 * Serving Flask app 'app'
 * Debug mode: on
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
```

Le site tourne sur le **port 5000** en localhost. Ouvrir `http://127.0.0.1:5000` dans le navigateur.

---

# Code final complet

## app.py

```python
from flask import Flask, render_template, abort

app = Flask(__name__)

posts = [
    {
        'id': 1,
        'title': 'Introduction to Flask',
        'content': 'Flask is a micro web framework written in Python.'
    },
    {
        'id': 2,
        'title': 'Python for System Administrators',
        'content': 'Python is a powerful language for automation and system administration.'
    }
]

@app.route('/')
def index():
    return render_template('index.html', posts=posts)

@app.route('/post/<int:post_id>')
def post(post_id):
    post = next((p for p in posts if p['id'] == post_id), None)
    if post is None:
        abort(404)
    return render_template('post.html', post=post)

if __name__ == '__main__':
    app.run(debug=True)
```

## templates/index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>Flask Blog</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <h1>Flask Blog</h1>

    {% for post in posts %}
        <div class="post-card">
            <h2>
                <a href="{{ url_for('post', post_id=post.id) }}">
                    {{ post.title }}
                </a>
            </h2>
            <p>{{ post.content[:100] }}...</p>
        </div>
    {% endfor %}
</body>
</html>
```

## templates/post.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ post.title }} - Flask Blog</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <header>
        <a href="{{ url_for('index') }}">← Retour au blog</a>
    </header>

    <article>
        <h1>{{ post.title }}</h1>
        <p>{{ post.content }}</p>
    </article>
</body>
</html>
```

---

# Flux de données: comment ça marche?

```
Visiteur tape: http://127.0.0.1:5000/
        ↓
Flask reçoit la requête GET /
        ↓
@app.route('/') → Exécute index()
        ↓
index() appelle render_template('index.html', posts=posts)
        ↓
Jinja2 charge templates/index.html
Jinja2 remplace {% for %} et {{ }} avec les vraies données
        ↓
Flask retourne le HTML généré au navigateur
        ↓
Navigateur affiche la page

─────────────────────────────────────────────────

Visiteur clique sur un article → /post/1
        ↓
Flask reçoit la requête GET /post/1
        ↓
@app.route('/post/<int:post_id>') → Exécute post(post_id=1)
        ↓
Cherche l'article avec id=1 dans la liste posts
        ↓
Appelle render_template('post.html', post=article_trouvé)
        ↓
Jinja2 génère le HTML avec les données de l'article
        ↓
Navigateur affiche l'article
```

---

# Points de sécurité à retenir

**debug=True en développement seulement**

Le mode debug expose un terminal Python interactif dans le navigateur en cas d'erreur. En production, un attaquant pourrait exécuter du code arbitraire sur le serveur. Toujours désactiver en production.

```python
# Développement
app.run(debug=True)

# Production (utiliser Gunicorn ou uWSGI à la place)
app.run(debug=False)
```

**Validation des entrées utilisateur**

Notre route `/post/<int:post_id>` accepte seulement des entiers. Flask rejette automatiquement `/post/abc`. C'est une première couche de validation.

**Pas de données sensibles dans les templates**

Les templates Jinja2 ont accès à toutes les variables transmises. Ne jamais passer des objets contenant des mots de passe ou des tokens à un template.

**Échappement HTML automatique**

Jinja2 échappe automatiquement les caractères spéciaux (`<`, `>`, `"`, `&`) dans `{{ }}`. Si un post contient `<script>alert('XSS')</script>`, il sera affiché comme texte et non exécuté. Protection XSS de base intégrée.

---

# Résumé des étapes

| Étape | Action | Commande/Fichier |
|-------|--------|-----------------|
| 1 | Créer l'environnement | `mkdir + python -m venv + pip install flask` |
| 2 | Créer app.py | `nano app.py` |
| 3 | Ajouter route `/` | `@app.route('/') def index()` |
| 4 | Créer index.html | `nano templates/index.html` avec Jinja2 |
| 5 | Ajouter route `/post/<id>` | `@app.route('/post/<int:post_id>')` |
| 6 | Créer post.html | `nano templates/post.html` |
| 7 | Ajouter CSS | `nano static/css/style.css` |
| 8 | Lancer le serveur | `python app.py` |

---

**Exercice complété ✓**

Prochaine étape: Ajouter la persistance avec une vraie base de données (Flask-SQLAlchemy + SQLite/PostgreSQL).
