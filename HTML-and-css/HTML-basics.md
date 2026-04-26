# HTML : Les bases

> Mes notes de formation Jedha sur HTML. Pas du développement web à plein temps, mais comprendre HTML c'est indispensable en sécu : phishing, XSS, analyse de pages web, tout passe par là.

## Sommaire

1. [C'est quoi HTML et à quoi ça sert](#1-cest-quoi-html-et-à-quoi-ça-sert)
2. [Les balises HTML](#2-les-balises-html)
3. [Structure de base d'une page HTML](#3-structure-de-base-dune-page-html)
4. [Les balises essentielles à connaître](#4-les-balises-essentielles-à-connaître)
5. [Cheatsheet rapide](#5-cheatsheet-rapide)

---

## 1. C'est quoi HTML et à quoi ça sert

**HTML** = *HyperText Markup Language*.

C'est le langage qui structure le contenu d'une page web. Pas un langage de programmation : pas de logique, pas de calculs, pas de conditions. C'est un **langage de balisage**, c'est-à-dire qu'il décrit ce qu'est chaque morceau de contenu : un titre, un paragraphe, une image, un lien, un tableau...

Quand mon navigateur reçoit une page web, il reçoit du HTML. Il le lit, l'interprète, et affiche le résultat visuel. Ce que je vois sur une page c'est le rendu, pas le code brut.

**Les trois piliers d'une page web :**
* **HTML** = la structure (le squelette)
* **CSS** = la mise en forme (l'apparence)
* **JavaScript** = le comportement (l'interactivité)

HTML s'occupe uniquement du "quoi". CSS dira "comment ça ressemble". JS dira "quoi faire quand l'utilisateur clique".

**Pourquoi c'est important en cybersécurité :**

Je vais lire du HTML constamment. Les attaques de type **XSS** (*Cross-Site Scripting*) consistent à injecter du code malveillant dans le HTML d'une page. Les pages de **phishing** sont du HTML copié-modifié pour tromper l'utilisateur. Analyser une page suspecte dans Burp Suite ou dans les DevTools du navigateur, c'est lire du HTML. Autant savoir ce que je lis.

---

## 2. Les balises HTML

### C'est quoi une balise

Le HTML fonctionne avec des **balises** (*tags*). Une balise c'est un mot-clé entouré de chevrons `< >`.

La plupart des balises fonctionnent par paire : une **balise ouvrante** et une **balise fermante**. La fermante a un slash `/` avant le mot-clé.

```html
<p>Mon texte ici.</p>
```

Ici :
* `<p>` = balise ouvrante (p = paragraphe)
* `Mon texte ici.` = le contenu
* `</p>` = balise fermante

Tout ce qui est entre les deux balises est le contenu de cet élément.

### Les balises auto-fermantes

Certaines balises n'ont pas de contenu à entourer, elles se suffisent à elles-mêmes. On les appelle **balises auto-fermantes** ou *void elements*.

```html
<img src="photo.jpg" alt="Ma photo">
<br>
<hr>
<input type="text">
```

Pas besoin de `</img>` ou `</br>`. Elles n'encadrent rien.

### Les attributs

Les balises peuvent avoir des **attributs** : des informations supplémentaires placées dans la balise ouvrante.

Format : `attribut="valeur"`

```html
<a href="https://google.com">Clique ici</a>
```

Ici `href` est un attribut de la balise `<a>` (lien hypertexte). Il indique vers où pointe le lien.

Autre exemple avec plusieurs attributs :
```html
<img src="photo.jpg" alt="Portrait" width="300">
```

* `src` = chemin vers l'image
* `alt` = texte alternatif (si l'image ne charge pas)
* `width` = largeur en pixels

### L'imbrication

Les balises peuvent s'imbriquer les unes dans les autres. C'est ce qui crée la **hiérarchie** du document.

```html
<div>
  <p>Ce paragraphe est <strong>à l'intérieur</strong> d'une div.</p>
</div>
```

Règle à respecter : fermer dans l'ordre inverse de l'ouverture. Si j'ouvre `<div>` puis `<p>`, je ferme `</p>` avant `</div>`.

```html
<!-- Correct -->
<div><p>Texte</p></div>

<!-- Incorrect : croisement de balises -->
<div><p>Texte</div></p>
```

### Les commentaires

En HTML, un commentaire ne s'affiche pas dans le navigateur mais est visible dans le code source. Je vais souvent trouver des infos intéressantes dans les commentaires d'une page lors d'une analyse.

```html
<!-- Ceci est un commentaire -->
<!-- TODO: supprimer les credentials de test avant le déploiement -->
```

---

## 3. Structure de base d'une page HTML

Toute page HTML valide respecte une structure minimale. La voilà :

```html
<!DOCTYPE html>
<html lang="fr">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Le titre de ma page</title>
  </head>
  <body>
    <h1>Bonjour</h1>
    <p>Le contenu visible de ma page est ici.</p>
  </body>
</html>
```

Je décompose chaque partie :

### `<!DOCTYPE html>`

Pas vraiment une balise HTML, c'est une **déclaration**. Elle indique au navigateur que ce document utilise HTML5 (la version actuelle). Toujours en première ligne, toujours là.

### `<html>`

La balise racine. Tout le reste est à l'intérieur. L'attribut `lang` indique la langue du document, utile pour les lecteurs d'écran et le SEO.

```html
<html lang="fr">
  ...
</html>
```

### `<head>`

Le **head** contient les métadonnées de la page : des infos sur le document qui ne s'affichent pas directement dans la page mais que le navigateur utilise.

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mon site</title>
  <link rel="stylesheet" href="style.css">
</head>
```

Ce qu'on trouve dans le `<head>` :

* `<meta charset="UTF-8">` : encodage des caractères. UTF-8 gère les accents, les emojis, tout. Toujours présent.
* `<meta name="viewport" ...>` : gère l'affichage sur mobile.
* `<title>` : le titre qui s'affiche dans l'onglet du navigateur.
* `<link>` : pour lier une feuille de style CSS externe.
* `<script>` : pour lier un fichier JavaScript.

**Note sécu** : le `<head>` est souvent là où sont chargés des scripts tiers, des trackers, des CDN. Quand j'analyse une page, c'est un des premiers endroits à regarder.

### `<body>`

Le **body** contient tout ce qui s'affiche visuellement dans le navigateur. Tout le contenu visible : titres, textes, images, liens, formulaires... C'est ici que ça se passe.

```html
<body>
  <h1>Titre principal</h1>
  <p>Mon paragraphe.</p>
</body>
```

---

## 4. Les balises essentielles à connaître

### Titres : `<h1>` à `<h6>`

Six niveaux de titres, du plus important au moins important.

```html
<h1>Titre principal (un seul par page)</h1>
<h2>Sous-titre</h2>
<h3>Sous-sous-titre</h3>
<h4>Encore en dessous</h4>
<h5>Rarement utilisé</h5>
<h6>Pratiquement jamais</h6>
```

`<h1>` est le titre le plus important, `<h6>` le moins. En pratique j'utilise surtout `h1`, `h2`, `h3`.

### Paragraphe : `<p>`

```html
<p>Mon paragraphe de texte. Le navigateur ajoute automatiquement de l'espace avant et après.</p>
```

### Liens : `<a>`

*Anchor*. Crée un lien cliquable.

```html
<a href="https://google.com">Aller sur Google</a>
<a href="/contact">Page de contact (lien interne)</a>
<a href="https://google.com" target="_blank">Ouvrir dans un nouvel onglet</a>
```

L'attribut `href` (*HyperText REFerence*) indique la destination. `target="_blank"` ouvre dans un nouvel onglet.

**Note sécu** : les liens `<a>` sont au cœur du phishing. Un lien peut afficher `mabanque.fr` comme texte mais pointer vers `evil-site.com` via `href`. Toujours vérifier où pointe réellement un lien avant de cliquer.

### Images : `<img>`

```html
<img src="photo.jpg" alt="Description de l'image">
<img src="https://exemple.com/image.png" alt="Image distante" width="400">
```

* `src` : chemin ou URL de l'image
* `alt` : texte alternatif (obligatoire pour l'accessibilité)
* `width` / `height` : dimensions

### Listes

**Liste non ordonnée** (à puces) :
```html
<ul>
  <li>Premier élément</li>
  <li>Deuxième élément</li>
  <li>Troisième élément</li>
</ul>
```

**Liste ordonnée** (numérotée) :
```html
<ol>
  <li>Première étape</li>
  <li>Deuxième étape</li>
  <li>Troisième étape</li>
</ol>
```

### Mise en forme du texte

```html
<strong>Texte en gras (important)</strong>
<em>Texte en italique (emphase)</em>
<br>          <!-- Saut de ligne -->
<hr>          <!-- Ligne horizontale de séparation -->
```

### Conteneurs : `<div>` et `<span>`

Ce sont des balises sans signification sémantique particulière. Elles servent juste à regrouper du contenu pour le cibler en CSS ou en JavaScript.

```html
<!-- div : conteneur de bloc (prend toute la largeur) -->
<div class="carte">
  <h2>Titre de la carte</h2>
  <p>Contenu de la carte</p>
</div>

<!-- span : conteneur en ligne (dans du texte) -->
<p>Ce mot est <span style="color: red;">rouge</span> dans la phrase.</p>
```

### Formulaires : `<form>`

Les formulaires sont partout et sont une cible majeure en sécu.

```html
<form action="/login" method="POST">
  <label for="username">Nom d'utilisateur :</label>
  <input type="text" id="username" name="username">

  <label for="password">Mot de passe :</label>
  <input type="password" id="password" name="password">

  <button type="submit">Se connecter</button>
</form>
```

Points importants :
* `action` : l'URL vers laquelle les données sont envoyées
* `method` : `GET` (données dans l'URL) ou `POST` (données dans le corps de la requête)
* `input type="password"` : masque le texte saisi
* `name` : le nom du champ tel qu'il sera transmis au serveur

**Note sécu** : les formulaires HTML ne chiffrent rien par eux-mêmes. C'est HTTPS qui sécurise le transport. Un formulaire en HTTP envoie les données en clair sur le réseau. Les champs `name` des inputs sont exactement les paramètres qu'on voit dans les requêtes HTTP interceptées avec Burp Suite.

### Tableaux : `<table>`

```html
<table>
  <thead>
    <tr>
      <th>Nom</th>
      <th>Port</th>
      <th>Protocole</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>HTTP</td>
      <td>80</td>
      <td>TCP</td>
    </tr>
    <tr>
      <td>HTTPS</td>
      <td>443</td>
      <td>TCP</td>
    </tr>
  </tbody>
</table>
```

* `<table>` : le tableau
* `<thead>` : en-tête du tableau
* `<tbody>` : corps du tableau
* `<tr>` : une ligne (*table row*)
* `<th>` : une cellule d'en-tête (*table header*)
* `<td>` : une cellule de données (*table data*)

---

## 5. Cheatsheet rapide

### Structure

| Balise | Rôle |
|--------|------|
| `<!DOCTYPE html>` | Déclaration HTML5 |
| `<html>` | Racine du document |
| `<head>` | Métadonnées (non visibles) |
| `<title>` | Titre de l'onglet |
| `<meta charset="UTF-8">` | Encodage caractères |
| `<body>` | Contenu visible |

### Contenu

| Balise | Rôle |
|--------|------|
| `<h1>` à `<h6>` | Titres (du plus au moins important) |
| `<p>` | Paragraphe |
| `<a href="...">` | Lien |
| `<img src="..." alt="...">` | Image |
| `<ul>` + `<li>` | Liste à puces |
| `<ol>` + `<li>` | Liste numérotée |
| `<strong>` | Gras |
| `<em>` | Italique |
| `<br>` | Saut de ligne |
| `<hr>` | Ligne de séparation |

### Mise en page

| Balise | Rôle |
|--------|------|
| `<div>` | Conteneur bloc |
| `<span>` | Conteneur en ligne |

### Formulaires

| Balise | Rôle |
|--------|------|
| `<form>` | Formulaire |
| `<input type="text">` | Champ texte |
| `<input type="password">` | Champ mot de passe |
| `<input type="email">` | Champ email |
| `<input type="checkbox">` | Case à cocher |
| `<button type="submit">` | Bouton d'envoi |
| `<label>` | Étiquette d'un champ |

### Commentaire

```html
<!-- Ceci ne s'affiche pas dans le navigateur mais est visible dans le code source -->
```

---

## Pour aller plus loin

Deux réflexes à prendre maintenant :

* **Clic droit > Inspecter** sur n'importe quelle page web pour voir son HTML en direct dans les DevTools. C'est le meilleur terrain d'entraînement.
* **Clic droit > Afficher la source** pour voir le HTML brut tel qu'il a été envoyé par le serveur.

Ces deux outils sont aussi ceux d'un attaquant en phase de reconnaissance web. Les maîtriser comme utilisateur, c'est les comprendre comme analyste.
