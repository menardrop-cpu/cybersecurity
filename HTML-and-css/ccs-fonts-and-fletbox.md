# CSS : Fonts personnalisées & Flexbox

> Mes notes sur les deux outils CSS qui changent vraiment la gueule d'une page : les polices custom et Flexbox. Flexbox en particulier c'est ce qui me permet de mettre des éléments côte à côte sans me battre avec le CSS. Je vais l'utiliser partout.

## Sommaire

1. [Google Fonts : importer des polices personnalisées](#1-google-fonts--importer-des-polices-personnalisées)
2. [Appliquer les fonts en CSS](#2-appliquer-les-fonts-en-css)
3. [Flexbox : c'est quoi](#3-flexbox--cest-quoi)
4. [Comment utiliser Flexbox](#4-comment-utiliser-flexbox)
5. [Propriétés Flexbox de base](#5-propriétés-flexbox-de-base)
6. [Propriétés avancées](#6-propriétés-avancées)
7. [Flexbox sur la navbar](#7-flexbox-sur-la-navbar)
8. [Flexbox sur une image de profil](#8-flexbox-sur-une-image-de-profil)
9. [Cheatsheet Flexbox](#9-cheatsheet-flexbox)

---

## 1. Google Fonts : importer des polices personnalisées

### Le problème des polices système

Par défaut, une page web utilise les polices installées sur l'ordinateur de l'utilisateur. `Arial`, `Times New Roman`, `Georgia`... Ces polices varient d'un OS à l'autre et donnent un rendu générique et daté.

Pour avoir une police précise et cohérente sur tous les navigateurs et tous les appareils, j'utilise des **web fonts** : des polices hébergées sur un serveur, chargées à l'affichage de la page.

### Google Fonts

[Google Fonts](https://fonts.google.com) est un catalogue gratuit de polices open source. C'est la solution la plus simple pour intégrer une police custom.

**Comment ça marche :**

1. Je vais sur fonts.google.com
2. Je choisis une police (par exemple `Inter` ou `Roboto`)
3. Je sélectionne les graisses dont j'ai besoin (Regular 400, Bold 700...)
4. Google me génère un code d'import à coller dans mon HTML ou mon CSS

### Intégrer Google Fonts dans le HTML

La méthode recommandée : coller le lien fourni par Google dans le `<head>`, **avant** le lien vers mon fichier CSS.

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mon site</title>

  <!-- Google Fonts : toujours avant mon CSS -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet">

  <!-- Mon CSS après -->
  <link rel="stylesheet" href="style.css">
</head>
```

**Décodage du lien Google Fonts :**

```
https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap
```

* `family=Inter` : la police demandée s'appelle Inter
* `wght@400;700` : je charge le grammage 400 (regular) et 700 (bold)
* `display=swap` : pendant le chargement de la font, le navigateur affiche une font système. Quand la custom est prête, il swipe. Évite le flash de texte invisible.

**Les deux `<link rel="preconnect">`** : ils indiquent au navigateur d'établir la connexion avec les serveurs Google à l'avance, avant même que la requête de font soit faite. Ça accélère le chargement.

### Intégrer Google Fonts directement en CSS

Autre option : importer directement dans le fichier CSS avec `@import`. Plus simple à maintenir, légèrement plus lent à charger.

```css
/* En haut du fichier style.css, avant toute autre règle */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');
```

Je préfère la méthode HTML pour la performance, mais les deux fonctionnent.

**Note sécu** : charger Google Fonts, c'est faire une requête vers les serveurs de Google à chaque visite de ma page. Google peut logger les IPs des visiteurs. Pour un portfolio perso c'est négligeable, mais pour un site qui veut protéger la vie privée de ses utilisateurs, il vaut mieux héberger les fonts soi-même. À garder en tête.

---

## 2. Appliquer les fonts en CSS

Une fois la font importée, je l'applique avec la propriété `font-family`.

### Syntaxe de base

```css
body {
  font-family: 'Inter', sans-serif;
}
```

Le deuxième argument (`sans-serif`) est la **font de fallback** : si Inter ne charge pas, le navigateur utilise la police sans-serif par défaut du système. Toujours mettre un fallback.

### Appliquer sur des éléments spécifiques

Je peux combiner plusieurs polices sur une même page : une pour les titres, une autre pour le corps de texte.

```css
/* Police de texte : lisibilité */
body {
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  font-weight: 400;
  line-height: 1.6;
}

/* Police de titres : caractère */
h1, h2, h3 {
  font-family: 'Playfair Display', serif;
  font-weight: 700;
}
```

### Propriétés CSS liées aux fonts

```css
p {
  font-family: 'Inter', sans-serif;  /* Quelle police */
  font-size: 16px;                   /* Taille */
  font-weight: 400;                  /* Graisse : 400 = normal, 700 = bold */
  font-style: italic;                /* Normal ou italic */
  line-height: 1.6;                  /* Interligne (1.6 = 1.6x la taille de font) */
  letter-spacing: 0.5px;            /* Espacement entre les lettres */
  text-align: center;               /* Alignement : left, right, center, justify */
  color: #333333;                   /* Couleur du texte */
  text-transform: uppercase;        /* uppercase, lowercase, capitalize */
  text-decoration: underline;       /* underline, none (pour enlever le soulignement des liens) */
}
```

### Exemple complet : styles typographiques d'une page

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&family=Space+Grotesk:wght@500;700&display=swap');

body {
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  font-weight: 400;
  line-height: 1.7;
  color: #1a1a2e;
}

h1 {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 56px;
  font-weight: 700;
  line-height: 1.1;
}

h2 {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 36px;
  font-weight: 700;
}

h3 {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 24px;
  font-weight: 500;
}

a {
  color: #6c63ff;
  text-decoration: none;  /* Enlève le soulignement par défaut */
}

a:hover {
  text-decoration: underline;
}
```

---

## 3. Flexbox : c'est quoi

### Le problème que Flexbox résout

Par défaut en HTML/CSS, les éléments `<div>` et les balises de bloc se placent les uns **sous** les autres. Pour les mettre **côte à côte** (une navbar horizontale, des cards en ligne, deux colonnes...), il fallait historiquement des hacks avec `float` ou `display: inline-block`. C'était pénible, fragile, et ça rendait tout le monde fou.

**Flexbox** (*Flexible Box Layout*) est un système de mise en page CSS conçu pour distribuer et aligner des éléments dans un conteneur, sur un axe (horizontal ou vertical), de façon prévisible et contrôlable.

Avec Flexbox :
* Je mets des éléments côte à côte en une ligne
* Je les aligne verticalement et horizontalement facilement
* Ils s'adaptent automatiquement à l'espace disponible
* Le responsive devient simple

### Le modèle mental : conteneur et enfants

Flexbox fonctionne sur deux niveaux :

* Le **conteneur flex** (*flex container*) : l'élément parent sur lequel j'applique `display: flex`
* Les **éléments flex** (*flex items*) : les enfants directs du conteneur, qui obéissent aux règles flex

```html
<div class="container">   <!-- Flex container -->
  <div class="item">A</div>  <!-- Flex item -->
  <div class="item">B</div>  <!-- Flex item -->
  <div class="item">C</div>  <!-- Flex item -->
</div>
```

```css
.container {
  display: flex;  /* Déclenche Flexbox sur ce conteneur */
}
```

Résultat : A, B, C s'affichent côte à côte au lieu de les uns sous les autres.

### Les deux axes

Flexbox raisonne sur deux axes :

* **L'axe principal** (*main axis*) : la direction dans laquelle les items se disposent. Par défaut : horizontal (gauche à droite).
* **L'axe croisé** (*cross axis*) : perpendiculaire à l'axe principal. Par défaut : vertical.

Toutes les propriétés Flexbox font référence à ces deux axes. C'est le concept central à avoir en tête.

```
Axe principal (main axis) →
+-------+-------+-------+
|       |       |       |  ↑
|   A   |   B   |   C   |  Axe croisé
|       |       |       |  ↓
+-------+-------+-------+
```

---

## 4. Comment utiliser Flexbox

### Activer Flexbox

Une seule ligne CSS sur le conteneur :

```css
.container {
  display: flex;
}
```

Tous les enfants directs deviennent automatiquement des flex items. Les petits-enfants ne sont pas affectés.

### Sans Flexbox vs avec Flexbox

**HTML :**
```html
<div class="cards-container">
  <div class="card">Card 1</div>
  <div class="card">Card 2</div>
  <div class="card">Card 3</div>
</div>
```

**Sans Flexbox** : les trois cards s'empilent verticalement.

**Avec Flexbox** :
```css
.cards-container {
  display: flex;
}
```
Les trois cards s'affichent côte à côte.

---

## 5. Propriétés Flexbox de base

### `flex-direction` : orientation de l'axe principal

```css
.container {
  display: flex;
  flex-direction: row;            /* Défaut : horizontal, gauche à droite */
  flex-direction: row-reverse;    /* Horizontal, droite à gauche */
  flex-direction: column;         /* Vertical, haut en bas */
  flex-direction: column-reverse; /* Vertical, bas en haut */
}
```

`column` est utile pour centrer verticalement dans un conteneur en hauteur fixe.

### `justify-content` : alignement sur l'axe principal

Contrôle comment les items sont distribués **dans la direction de l'axe principal**.

```css
.container {
  display: flex;
  justify-content: flex-start;    /* Défaut : items au début */
  justify-content: flex-end;      /* Items à la fin */
  justify-content: center;        /* Items centrés */
  justify-content: space-between; /* Espaces égaux entre les items, bords collés */
  justify-content: space-around;  /* Espaces égaux autour des items */
  justify-content: space-evenly;  /* Espaces parfaitement égaux partout */
}
```

Celui que j'utilise le plus : `space-between` pour les navbars (logo à gauche, menu à droite), et `center` pour centrer.

### `align-items` : alignement sur l'axe croisé

Contrôle comment les items sont alignés **perpendiculairement** à l'axe principal.

```css
.container {
  display: flex;
  align-items: stretch;     /* Défaut : les items s'étirent pour remplir la hauteur */
  align-items: flex-start;  /* Alignés en haut */
  align-items: flex-end;    /* Alignés en bas */
  align-items: center;      /* Centrés verticalement */
  align-items: baseline;    /* Alignés sur la ligne de base du texte */
}
```

`align-items: center` est la propriété que j'attendais depuis toujours pour centrer verticalement un élément. Ça marche en une ligne.

### `gap` : espacement entre les items

```css
.container {
  display: flex;
  gap: 20px;          /* Même espace entre tous les items */
  gap: 20px 40px;     /* row-gap column-gap */
  row-gap: 20px;      /* Espace vertical seulement */
  column-gap: 40px;   /* Espace horizontal seulement */
}
```

`gap` remplace les vieilles techniques avec `margin`. Beaucoup plus propre.

### `flex-wrap` : retour à la ligne

Par défaut, Flexbox essaie de tout faire tenir sur une ligne et compresse les items si nécessaire. `flex-wrap` autorise le retour à la ligne.

```css
.container {
  display: flex;
  flex-wrap: nowrap;   /* Défaut : tout sur une ligne, compression si besoin */
  flex-wrap: wrap;     /* Retour à la ligne si ça déborde */
  flex-wrap: wrap-reverse; /* Retour à la ligne, lignes inversées */
}
```

`flex-wrap: wrap` est essentiel pour le responsive : les cards qui tiennent en 3 colonnes sur desktop passent à 1 colonne sur mobile automatiquement.

---

## 6. Propriétés avancées

Ces propriétés s'appliquent sur les **flex items** (les enfants), pas sur le conteneur.

### `flex` : contrôler la taille des items

La propriété `flex` est un raccourci pour trois propriétés : `flex-grow`, `flex-shrink`, `flex-basis`.

```css
.item {
  flex: 1;        /* Raccourci : flex-grow: 1, flex-shrink: 1, flex-basis: 0 */
  flex: 0 0 200px; /* Taille fixe de 200px, ne grandit pas, ne rétrécit pas */
  flex: 2;        /* Prend deux fois plus de place que flex: 1 */
}
```

**`flex: 1` sur tous les items** : ils se partagent l'espace disponible de façon égale.

```css
.cards-container {
  display: flex;
  gap: 24px;
}

.card {
  flex: 1;  /* Chaque card prend un tiers de l'espace */
}
```

### `align-self` : exception pour un item individuel

Overrides `align-items` pour un seul item.

```css
.container {
  display: flex;
  align-items: center;  /* Tous les items centrés */
}

.item-special {
  align-self: flex-end;  /* Cet item spécifique en bas */
}
```

### `order` : changer l'ordre d'affichage

Modifie l'ordre visuel sans toucher au HTML. Par défaut tous les items ont `order: 0`.

```css
.item-a { order: 2; }
.item-b { order: 1; }
.item-c { order: 3; }
/* Affichage : B, A, C */
```

---

## 7. Flexbox sur la navbar

La navbar est le cas d'usage classique de Flexbox. Je veux le logo à gauche et les liens à droite, le tout centré verticalement.

**HTML :**
```html
<nav class="navbar">
  <div class="nav-logo">
    <a href="index.html">Pierre</a>
  </div>
  <ul class="nav-links">
    <li><a href="#accueil">Accueil</a></li>
    <li><a href="#projets">Projets</a></li>
    <li><a href="#contact">Contact</a></li>
  </ul>
</nav>
```

**CSS :**
```css
.navbar {
  display: flex;
  justify-content: space-between;  /* Logo à gauche, liens à droite */
  align-items: center;             /* Centré verticalement */
  padding: 20px 40px;
  background-color: #1a1a2e;
}

.nav-logo a {
  font-size: 24px;
  font-weight: 700;
  color: #ffffff;
  text-decoration: none;
}

/* Les liens : aussi un flex container pour les mettre côte à côte */
.nav-links {
  display: flex;
  gap: 32px;
  list-style: none;     /* Enlève les puces de la liste */
  margin: 0;
  padding: 0;
}

.nav-links a {
  color: #cccccc;
  text-decoration: none;
  font-size: 16px;
  transition: color 0.2s;
}

.nav-links a:hover {
  color: #ffffff;
}
```

**Ce que je retiens de cet exemple :**
* `justify-content: space-between` sur la navbar pousse le logo et les liens aux extrémités
* `align-items: center` les aligne verticalement sans avoir à calculer de margin
* `list-style: none` + `margin: 0; padding: 0;` sont le reset classique pour utiliser une `<ul>` comme conteneur flex sans les styles par défaut du navigateur
* Flexbox peut s'imbriquer : la navbar est flex, et la `<ul>` à l'intérieur est aussi flex

---

## 8. Flexbox sur une image de profil

Cas concret : afficher une photo de profil avec un texte à côté, les deux centrés verticalement.

**HTML :**
```html
<section class="profil">
  <div class="profil-image">
    <img src="images/profil.jpg" alt="Photo de profil de Pierre">
  </div>
  <div class="profil-texte">
    <h1>Pierre Dupont</h1>
    <p>Analyste cybersécurité en formation · GRC & Consulting</p>
    <a href="#contact" class="btn">Me contacter</a>
  </div>
</section>
```

**CSS :**
```css
.profil {
  display: flex;
  align-items: center;   /* Image et texte alignés verticalement au centre */
  gap: 48px;
  padding: 80px 40px;
}

.profil-image img {
  width: 200px;
  height: 200px;
  border-radius: 50%;    /* Rend l'image circulaire */
  object-fit: cover;     /* Recadre l'image pour remplir le cercle sans déformer */
}

.profil-texte {
  display: flex;
  flex-direction: column;  /* Titre, texte et bouton empilés verticalement */
  gap: 16px;
}

.profil-texte h1 {
  font-size: 48px;
  font-weight: 700;
  margin: 0;
}

.profil-texte p {
  font-size: 18px;
  color: #666;
  margin: 0;
}

.btn {
  display: inline-block;
  padding: 12px 28px;
  background-color: #6c63ff;
  color: #ffffff;
  text-decoration: none;
  border-radius: 8px;
  font-weight: 700;
  align-self: flex-start;  /* Le bouton ne s'étire pas sur toute la largeur */
}
```

**Ce que je retiens :**
* `flex-direction: row` (défaut) pour mettre image et texte côte à côte
* `flex-direction: column` sur le conteneur texte pour empiler titre/description/bouton
* `border-radius: 50%` + `object-fit: cover` = image circulaire parfaite
* `align-self: flex-start` sur le bouton pour qu'il garde sa taille naturelle au lieu de s'étirer

---

## 9. Cheatsheet Flexbox

### Sur le conteneur (parent)

| Propriété | Valeurs | Effet |
|-----------|---------|-------|
| `display: flex` | | Active Flexbox |
| `flex-direction` | `row` `column` `row-reverse` `column-reverse` | Direction de l'axe principal |
| `justify-content` | `flex-start` `flex-end` `center` `space-between` `space-around` `space-evenly` | Alignement sur l'axe principal |
| `align-items` | `stretch` `flex-start` `flex-end` `center` `baseline` | Alignement sur l'axe croisé |
| `flex-wrap` | `nowrap` `wrap` `wrap-reverse` | Retour à la ligne |
| `gap` | `20px` `20px 40px` | Espacement entre items |

### Sur les items (enfants)

| Propriété | Valeurs | Effet |
|-----------|---------|-------|
| `flex` | `1` `0 0 200px` | Taille flexible |
| `align-self` | Mêmes que `align-items` | Override d'alignement individuel |
| `order` | Nombre entier | Ordre d'affichage |

### Combos que j'utilise tout le temps

```css
/* Centrer horizontalement et verticalement */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* Navbar : logo gauche, liens droite */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* Cards en ligne qui s'adaptent */
.cards {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;
}
.card {
  flex: 1;
  min-width: 280px;  /* Empêche les cards d'être trop petites */
}

/* Deux colonnes égales */
.deux-colonnes {
  display: flex;
  gap: 40px;
}
.colonne {
  flex: 1;
}
```

---

## Pour aller plus loin

* **Flexbox Froggy** (flexboxfroggy.com) : jeu interactif pour pratiquer toutes les propriétés Flexbox. Je le fais en entier, ça prend 20 minutes et ça fixe vraiment les concepts.
* **CSS Grid** : le complément de Flexbox pour les mises en page à deux dimensions (lignes ET colonnes). À voir après Flexbox.
* Dans les DevTools du navigateur (Clic droit > Inspecter), je peux voir les guides Flexbox en survolant un conteneur flex dans le panneau Elements. Indispensable pour debugger.
