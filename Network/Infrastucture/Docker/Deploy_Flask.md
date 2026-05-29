# Déploiement d'une application Flask avec Docker, Nginx et PostgreSQL

## Contexte

Dans le cadre de ma formation en cybersécurité (Jedha Bootcamp, module Infrastructure Security), j'ai déployé une application web Python existante en utilisant Docker. L'objectif était de concevoir une architecture de déploiement portable, reproductible, et sécurisée, en appliquant les bonnes pratiques de production : HTTPS, headers de sécurité, isolation réseau.

## Résultat

L'application Todo (liste de tâches) tourne en local avec 3 services Docker orchestrés :

```
Client (navigateur)
    ↓ HTTPS (port 443)
Nginx (reverse proxy)
    ↓ HTTP interne (port 5000)
Flask + Gunicorn
    ↓ TCP (port 5432)
PostgreSQL
```

---

## Stack technique

| Composant | Rôle |
|-----------|------|
| Python 3.11 + Flask | Framework web de l'application |
| Gunicorn | Serveur WSGI pour exécuter Flask en production |
| PostgreSQL 15 | Base de données relationnelle |
| Nginx | Reverse proxy, HTTPS, headers de sécurité |
| Docker + Docker Compose | Conteneurisation et orchestration |
| mkcert | Génération de certificats SSL de confiance en local |

---

## Structure du projet

```
todo-app/
├── app/
│   ├── __init__.py          # Création de l'app Flask (create_app())
│   ├── models.py            # Modèle SQLAlchemy (table Todo)
│   ├── routes.py            # Endpoints (/, /add, /delete, /complete)
│   ├── templates/
│   │   ├── base.html
│   │   ├── index.html
│   │   └── error.html
│   └── static/
│       ├── css/style.css
│       └── js/app.js
├── nginx/
│   ├── nginx.conf           # Configuration du reverse proxy
│   └── certs/
│       ├── cert.pem         # Certificat SSL (généré via mkcert)
│       └── key.pem          # Clé privée SSL
├── Dockerfile               # Recette de construction de l'image Flask
├── docker-compose.yml       # Orchestration des 3 services
├── requirements.txt         # Dépendances Python
└── run.py                   # Point d'entrée Flask
```

---

## Étapes de réalisation

### Étape 1 : Analyse du code existant

Avant d'écrire quoi que ce soit, j'ai lu le code de l'application pour comprendre ce qu'elle attend :

```bash
cat run.py
cat app/__init__.py
cat requirements.txt
```

Ce que j'ai identifié :

* Le point d'entrée est `run.py`, qui appelle `create_app()` depuis `app/__init__.py`
* L'app utilise Flask + SQLAlchemy pour la DB
* La connexion à PostgreSQL se fait via la variable d'environnement `DATABASE_URL`
* Le port par défaut de Flask est `5000` (aucun port explicite dans `app.run()` = valeur par défaut Flask)
* `psycopg2-binary` est déjà dans les dépendances (driver PostgreSQL)

**Comment j'ai su quel port utiliser :** `run.py` appelle `app.run(host="0.0.0.0", debug=True)` sans préciser de port. La documentation Flask indique que le port par défaut est `5000`.

---

### Étape 2 : Ajout de Gunicorn au requirements.txt

Le `requirements.txt` d'origine ne contient pas Gunicorn :

```
Flask==2.3.3
Flask-SQLAlchemy==3.1.1
psycopg2-binary==2.9.7
python-dotenv==1.0.0
```

Je l'ai ajouté :

```bash
echo "gunicorn==21.2.0" >> requirements.txt
```

**Pourquoi Gunicorn :** `flask run` est un serveur de développement mono-thread, incapable de gérer plusieurs requêtes simultanées. Gunicorn est un serveur WSGI de production qui lance plusieurs workers (processus) pour absorber la charge. La version `21.2.0` est précisée dans les notes techniques du projet.

---

### Étape 3 : Construction du Dockerfile

Le Dockerfile est la recette qui dit à Docker comment construire l'image de l'application.

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ ./app/
COPY run.py .

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "run:app"]
```

**Explication de chaque ligne :**

`FROM python:3.11-slim` : image de base. Le suffixe `slim` signifie "version minimale", sans outils de développement, donc plus légère et moins exposée.

`WORKDIR /app` : définit `/app` comme répertoire de travail dans le container. Tous les chemins relatifs seront résolus par rapport à ce dossier.

`RUN apt-get install gcc libpq-dev` : `psycopg2` (le driver PostgreSQL pour Python) a besoin de `gcc` (compilateur C) et `libpq-dev` (headers de la librairie PostgreSQL) pour fonctionner. Sans ces deux paquets, pip plante à l'installation. Le `rm -rf /var/lib/apt/lists/*` nettoie le cache apt pour alléger l'image finale.

`COPY requirements.txt .` puis `RUN pip install` : on copie les dépendances EN PREMIER, avant le code. Si on modifie uniquement le code applicatif, Docker réutilise la couche pip depuis son cache (pas besoin de réinstaller tous les packages). C'est une optimisation du système de couches (layers) de Docker.

`COPY app/ ./app/` et `COPY run.py .` : on copie uniquement ce qui est nécessaire à l'exécution (pas les fichiers de config Docker, pas le dossier nginx, etc.).

`CMD ["gunicorn", "--bind", "0.0.0.0:5000", "run:app"]` : commande lancée au démarrage du container. `run:app` signifie "fichier `run.py`, variable `app`", ce qui correspond exactement à la structure du projet.

---

### Étape 4 : Génération des certificats SSL avec mkcert

Pour activer HTTPS en local, j'ai utilisé `mkcert`, l'outil recommandé dans le cours. Contrairement à `openssl` qui génère des certificats auto-signés non reconnus par les navigateurs, `mkcert` installe une autorité de certification (CA) locale et émet des certificats automatiquement approuvés sur la machine.

```bash
# Installation
brew install mkcert

# Création de la CA locale et ajout dans le trust store système
mkcert -install

# Génération du certificat pour localhost
mkdir -p nginx/certs
cd nginx/certs
mkcert -key-file key.pem -cert-file cert.pem localhost
cd ../..
```

**Résultat :** deux fichiers dans `nginx/certs/` :

* `cert.pem` : certificat public (présenté au navigateur)
* `key.pem` : clé privée (utilisée par Nginx pour le chiffrement)

**Pourquoi mkcert plutôt qu'openssl :** le cours (module "Setup HTTPS") explique que mkcert est l'outil adapté au développement local car il évite les avertissements "certificat non approuvé" en s'intégrant au trust store du système.

---

### Étape 5 : Configuration Nginx

Nginx joue ici le rôle de reverse proxy sécurisé. Il est le seul point d'entrée exposé vers l'extérieur. Flask ne reçoit que du trafic interne provenant de Nginx.

```nginx
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    server_tokens off;
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;

    server {
        listen 80;
        server_name localhost;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        server_name localhost;

        ssl_certificate /etc/nginx/certs/cert.pem;
        ssl_certificate_key /etc/nginx/certs/key.pem;
        ssl_protocols TLSv1.2 TLSv1.3;

        location / {
            proxy_pass http://web:5000;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

**Explication des choix de sécurité :**

`server_tokens off` : masque la version de Nginx dans les réponses HTTP. Un attaquant ne peut pas cibler une version spécifique avec une CVE connue.

`Strict-Transport-Security` (HSTS) : force le navigateur à utiliser HTTPS pendant 1 an. Même si un utilisateur tape `http://`, il est automatiquement redirigé.

`X-Content-Type-Options: nosniff` : empêche le navigateur de "deviner" le type MIME d'un fichier. Protection contre certaines attaques XSS.

`X-Frame-Options: DENY` : empêche l'intégration de la page dans une iframe. Protection contre le clickjacking.

`return 301 https://` : redirection permanente de HTTP vers HTTPS. Tout trafic non chiffré est rejeté.

`ssl_protocols TLSv1.2 TLSv1.3` : uniquement les versions modernes de TLS. TLS 1.0 et 1.1 sont obsolètes et vulnérables.

`proxy_set_header X-Forwarded-Proto $scheme` : indique à Flask qu'il est derrière HTTPS. Sans ce header, Flask génère des URLs en `http://` au lieu de `https://`.

---

### Étape 6 : Orchestration avec Docker Compose

Docker Compose définit les 3 services, leurs relations, les réseaux d'isolement et les volumes de persistance.

```yaml
services:

  db:
    image: postgres:15
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: todo_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  web:
    build: .
    restart: unless-stopped
    environment:
      FLASK_APP: run.py
      FLASK_ENV: development
      SECRET_KEY: dev_key_todo_app_2025
      DATABASE_URL: postgresql://postgres:postgres@db:5432/todo_db
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend
      - frontend

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
    depends_on:
      - web
    networks:
      - frontend

volumes:
  postgres_data:

networks:
  backend:
  frontend:
```

**Les points clés de cette configuration :**

`healthcheck` sur db : Docker Compose attend que PostgreSQL réponde réellement avant de démarrer Flask. Sans ça, Flask démarre avant que la DB soit prête, et la connexion échoue.

`depends_on: condition: service_healthy` : Flask ne démarre qu'une fois que le healthcheck de PostgreSQL est passé.

`DATABASE_URL: postgresql://postgres:postgres@db:5432/todo_db` : l'hôte est `db`, pas `localhost`. Dans Docker, chaque service est joignable via son nom de service grâce à la résolution DNS interne.

`networks: backend / frontend` : architecture en deux niveaux réseau. PostgreSQL est uniquement sur `backend`, accessible seulement par Flask. Nginx est uniquement sur `frontend`, accessible seulement par Flask. Nginx ne peut pas atteindre PostgreSQL directement. Si Nginx est compromis, l'attaquant ne peut pas accéder à la base de données.

`volumes: ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro` : le fichier de configuration local est monté dans le container en lecture seule (`:ro`). Nginx lit la config depuis le disque de la machine hôte, ce qui permet de modifier la config sans reconstruire l'image.

`postgres_data` : volume nommé qui persiste les données PostgreSQL. Les données survivent aux arrêts/redémarrages des containers.

---

### Étape 7 : Déploiement

```bash
docker compose up --build -d
```

`--build` force la reconstruction de l'image Flask (prend en compte les changements du Dockerfile ou du code).

`-d` lance les containers en arrière-plan (detached).

**Vérification :**

```bash
docker compose ps
```

```
NAME               IMAGE          STATUS
todo-app-db-1      postgres:15    Up (healthy)
todo-app-nginx-1   nginx:alpine   Up
todo-app-web-1     todo-app-web   Up
```

**Test :**

```bash
curl -k https://localhost
```

---

## Problèmes rencontrés et solutions

### nginx.conf créé comme dossier

**Symptôme :** `zsh: is a directory: nginx/nginx.conf`

**Cause :** la commande `mkdir -p nginx/nginx.conf` avait créé un dossier plutôt qu'un fichier.

**Diagnostic :** `ls -la nginx/` a montré que `nginx.conf` avait les permissions d'un dossier (`drwxr-xr-x`).

**Solution :** `rm -rf nginx/nginx.conf` puis recréation avec `cat > nginx/nginx.conf`.

---

### Nginx en état `Restarting` avec "unexpected }"

**Symptôme :** `nginx: [emerg] unexpected "}" in /etc/nginx/nginx.conf:20`

**Cause :** des backslashes parasites (`\;`) s'étaient glissés dans le fichier lors de la création avec heredoc (problème d'échappement zsh).

**Diagnostic :** `docker compose logs nginx | grep emerg` + `cat nginx/nginx.conf` ont confirmé les caractères incorrects.

**Solution :**
```bash
sed -i '' 's|request_uri\\;|request_uri;|g' nginx/nginx.conf
sed -i '' 's|web:5000\\;|web:5000;|g' nginx/nginx.conf
```

---

### "worker_processes directive is not allowed here"

**Symptôme :** `"worker_processes" directive is not allowed here in /etc/nginx/conf.d/default.conf:1`

**Cause :** le `docker-compose.yml` montait le fichier vers `/etc/nginx/conf.d/default.conf`. Ce dossier est réservé aux blocs `server {}`. Les directives globales comme `worker_processes` doivent être dans `/etc/nginx/nginx.conf`.

**Solution :** corriger le chemin de montage dans `docker-compose.yml`.

---

## Commandes utiles

```bash
# Lancer la stack
docker compose up --build -d

# Voir l'état des services
docker compose ps

# Voir les logs d'un service
docker compose logs -f nginx
docker compose logs -f web

# Arrêter la stack (conserve les données)
docker compose down

# Arrêter et supprimer les données
docker compose down -v

# Accéder au shell d'un container
docker compose exec web bash
docker compose exec db psql -U postgres -d todo_db
```

---

## Architecture de sécurité

```
Internet
    |
    | 443 (HTTPS / TLS 1.2+)
    ↓
+-------------------+
|      Nginx        |   réseau: frontend
|  reverse proxy    |   headers: HSTS, X-Frame, CSP
+-------------------+
    |
    | 5000 (HTTP interne)
    ↓
+-------------------+
|  Flask + Gunicorn |   réseaux: frontend + backend
|  application      |
+-------------------+
    |
    | 5432 (TCP interne)
    ↓
+-------------------+
|   PostgreSQL 15   |   réseau: backend uniquement
|   base de données |
+-------------------+
```

PostgreSQL n'est jamais exposé vers l'extérieur. Nginx ne peut pas accéder à la base de données. Flask est le seul composant qui communique avec les deux niveaux.

---

## Ressources utilisées

* Documentation Docker Compose : https://docs.docker.com/compose/
* Documentation Nginx : https://nginx.org/en/docs/
* mkcert : https://github.com/FiloSottile/morcilla/mkcert
* OWASP Secure Headers : https://owasp.org/www-project-secure-headers/
* Cours Jedha Bootcamp : modules "Docker Web Development", "Setup HTTPS", "Infrastructure Security"
