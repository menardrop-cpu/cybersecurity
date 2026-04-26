# HTML : Construire son premier site web

> Suite de mes notes HTML. Ce cours va plus loin que les bases : on passe à la construction réelle d'une page, de l'esquisse sur papier jusqu'au formulaire final. C'est le workflow concret d'un dev front, et c'est aussi la structure qu'un analyste sécu va retrouver sur 99% des sites qu'il audite.

## Sommaire

1. [Wireframe : esquisser avant de coder](#1-wireframe--esquisser-avant-de-coder)
2. [Le tag `<head>` en détail](#2-le-tag-head-en-détail)
3. [La navigation](#3-la-navigation)
4. [Le header](#4-le-header)
5. [Les cards](#5-les-cards)
6. [Les formulaires](#6-les-formulaires)
7. [Page complète : exemple récapitulatif](#7-page-complète--exemple-récapitulatif)

---

## 1. Wireframe : esquisser avant de coder

### C'est quoi un wireframe

Un **wireframe** c'est une maquette basse fidélité d'une page web. Pas de couleurs, pas de polices, pas de design. Juste des blocs, des lignes, des rectangles qui représentent la structure de la page.

C'est l'équivalent du plan d'architecte avant de construire. Je dessine où vont être les éléments, leur taille relative, leur ordre, avant d'écrire une seule ligne de code. Ça évite de coder dans le vide et de tout refaire trois fois.

Un wireframe peut être :
* Dessiné à la main sur du papier (le plus rapide)
* Fait sur un outil comme Figma, Balsamiq, ou même draw.io

### Ce qu'un wireframe représente

Un wireframe découpe une page en **zones fonctionnelles** :

```
+------------------------------------------+
|              NAVIGATION                  |
+------------------------------------------+
|                                          |
|               HEADER                    |
|         (titre + bouton CTA)            |
|                                          |
+----------+----------+--------------------+
|          |          |                    |
|  CARD 1  |  CARD 2  |      CARD 3       |
|          |          |                    |
+----------+----------+--------------------+
|                                          |
|              FORMULAIRE                  |
|                                          |
+------------------------------------------+
|               FOOTER                    |
+------------------------------------------+
```

Les zones typiques d'un site :
* **Navigation** : menu en haut, liens vers les sections
* **Header** : zone d'accroche principale (hero section)
* **Contenu** : cards, articles, sections...
* **Formulaire** : contact, login, inscription...
* **Footer** : informations secondaires, liens légaux...

### Wireframer avant de coder : pourquoi c'est important

Sans wireframe, je vais coder la tête dans le guidon et rater la structure globale. Avec un wireframe, même grossier, j'ai une carte de ce que je construis. Chaque zone du wireframe va correspondre à un bloc HTML.

**Note sécu** : comprendre la structure d'une page par zones, c'est aussi ce que je fais quand j'analyse une cible. Je repère la nav, les formulaires, les endpoints. Le wireframe m'entraîne à penser en blocs fonctionnels, pas en pixels.

---

## 2. Le tag `<head>` en détail

Le `<head>` ne s'affiche pas à l'écran mais il pilote beaucoup de choses. C'est là que le navigateur reçoit ses instructions de configuration.

### Structure complète d'un `<head>` bien écrit

```html
<head>
  <!-- Encodage : toujours en premier -->
  <meta charset="UTF-8">

  <!-- Comportement sur mobile -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- Description pour les moteurs de recherche -->
  <meta name="description" content="Mon site de cybersécurité personnel">

  <!-- Auteur -->
  <meta name="author" content="Pierre">

  <!-- Titre de l'onglet -->
  <title>Pierre | Cybersécurité</title>

  <!-- Feuille de style CSS externe -->
  <link rel="stylesheet" href="style.css">

  <!-- Favicon (icône dans l'onglet) -->
  <link rel="icon" type="image/png" href="favicon.png">
</head>
```

### Détail de chaque élément

**`<meta charset="UTF-8">`**

Définit l'encodage du document. UTF-8 supporte tous les caractères : accents français, caractères arabes, emojis... Toujours en premier dans le `<head>`.

**`<meta name="viewport" content="width=device-width, initial-scale=1.0">`**

Sans ça, les mobiles affichent la page comme si elle était sur un écran desktop et la zoom out. Avec ça, la page s'adapte à la largeur de l'écran. Indispensable dès qu'on veut un site responsive.

**`<meta name="description" content="...">`**

Le texte qui apparaît sous le titre dans les résultats Google. N'affecte pas le rendu de la page mais important pour le référencement.

**`<title>`**

Le texte dans l'onglet du navigateur. C'est aussi le titre qui apparaît dans les résultats de recherche. Doit être unique par page et descriptif.

**`<link rel="stylesheet" href="style.css">`**

Lie une feuille de style CSS externe. Le navigateur va chercher le fichier `style.css` et appliquer ses règles à la page. Sans ça, tout s'affiche sans mise en forme.

**Note sécu** : le `<head>` est souvent rempli de scripts tiers (analytics, publicité, trackers). Quand j'audite une page, je lis systématiquement le `<head>` pour identifier toutes les ressources externes chargées. Chacune est une dépendance potentiellement vulnérable.

---

## 3. La navigation

La navigation c'est le menu qui permet de se déplacer entre les sections ou les pages du site. Elle est quasi systématiquement en haut de page.

### La structure sémantique : `<nav>`

HTML5 a introduit des balises sémantiques qui donnent du sens à la structure. Pour la navigation, c'est `<nav>`.

```html
<nav>
  <!-- Le menu va ici -->
</nav>
```

`<nav>` indique au navigateur (et aux outils d'accessibilité) que ce bloc est une zone de navigation. C'est mieux que d'utiliser une `<div>` générique.

### Les `<div>` : conteneurs génériques

Avant d'aller plus loin, je dois maîtriser les `<div>`.

Un `<div>` (*division*) est un **conteneur bloc générique** sans signification sémantique. Il sert à regrouper des éléments pour les styler ensemble ou les organiser.

```html
<div class="container">
  <div class="logo">Mon Logo</div>
  <div class="menu">...</div>
</div>
```

Les `<div>` sont invisibles par défaut. Ils n'ajoutent aucun style. Ils prennent toute la largeur disponible et se placent les uns sous les autres (comportement de bloc).

L'attribut `class` leur donne un nom pour les cibler en CSS :
```html
<div class="carte">Contenu d'une carte</div>
<div class="carte">Contenu d'une autre carte</div>
```

L'attribut `id` donne un identifiant **unique** à un élément :
```html
<div id="header-principal">...</div>
```

Règle : `class` pour cibler plusieurs éléments du même type, `id` pour un élément unique sur la page.

### Les listes dans la navigation

Le menu d'un site est conventionnellement construit avec une liste non ordonnée `<ul>`. Chaque item de menu est un `<li>`.

```html
<nav>
  <ul>
    <li>Accueil</li>
    <li>À propos</li>
    <li>Projets</li>
    <li>Contact</li>
  </ul>
</nav>
```

Pourquoi une liste ? Parce que c'est ce que c'est sémantiquement : une liste d'éléments de navigation. Le CSS se chargera ensuite de les afficher horizontalement au lieu de verticalement.

### Les liens avec `<a>` : anchor tags

Les items du menu doivent être cliquables. J'utilise les balises `<a>` (*anchor*) à l'intérieur des `<li>`.

```html
<nav>
  <ul>
    <li><a href="index.html">Accueil</a></li>
    <li><a href="about.html">À propos</a></li>
    <li><a href="projects.html">Projets</a></li>
    <li><a href="#contact">Contact</a></li>
  </ul>
</nav>
```

Types de liens :

**Lien vers une autre page (chemin relatif) :**
```html
<a href="about.html">À propos</a>
```

**Lien vers une URL externe :**
```html
<a href="https://github.com/pierre" target="_blank">Mon GitHub</a>
```
`target="_blank"` ouvre dans un nouvel onglet.

**Lien vers une ancre sur la même page :**
```html
<!-- Le lien -->
<a href="#contact">Aller au formulaire</a>

<!-- La cible quelque part plus bas dans la page -->
<section id="contact">
  ...
</section>
```
Le `#` devant l'id permet de naviguer vers une section de la même page sans recharger.

**Navigation complète assemblée :**
```html
<nav>
  <div class="nav-container">
    <div class="logo">
      <a href="index.html">MonSite</a>
    </div>
    <ul class="nav-links">
      <li><a href="index.html">Accueil</a></li>
      <li><a href="about.html">À propos</a></li>
      <li><a href="projects.html">Projets</a></li>
      <li><a href="#contact">Contact</a></li>
    </ul>
  </div>
</nav>
```

---

## 4. Le header

Le **header** (ou *hero section*) c'est la grande zone d'accroche en haut de la page, juste sous la navigation. C'est ce que l'utilisateur voit en premier. Elle contient généralement un titre fort et un bouton d'appel à l'action (CTA, *Call To Action*).

### La balise `<header>`

```html
<header>
  <!-- Contenu du header -->
</header>
```

Comme `<nav>`, `<header>` est une balise sémantique HTML5. Elle indique que ce bloc est l'en-tête de la page (ou d'une section).

### Le titre avec `<h1>`

Le `<h1>` est le titre le plus important de la page. Il ne doit y en avoir **qu'un seul** par page.

```html
<header>
  <h1>Pierre Dupont</h1>
  <p>Analyste en cybersécurité · GRC & Consulting</p>
</header>
```

### Le bouton avec `<button>`

```html
<button type="button">Voir mes projets</button>
```

L'attribut `type` peut prendre trois valeurs :
* `type="button"` : bouton simple, ne fait rien par défaut (comportement défini en JS)
* `type="submit"` : soumet un formulaire (on le verra dans la partie formulaires)
* `type="reset"` : réinitialise un formulaire

**Bouton vs lien : quand utiliser quoi ?**

* `<a>` pour naviguer vers une URL
* `<button>` pour déclencher une action (ouvrir un modal, soumettre un formulaire, déclencher du JS)

En pratique on voit souvent des `<a>` stylés en bouton pour les CTA qui naviguent vers une autre page :
```html
<a href="projects.html" class="btn">Voir mes projets</a>
```

**Header complet :**
```html
<header>
  <h1>Pierre Dupont</h1>
  <p>Analyste en cybersécurité · GRC & Consulting</p>
  <button type="button">Découvrir mes projets</button>
</header>
```

---

## 5. Les cards

Les **cards** (cartes) sont des blocs d'information autonomes et répétables. On les voit partout : articles de blog, profils, produits, projets... Une card regroupe un visuel, un titre, un texte et parfois un bouton.

### Titre et paragraphe : `<h2>` et `<p>`

Dans une card, le titre est généralement un `<h2>` (le `<h1>` étant réservé au titre principal de la page).

```html
<h2>Titre de la card</h2>
<p>Description courte du contenu de la card. Quelques phrases suffisent.</p>
```

### Structure de base d'une card

Une card c'est un `<div>` avec une classe, qui contient des éléments organisés :

```html
<div class="card">
  <h2>Reconnaissance réseau</h2>
  <p>Techniques et outils pour cartographier une infrastructure cible.</p>
  <a href="projet.html">Voir le projet</a>
</div>
```

Pour plusieurs cards côte à côte, je les regroupe dans un conteneur :

```html
<section class="cards-container">

  <div class="card">
    <h2>Projet 1</h2>
    <p>Description du premier projet.</p>
    <a href="#">Voir plus</a>
  </div>

  <div class="card">
    <h2>Projet 2</h2>
    <p>Description du deuxième projet.</p>
    <a href="#">Voir plus</a>
  </div>

  <div class="card">
    <h2>Projet 3</h2>
    <p>Description du troisième projet.</p>
    <a href="#">Voir plus</a>
  </div>

</section>
```

La balise `<section>` est une autre balise sémantique HTML5. Elle représente une section thématique de la page.

### Ajouter une image avec `<img>`

```html
<div class="card">
  <img src="images/projet1.png" alt="Capture du rapport de reconnaissance">
  <h2>Reconnaissance réseau</h2>
  <p>Techniques et outils pour cartographier une infrastructure cible.</p>
  <a href="#">Voir le projet</a>
</div>
```

Attributs de `<img>` :

| Attribut | Rôle | Obligatoire |
|----------|------|-------------|
| `src` | Chemin ou URL de l'image | Oui |
| `alt` | Texte alternatif si l'image ne charge pas | Oui (accessibilité) |
| `width` | Largeur en pixels | Non |
| `height` | Hauteur en pixels | Non |

**Chemins d'image :**
```html
<!-- Image locale dans un dossier images/ -->
<img src="images/photo.jpg" alt="Ma photo">

<!-- Image dans le même dossier que le HTML -->
<img src="photo.jpg" alt="Ma photo">

<!-- Image distante via URL -->
<img src="https://exemple.com/image.png" alt="Image externe">
```

**Note sécu** : les balises `<img>` avec des sources externes chargent une ressource depuis un serveur tiers à chaque affichage de la page. Ce serveur peut logger mon IP, mon user-agent, l'heure de visite. C'est la technique du **tracking pixel** : une image 1x1 invisible utilisée pour tracker les ouvertures d'emails ou les visites de pages.

---

## 6. Les formulaires

Les formulaires permettent à l'utilisateur d'envoyer des données au serveur. C'est la partie du HTML la plus sensible en termes de sécurité. Je fais attention à chaque attribut ici.

### `<form>` : le conteneur

```html
<form action="/submit" method="POST">
  <!-- Les champs du formulaire -->
</form>
```

Deux attributs critiques :

**`action`** : l'URL vers laquelle les données sont envoyées quand le formulaire est soumis. Si absent, le formulaire soumet vers la page courante.

**`method`** : la méthode HTTP utilisée pour envoyer les données.
* `GET` : les données apparaissent dans l'URL (`/search?q=ma+recherche`). Visible dans les logs, dans l'historique, dans les bookmarks. À utiliser uniquement pour des recherches, jamais pour des données sensibles.
* `POST` : les données sont dans le corps de la requête HTTP, pas dans l'URL. À utiliser pour les logins, les inscriptions, tout ce qui est sensible.

**Note sécu** : `POST` ne veut pas dire sécurisé. Les données POST sont quand même en clair si la connexion n'est pas en HTTPS. Ce qui sécurise c'est HTTPS, pas la méthode.

### `<input>` : les champs de saisie

`<input>` est une balise auto-fermante. Son comportement change selon l'attribut `type`.

```html
<!-- Champ texte simple -->
<input type="text" name="username" placeholder="Nom d'utilisateur">

<!-- Mot de passe (caractères masqués) -->
<input type="password" name="password" placeholder="Mot de passe">

<!-- Email (validation de format côté navigateur) -->
<input type="email" name="email" placeholder="mon@email.com">

<!-- Nombre -->
<input type="number" name="age" min="18" max="99">

<!-- Case à cocher -->
<input type="checkbox" name="conditions" id="conditions">

<!-- Bouton radio (choix unique dans un groupe) -->
<input type="radio" name="role" value="admin"> Admin
<input type="radio" name="role" value="user"> Utilisateur

<!-- Champ caché (non visible mais envoyé au serveur) -->
<input type="hidden" name="csrf_token" value="abc123xyz">

<!-- Sélecteur de fichier -->
<input type="file" name="document">
```

Attributs importants de `<input>` :

| Attribut | Rôle |
|----------|------|
| `type` | Type de champ |
| `name` | Nom du paramètre envoyé au serveur |
| `id` | Identifiant unique (pour le lier à un `<label>`) |
| `placeholder` | Texte indicatif grisé dans le champ vide |
| `value` | Valeur pré-remplie ou valeur d'un radio/checkbox |
| `required` | Rend le champ obligatoire (validation navigateur) |
| `disabled` | Désactive le champ |

**Note sécu sur `type="hidden"`** : ces champs sont invisibles à l'écran mais présents dans le HTML et dans la requête envoyée. Ils servent souvent à transporter des tokens CSRF ou des identifiants de session. Un attaquant qui inspecte le code source les voit immédiatement. C'est souvent là que se trouvent des informations intéressantes sur l'architecture backend.

### `<label>` : étiquetter les champs

Le `<label>` associe un texte à un champ. L'attribut `for` doit correspondre à l'`id` de l'input.

```html
<label for="username">Nom d'utilisateur :</label>
<input type="text" id="username" name="username">
```

Pourquoi c'est important :
* Cliquer sur le label donne le focus au champ (meilleure UX)
* Indispensable pour l'accessibilité (lecteurs d'écran)

### `<textarea>` : zone de texte multi-lignes

Pour les messages longs (formulaire de contact, commentaires...).

```html
<textarea name="message" rows="6" cols="50" placeholder="Votre message..."></textarea>
```

Contrairement à `<input>`, `<textarea>` a une balise fermante. Le contenu par défaut se place entre les deux balises.

Attributs utiles :
* `rows` : nombre de lignes visibles
* `cols` : largeur en caractères
* `placeholder` : texte indicatif
* `maxlength` : nombre maximum de caractères autorisés

### Le bouton de soumission

Deux façons de créer le bouton d'envoi :

```html
<!-- Recommandé -->
<button type="submit">Envoyer</button>

<!-- Ancienne façon, fonctionne aussi -->
<input type="submit" value="Envoyer">
```

Je préfère `<button type="submit">` parce qu'il accepte du HTML à l'intérieur (icônes, texte formaté...).

### Formulaire de contact complet

```html
<section id="contact">
  <h2>Me contacter</h2>

  <form action="/contact" method="POST">

    <div class="form-group">
      <label for="nom">Nom :</label>
      <input type="text" id="nom" name="nom" placeholder="Votre nom" required>
    </div>

    <div class="form-group">
      <label for="email">Email :</label>
      <input type="email" id="email" name="email" placeholder="votre@email.com" required>
    </div>

    <div class="form-group">
      <label for="sujet">Sujet :</label>
      <input type="text" id="sujet" name="sujet" placeholder="Objet de votre message">
    </div>

    <div class="form-group">
      <label for="message">Message :</label>
      <textarea id="message" name="message" rows="6" placeholder="Votre message..." required></textarea>
    </div>

    <button type="submit">Envoyer le message</button>

  </form>
</section>
```

---

## 7. Page complète : exemple récapitulatif

Voilà ce que donne une page assemblée avec tout ce qu'on a vu :

```html
<!DOCTYPE html>
<html lang="fr">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Portfolio de Pierre - Analyste cybersécurité">
  <title>Pierre | Cybersécurité</title>
  <link rel="stylesheet" href="style.css">
</head>

<body>

  <!-- NAVIGATION -->
  <nav>
    <div class="nav-container">
      <div class="logo">
        <a href="index.html">Pierre</a>
      </div>
      <ul class="nav-links">
        <li><a href="#accueil">Accueil</a></li>
        <li><a href="#projets">Projets</a></li>
        <li><a href="#contact">Contact</a></li>
      </ul>
    </div>
  </nav>

  <!-- HEADER -->
  <header id="accueil">
    <h1>Pierre Dupont</h1>
    <p>Analyste en cybersécurité · GRC & Consulting</p>
    <button type="button">Voir mes projets</button>
  </header>

  <!-- CARDS / PROJETS -->
  <section id="projets">
    <h2>Mes projets</h2>

    <div class="cards-container">

      <div class="card">
        <img src="images/recon.png" alt="Projet reconnaissance réseau">
        <h3>Reconnaissance réseau</h3>
        <p>Cartographie d'une infrastructure avec Nmap et Shodan.</p>
        <a href="projets/recon.html">Voir le writeup</a>
      </div>

      <div class="card">
        <img src="images/arp.png" alt="Projet ARP spoofing">
        <h3>ARP Spoofing</h3>
        <p>Démonstration d'une attaque Man-in-the-Middle sur réseau local.</p>
        <a href="projets/arp.html">Voir le writeup</a>
      </div>

      <div class="card">
        <img src="images/xss.png" alt="Projet XSS">
        <h3>Cross-Site Scripting</h3>
        <p>Identification et exploitation d'une vulnérabilité XSS réfléchie.</p>
        <a href="projets/xss.html">Voir le writeup</a>
      </div>

    </div>
  </section>

  <!-- FORMULAIRE DE CONTACT -->
  <section id="contact">
    <h2>Me contacter</h2>

    <form action="/contact" method="POST">

      <div class="form-group">
        <label for="nom">Nom :</label>
        <input type="text" id="nom" name="nom" placeholder="Votre nom" required>
      </div>

      <div class="form-group">
        <label for="email">Email :</label>
        <input type="email" id="email" name="email" placeholder="votre@email.com" required>
      </div>

      <div class="form-group">
        <label for="message">Message :</label>
        <textarea id="message" name="message" rows="6" placeholder="Votre message..." required></textarea>
      </div>

      <button type="submit">Envoyer</button>

    </form>
  </section>

</body>

</html>
```

---

## Cheatsheet de cette partie

### Balises sémantiques HTML5

| Balise | Rôle |
|--------|------|
| `<nav>` | Zone de navigation |
| `<header>` | En-tête de page ou de section |
| `<main>` | Contenu principal |
| `<section>` | Section thématique |
| `<article>` | Contenu autonome |
| `<footer>` | Pied de page |

### Head

| Balise | Rôle |
|--------|------|
| `<meta charset="UTF-8">` | Encodage |
| `<meta name="viewport" ...>` | Responsive mobile |
| `<meta name="description" ...>` | Description SEO |
| `<title>` | Titre de l'onglet |
| `<link rel="stylesheet" href="...">` | Feuille CSS externe |

### Formulaires

| Balise / Attribut | Rôle |
|-------------------|------|
| `<form action="..." method="...">` | Conteneur du formulaire |
| `method="GET"` | Données dans l'URL |
| `method="POST"` | Données dans le corps de requête |
| `<input type="text">` | Champ texte |
| `<input type="password">` | Champ mot de passe masqué |
| `<input type="email">` | Champ email avec validation |
| `<input type="hidden">` | Champ invisible (CSRF tokens...) |
| `<input type="checkbox">` | Case à cocher |
| `<input type="radio">` | Bouton radio |
| `<textarea>` | Zone de texte multiligne |
| `<button type="submit">` | Bouton d'envoi |
| `<label for="id">` | Étiquette d'un champ |
| `required` | Champ obligatoire |
| `placeholder` | Texte indicatif |

---

## Notes sécu à retenir

* Toujours lire le `<head>` en premier lors d'une analyse de page : scripts externes, ressources chargées, métadonnées révélatrices.
* Les `<input type="hidden">` sont invisibles à l'écran mais visibles dans le code source et dans les requêtes interceptées. Mine d'or pour comprendre comment une appli fonctionne.
* Un formulaire en `method="GET"` qui gère des données sensibles (login, search avec données perso) est une mauvaise pratique : les données finissent dans les logs du serveur et l'historique du navigateur.
* Un `<a href>` peut pointer n'importe où indépendamment du texte affiché. Ne jamais faire confiance au texte d'un lien, toujours vérifier la vraie destination.
* Les images externes trackent. Un simple `<img src="https://tracker.io/pixel.png">` invisible dans une page ou un email suffit à logger chaque visiteur.
