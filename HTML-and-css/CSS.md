# CSS : Introduction complète & Déploiement

> Mes notes sur CSS de zéro jusqu'au déploiement. CSS c'est ce qui transforme une page HTML brute en quelque chose de présentable. Je couvre ici toutes les façons d'écrire du CSS, les sélecteurs, les unités, et comment mettre le site en ligne avec Vercel.

## Sommaire

1. [C'est quoi CSS](#1-cest-quoi-css)
2. [Les trois façons d'écrire du CSS](#2-les-trois-façons-décrire-du-css)
   * Inline styling
   * Embedded styling
   * External stylesheet
3. [Les sélecteurs](#3-les-sélecteurs)
   * Sélecteur de balise
   * Sélecteur d'ID
   * Sélecteur de classe
4. [Styliser les boutons](#4-styliser-les-boutons)
5. [Les unités CSS](#5-les-unités-css)
   * Unités absolues
   * Unités relatives
6. [Plus loin en CSS](#6-plus-loin-en-css)
   * Box model
   * Couleurs
   * Pseudo-classes
   * Positionnement
7. [Déployer avec Vercel](#7-déployer-avec-vercel)
8. [Cheatsheet CSS](#8-cheatsheet-css)

---

## 1. C'est quoi CSS

**CSS** = *Cascading Style Sheets*. Feuilles de style en cascade.

C'est le langage qui contrôle l'apparence visuelle d'une page HTML. HTML structure le contenu, CSS le met en forme : couleurs, polices, tailles, espacements, positions, animations...

Le mot **cascade** est important. CSS applique ses règles dans un ordre de priorité précis quand plusieurs règles s'appliquent au même élément. Comprendre la cascade m'évite beaucoup de bugs incompréhensibles.

### La syntaxe de base

Une règle CSS se compose d'un **sélecteur** et d'un **bloc de déclarations** :

```css
sélecteur {
  propriété: valeur;
  propriété: valeur;
}
```

Exemple concret :

```css
h1 {
  color: #1a1a2e;
  font-size: 48px;
  font-weight: 700;
}
```

* `h1` : le sélecteur (qui est ciblé)
* `color`, `font-size`, `font-weight` : les propriétés (quoi modifier)
* `#1a1a2e`, `48px`, `700` : les valeurs (comment le modifier)

Chaque déclaration se termine par un `;`. Le bloc est entre `{ }`.

---

## 2. Les trois façons d'écrire du CSS

### Inline styling : directement dans la balise HTML

Je place le CSS directement dans l'attribut `style` d'une balise HTML.

```html
<p style="color: red; font-size: 18px;">Ce texte est rouge.</p>
<h1 style="color: #1a1a2e; text-align: center;">Mon titre</h1>
```

**Avantages :**
* Rapide pour tester
* Priorité maximale dans la cascade (écrase tout le reste)

**Inconvénients :**
* Impossible à maintenir sur un vrai projet
* Si je veux changer la couleur de 50 paragraphes, je modifie 50 lignes
* Mélange structure (HTML) et présentation (CSS), mauvaise pratique

**Quand je l'utilise :** pour tester une propriété rapidement. Jamais en production.

**Note sécu :** le `style` inline est parfois utilisé pour du contenu malveillant injecté via XSS. Un `style="display:none"` peut cacher du contenu, `style="position:fixed"` peut superposer un faux formulaire sur la page. À garder en tête lors de l'analyse de pages suspectes.

### Embedded styling : dans le `<head>` avec `<style>`

Je place le CSS dans une balise `<style>` dans le `<head>` du document HTML.

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Mon site</title>

  <style>
    body {
      font-family: 'Inter', sans-serif;
      background-color: #f8f9fa;
      color: #1a1a2e;
    }

    h1 {
      font-size: 48px;
      color: #6c63ff;
    }

    p {
      line-height: 1.7;
    }
  </style>

</head>
<body>
  <h1>Mon titre</h1>
  <p>Mon paragraphe.</p>
</body>
</html>
```

**Avantages :**
* Tout dans un seul fichier (pratique pour des exemples courts)
* Meilleur que l'inline pour la maintenabilité

**Inconvénients :**
* Le CSS est lié à une seule page HTML
* Sur un site de plusieurs pages, je duplique le CSS dans chaque fichier

**Quand je l'utilise :** pour des prototypes rapides, des pages uniques, des emails HTML.

### External stylesheet : fichier CSS séparé

La bonne pratique. Je crée un fichier `style.css` séparé et je le lie à mon HTML avec un `<link>`.

**index.html :**
```html
<head>
  <meta charset="UTF-8">
  <title>Mon site</title>
  <link rel="stylesheet" href="style.css">
</head>
```

**style.css :**
```css
body {
  font-family: 'Inter', sans-serif;
  background-color: #f8f9fa;
  color: #1a1a2e;
  margin: 0;
  padding: 0;
}

h1 {
  font-size: 48px;
  color: #6c63ff;
}

p {
  line-height: 1.7;
}
```

**Avantages :**
* Un seul fichier CSS pour tout le site : je change une couleur, ça change partout
* HTML et CSS séparés : code plus propre, plus maintenable
* Le navigateur met en cache le fichier CSS : performances améliorées

**C'est cette méthode que j'utilise toujours sur un vrai projet.**

### L'ordre de priorité (la cascade)

Quand plusieurs règles s'appliquent au même élément, CSS les applique dans cet ordre de priorité (du moins prioritaire au plus prioritaire) :

1. Styles du navigateur (par défaut)
2. External stylesheet
3. Embedded stylesheet (`<style>` dans `<head>`)
4. Inline style (`style="..."` dans la balise)
5. `!important` (à éviter, ça force une règle à primer sur tout)

```css
/* Éviter autant que possible */
p {
  color: red !important;
}
```

`!important` c'est le patch de la dernière chance. Si j'en mets partout, je perds le contrôle de ma cascade.

---

## 3. Les sélecteurs

Les sélecteurs me permettent de cibler précisément les éléments HTML que je veux styliser.

### Sélecteur de balise (type)

Cible tous les éléments d'un certain type.

```css
/* Tous les paragraphes */
p {
  color: #333;
  line-height: 1.7;
}

/* Tous les h2 */
h2 {
  font-size: 32px;
  margin-bottom: 16px;
}

/* Tous les liens */
a {
  color: #6c63ff;
  text-decoration: none;
}
```

### Sélecteur d'ID

Cible un élément **unique** identifié par son attribut `id`. En CSS, l'ID est précédé de `#`.

**HTML :**
```html
<nav id="main-nav">...</nav>
<section id="hero">...</section>
<footer id="site-footer">...</footer>
```

**CSS :**
```css
#main-nav {
  background-color: #1a1a2e;
  padding: 20px 40px;
}

#hero {
  min-height: 100vh;
  display: flex;
  align-items: center;
}

#site-footer {
  background-color: #0d0d1a;
  color: #aaaaaa;
}
```

**Règle :** un ID doit être unique sur la page. Si plusieurs éléments ont le même `id`, c'est une erreur HTML. Pour des groupes d'éléments similaires, j'utilise des classes.

**Spécificité :** le sélecteur ID a une priorité très haute dans la cascade. Si je me bats avec un style qui ne s'applique pas, c'est souvent parce qu'un ID l'écrase quelque part.

### Sélecteur de classe

Cible tous les éléments qui ont une `class` donnée. En CSS, la classe est précédée de `.`.

**HTML :**
```html
<div class="card">...</div>
<div class="card">...</div>
<div class="card featured">...</div>  <!-- Deux classes sur un même élément -->
```

**CSS :**
```css
.card {
  background-color: #ffffff;
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

/* La card featured en plus */
.featured {
  border: 2px solid #6c63ff;
}
```

Un élément peut avoir **plusieurs classes** séparées par des espaces. Les styles des deux classes s'appliquent et se cumulent.

### Sélecteurs combinés

```css
/* Tous les <p> à l'intérieur d'une .card */
.card p {
  font-size: 14px;
  color: #666;
}

/* Un <h2> directement enfant d'une .card (pas les petits-enfants) */
.card > h2 {
  font-size: 20px;
}

/* Cibler plusieurs sélecteurs en même temps */
h1, h2, h3 {
  font-family: 'Space Grotesk', sans-serif;
  font-weight: 700;
}

/* Un élément avec à la fois .card ET .featured */
.card.featured {
  border: 2px solid #6c63ff;
}
```

### Récap : ID vs Classe

| | ID (`#`) | Classe (`.`) |
|--|----------|--------------|
| Symbole | `#mon-id` | `.ma-classe` |
| Unicité | Un seul par page | Autant d'éléments que je veux |
| HTML | `id="mon-id"` | `class="ma-classe"` |
| Usage | Élément unique (header, footer, nav) | Groupes d'éléments similaires (cards, boutons) |
| Priorité cascade | Haute | Normale |

---

## 4. Styliser les boutons

Les boutons ont des styles par défaut moche que je dois réécrire. Voici comment en faire de beaux.

### Reset du style par défaut

```css
button {
  /* Supprimer les styles navigateur */
  border: none;
  outline: none;
  cursor: pointer;
  background: none;

  /* Mes styles */
  font-family: inherit;   /* Hérite la font du parent (sinon le bouton a sa propre font) */
}
```

### Un bouton simple

```css
.btn {
  display: inline-block;
  padding: 12px 28px;
  background-color: #6c63ff;
  color: #ffffff;
  font-size: 16px;
  font-weight: 600;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  text-decoration: none;    /* Si c'est un <a> stylé en bouton */
  transition: background-color 0.2s, transform 0.1s;
}

.btn:hover {
  background-color: #5a52e0;
  transform: translateY(-2px);   /* Léger effet de soulèvement au survol */
}

.btn:active {
  transform: translateY(0);      /* Retour à la position normale au clic */
}
```

### Variantes de boutons

```css
/* Bouton primaire : action principale */
.btn-primary {
  background-color: #6c63ff;
  color: #ffffff;
}

/* Bouton secondaire : action secondaire, contour */
.btn-secondary {
  background-color: transparent;
  color: #6c63ff;
  border: 2px solid #6c63ff;
}

.btn-secondary:hover {
  background-color: #6c63ff;
  color: #ffffff;
}

/* Bouton danger : actions destructives */
.btn-danger {
  background-color: #e74c3c;
  color: #ffffff;
}

/* Bouton désactivé */
.btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
  transform: none;
}
```

**HTML correspondant :**
```html
<button class="btn btn-primary">Voir mes projets</button>
<button class="btn btn-secondary">En savoir plus</button>
<a href="#contact" class="btn btn-primary">Me contacter</a>
```

---

## 5. Les unités CSS

### Unités absolues

Elles ont une valeur fixe, indépendante du contexte.

| Unité | Nom | Usage |
|-------|-----|-------|
| `px` | Pixel | Le plus courant. Taille fixe en pixels écran. |
| `pt` | Point | Pour l'impression. 1pt = 1.33px |
| `cm`, `mm` | Centimètre, millimètre | Impression uniquement |

En pratique je n'utilise presque que `px` parmi les absolues.

```css
.carte {
  width: 300px;
  height: 200px;
  border-radius: 12px;
}
```

### Unités relatives

Leur valeur dépend d'un référentiel : la taille de la font, la taille du viewport, l'élément parent...

#### `em` : relatif à la font-size du parent

```css
body {
  font-size: 16px;
}

h1 {
  font-size: 3em;   /* 3 × 16px = 48px */
  margin-bottom: 1em;  /* 1 × 48px = 48px (relatif à font-size de h1) */
}

p {
  font-size: 1em;   /* 1 × 16px = 16px */
  padding: 1.5em;   /* 1.5 × 16px = 24px */
}
```

`em` peut être source de confusion car il s'applique de façon imbriquée : si un parent est en `2em` et l'enfant en `2em`, l'enfant fait `2 × 2 = 4em` par rapport à la racine.

#### `rem` : relatif à la font-size de la racine (`<html>`)

Plus prévisible que `em`. Toujours relatif à l'élément racine, pas au parent.

```css
html {
  font-size: 16px;   /* La référence */
}

h1 { font-size: 3rem; }    /* Toujours 3 × 16px = 48px */
h2 { font-size: 2rem; }    /* Toujours 2 × 16px = 32px */
p  { font-size: 1rem; }    /* Toujours 1 × 16px = 16px */
```

Je préfère `rem` pour les tailles de texte. Prévisible et facile à maintenir.

#### `%` : relatif au parent

```css
.container {
  width: 80%;         /* 80% de la largeur du parent */
  max-width: 1200px;  /* Mais jamais plus de 1200px */
  margin: 0 auto;     /* Centré horizontalement */
}

img {
  width: 100%;        /* L'image prend toute la largeur de son conteneur */
}
```

#### `vw` et `vh` : relatif au viewport

Le **viewport** c'est la zone visible du navigateur.

* `1vw` = 1% de la largeur du viewport
* `1vh` = 1% de la hauteur du viewport

```css
/* Section qui occupe toute la hauteur de l'écran */
.hero {
  height: 100vh;
}

/* Texte dont la taille s'adapte à la largeur de l'écran */
h1 {
  font-size: 5vw;
}
```

### Quelle unité utiliser quand

| Usage | Unité recommandée |
|-------|------------------|
| Taille de texte | `rem` |
| Padding, margin | `rem` ou `px` |
| Largeur d'un conteneur | `%` ou `px` avec `max-width` |
| Hauteur plein écran | `vh` |
| Border, border-radius | `px` |
| Breakpoints responsive | `px` |

---

## 6. Plus loin en CSS

### Le box model

Chaque élément HTML est une boîte rectangulaire composée de quatre couches :

```
+------------------------------------------+
|               MARGIN                     |  ← Espace extérieur (transparent)
|   +----------------------------------+   |
|   |            BORDER                |   |  ← Bordure
|   |   +------------------------+     |   |
|   |   |        PADDING         |     |   |  ← Espace intérieur
|   |   |   +----------------+   |     |   |
|   |   |   |    CONTENT     |   |     |   |  ← Le contenu (texte, image...)
|   |   |   +----------------+   |     |   |
|   |   +------------------------+     |   |
|   +----------------------------------+   |
+------------------------------------------+
```

```css
.carte {
  /* Contenu */
  width: 300px;
  height: 200px;

  /* Padding : espace entre le contenu et la bordure */
  padding: 24px;            /* Tous côtés */
  padding: 16px 24px;       /* vertical | horizontal */
  padding: 8px 16px 24px 32px;  /* top | right | bottom | left */

  /* Border */
  border: 2px solid #6c63ff;
  border-radius: 12px;

  /* Margin : espace à l'extérieur de la bordure */
  margin: 0 auto;           /* Centrage horizontal classique */
  margin-bottom: 24px;
}
```

**`box-sizing: border-box`** : par défaut, `width` et `height` ne comptent que le contenu. Le padding et la border s'ajoutent en plus, ce qui rend les calculs imprévisibles. `border-box` inclut padding et border dans la taille déclarée.

```css
/* Je mets ça en haut de tout fichier CSS */
*, *::before, *::after {
  box-sizing: border-box;
}
```

Avec `box-sizing: border-box`, un élément à `width: 300px` fait vraiment 300px total, padding inclus. Beaucoup plus intuitif.

### Les couleurs

Trois façons principales d'exprimer une couleur en CSS :

```css
/* Hexadécimal : le plus courant */
color: #1a1a2e;          /* #RRGGBB */
color: #fff;             /* Raccourci pour #ffffff */

/* RGB */
color: rgb(26, 26, 46);

/* RGBA : avec transparence (alpha de 0 à 1) */
color: rgba(26, 26, 46, 0.8);
background-color: rgba(0, 0, 0, 0.5);   /* Noir à 50% de transparence */

/* Noms de couleur */
color: red;
color: white;
color: transparent;
```

Propriétés de couleur :
```css
.element {
  color: #333;                        /* Couleur du texte */
  background-color: #f8f9fa;          /* Couleur de fond */
  border-color: #6c63ff;              /* Couleur de bordure */
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);  /* Ombre */
}
```

### Les pseudo-classes

Les pseudo-classes ciblent un élément dans un certain **état**.

```css
/* Au survol */
a:hover {
  color: #5a52e0;
}

/* Au clic (état actif) */
button:active {
  transform: scale(0.98);
}

/* Quand un champ a le focus (sélectionné) */
input:focus {
  border-color: #6c63ff;
  outline: none;
  box-shadow: 0 0 0 3px rgba(108, 99, 255, 0.2);
}

/* Lien déjà visité */
a:visited {
  color: #888;
}

/* Premier enfant d'un parent */
li:first-child {
  font-weight: bold;
}

/* Dernier enfant */
li:last-child {
  border-bottom: none;
}

/* Éléments pairs/impairs (pour les tableaux) */
tr:nth-child(even) {
  background-color: #f5f5f5;
}
```

### Positionnement CSS

```css
/* Défaut : flux normal du document */
.element {
  position: static;
}

/* Relatif à sa position normale */
.element {
  position: relative;
  top: 10px;
  left: 20px;
}

/* Absolu : par rapport à son ancêtre positionné le plus proche */
.parent {
  position: relative;   /* Le parent devient le référentiel */
}
.enfant {
  position: absolute;
  top: 0;
  right: 0;             /* Collé en haut à droite du parent */
}

/* Fixed : par rapport au viewport, reste en place au scroll */
.navbar-fixe {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 100;         /* Au-dessus de tout */
}

/* Sticky : reste visible quand on scrolle jusqu'à lui */
.header-sticky {
  position: sticky;
  top: 0;
}
```

---

## 7. Déployer avec Vercel

Vercel est une plateforme de déploiement qui permet de mettre un site en ligne gratuitement, directement depuis un repo GitHub. C'est rapide, gratuit pour les projets perso, et donne une URL publique immédiatement.

### Prérequis

* Node.js installé (pour utiliser Vercel en CLI)
* Git configuré
* Un compte GitHub
* Un compte Vercel (vercel.com, connexion avec GitHub)

### Installer Node.js

Node.js est un environnement d'exécution JavaScript côté serveur. Vercel CLI l'utilise pour fonctionner.

```bash
# Vérifier si Node est déjà installé
node --version
npm --version

# Si pas installé : aller sur nodejs.org et télécharger la version LTS
# Ou via nvm (Node Version Manager, recommandé) :
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install --lts
nvm use --lts
```

### Installer Vercel CLI

```bash
npm install -g vercel
```

Vérification :
```bash
vercel --version
```

### Configurer Git pour le projet

Si le projet n'est pas encore un repo Git :

```bash
# Initialiser un repo Git
git init

# Ajouter tous les fichiers
git add .

# Premier commit
git commit -m "Initial commit"

# Lier au repo GitHub (créer le repo sur GitHub d'abord)
git remote add origin https://github.com/mon-username/mon-repo.git
git push -u origin main
```

Si Git n'est pas configuré :
```bash
git config --global user.name "Pierre Dupont"
git config --global user.email "pierre@email.com"
```

### Déployer avec Vercel

**Méthode 1 : depuis le terminal (CLI)**

```bash
# Dans le dossier du projet
vercel

# Vercel pose quelques questions :
# Set up and deploy? → Yes
# Which scope? → Mon compte
# Link to existing project? → No (premier déploiement)
# Project name? → mon-portfolio
# In which directory is your code? → ./ (dossier actuel)

# Vercel génère une URL de prévisualisation
# Pour déployer en production :
vercel --prod
```

**Méthode 2 : depuis l'interface Vercel (plus simple)**

1. Aller sur vercel.com
2. Cliquer "Add New Project"
3. Importer le repo GitHub
4. Vercel détecte automatiquement le type de projet
5. Cliquer "Deploy"

C'est tout. Vercel génère une URL en `*.vercel.app` accessible immédiatement.

### Déploiements automatiques

Une fois le projet lié à GitHub, **chaque push sur la branche `main` déclenche un redéploiement automatique**. Mon workflow devient :

```bash
# Je fais des modifications
git add .
git commit -m "Mise à jour du portfolio"
git push origin main
# → Vercel redéploie automatiquement en quelques secondes
```

C'est ce qu'on appelle du **CI/CD** (Continuous Integration / Continuous Deployment). Je code, je push, c'est en ligne.

### Ajouter un domaine personnalisé

Par défaut j'ai une URL `mon-projet.vercel.app`. Pour un vrai domaine :

1. Dans le dashboard Vercel, aller dans Settings > Domains
2. Ajouter mon domaine (ex: `pierre-cyber.fr`)
3. Configurer les DNS chez mon registrar (OVH, Namecheap...) selon les instructions Vercel
4. Attendre la propagation DNS (quelques minutes à 48h)

**Note sécu sur le déploiement :**
* Ne jamais committer de credentials, tokens ou clés API dans le repo. Vercel gère les variables d'environnement dans son interface.
* Activer HTTPS est automatique sur Vercel (certificat Let's Encrypt gratuit).
* Les logs de déploiement sont visibles dans le dashboard Vercel : utile pour débugger.

---

## 8. Cheatsheet CSS

### Sélecteurs

```css
div           /* Toutes les divs */
#mon-id       /* L'élément avec id="mon-id" */
.ma-classe    /* Tous les éléments avec class="ma-classe" */
div p         /* Tous les <p> dans un <div> */
div > p       /* Les <p> enfants directs d'un <div> */
h1, h2, h3    /* h1 ET h2 ET h3 */
.card.active  /* Un élément avec les deux classes */
a:hover       /* Lien au survol */
input:focus   /* Input sélectionné */
```

### Box model

```css
width: 300px;
height: 200px;
padding: 16px 24px;
margin: 0 auto;
border: 2px solid #333;
border-radius: 8px;
box-sizing: border-box;
```

### Typographie

```css
font-family: 'Inter', sans-serif;
font-size: 1rem;
font-weight: 700;
line-height: 1.6;
color: #333;
text-align: center;
text-decoration: none;
letter-spacing: 0.5px;
```

### Couleurs et fond

```css
color: #1a1a2e;
background-color: #f8f9fa;
background-color: rgba(0, 0, 0, 0.5);
box-shadow: 0 4px 20px rgba(0,0,0,0.1);
```

### Unités

| Unité | Usage |
|-------|-------|
| `px` | Taille fixe |
| `rem` | Taille de texte |
| `%` | Largeur relative au parent |
| `vh` | Hauteur relative au viewport |
| `vw` | Largeur relative au viewport |

### Méthodes d'application CSS (priorité croissante)

```
External stylesheet < Embedded <style> < Inline style="..." < !important
```

### Déploiement Vercel en 3 commandes

```bash
git add . && git commit -m "update" && git push origin main
# → Vercel redéploie automatiquement
```
