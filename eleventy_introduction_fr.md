# Eleventy (11ty) par Zach Leatherman

## 1. Introduction

[Eleventy](https://www.11ty.io) est un static site generator (SSG) créé et maintenu par [Zach Leatherman](https://www.zachleat.com/). Comme tel, il n'utilise pas de base de données. Cet outil vous permet de développer des sites basés sur des templates (codés avec [Nunjucks](https://mozilla.github.io/nunjucks/) dans notre cas) et des fichiers de données (YAML / Markdown / HTML / JSON / JS) présents dans un dossier source. Sur base de ces fichiers, Eleventy va générer un site entièrement statique dans un dossier de destination. Vous pourrez ensuite déployer ce site sur n'importe quel serveur web.

[Le but avoué d'Eleventy](https://www.11ty.io/docs/) est d'être une alternative à [Jekyll](https://jekyllrb.com/) écrite en JavaScript plutôt qu'en Ruby. Tout comme Jekyll, c'est un SSG simple à utiliser et à configurer une fois les principes de base bien compris.

Node étant assez rapide, Eleventy est un SSG performant. Il est également très flexible. Tour d'abord, Eleventy est écrit en Node et vous permet donc d'utiliser facilement l'ensemble de l'écosystème NPM pour en étendre les fonctionnalités. Il vous permet également d'utiliser une large gamme de langages de templating. Nous utiliserons [Nunjucks](https://mozilla.github.io/nunjucks/) de Mozilla.

## 2. Installation and configuration

### Installation

Commençons d'abord par créer un dossier pour notre projet

```text
mkdir eleventy-portfolio
cd eleventy-portfolio
```

L'installation se fait à l'aide d'un module NPM. Pour installer Eleventy, il faut commencer par créer dans notre projet un fichier `package.json` qui reprendra toutes nos dépendences Node.

```text
npm init
```

Dans un premier temps, vous pouvez simplement accepter l'ensemble des réponses aux questions posées. Vous pourrez toujours y revenir par après en éditant votre fichier `package.json`. Voici la [documentation complète](https://docs.npmjs.com/files/package.json) pour vous aider à comprendre quelles sont les paramètres et options disponibles.

Nous pouvons ensuite installer Eleventy comme dépendance locale pour notre projet:

```text
npm install --save-dev @11ty/eleventy
```

Eleventy va simplement installer un dossier `node_modules` mais rien d'autre. Pour pouvoir utiliser Eleventy, nous allons devoir le configurer et créer au moins un fichier. Pour le moment, nous allons simplement créer un fichier `index.html` statique qu'Eleventy devrait copier tel quel dans le dossier de destination qui est spécifié comme étant `_site` par défaut.

Une fois ce fichier créé, nous pouvons tester les commandes principales d'Eleventy:

- `npx eleventy`: pour faire tourner Eleventy
- `npx eleventy --serve`: pour faire tourner Browsersync afin d'avoir un serveur web local qui va reloader le site dans votre browser dès que le site change
- `npx eleventy --help`: pour avoir la liste des commandes existantes

Vous connaissez maintenant les commandes de base, nécessaires pour commencer à travailler. Si vous tapez `npx eleventy`, Eleventy devrait créer pour vous un dossier `_site` et y placer une copie de votre fichier `index.html`.

Voyons maintenant comment configurer Eleventy en fonction de nos besoins.

### Configuration

Nous allons commencer par créer une architecture de projet et configurer Eleventy grâce au fichier de configuration `.eleventy.js`:

- Supprimer le dossier `./site`
- Créer un dossier `./src` et y placer notre fichier `index.html`
- Créer un fichier `.eleventy.js` dans la racine de notre projet

Commençons par spécifier les dossiers source et de destination pour Eleventy:

**.eleventy.js**
```js
module.exports = function(eleventyConfig) {
  // override default config
  return {
    dir: {
      input: "src",
      output: "dist"
    }
  };
};
```

Si nous exécutons la commande `npx eleventy` dans notre terminal, Eleventy générera un dossier `./dist` pour y copier notre fichier `index.html`.

### Copier quelques fichiers tels quels

Nous pouvons également utiliser ce fichier de configuration pour demander à Eleventy de copier n'importe quel fichier ou dossier depuis le dossier source jusqu'au dossier de destination via [`addPassthroughCopy`](https://www.11ty.io/docs/copy/). De bons candidats sont les assets statiques de notre projet comme les images et les fichiers de polices.

- créer un dossier `./src/assets/img/` et y placer un fichier image
- créer un dossier `./src/assets/fonts/` et y placer quelques fichiers de polices
- créer un dossier `./src/assets/js/` et y placer un fichier JavaScript
- créer un dossier `./src/assets/css/` et y placer un fichier CSS

Modifier le fichier `.eleventy.js` comme suit:

**.eleventy.js**
```js
module.exports = function(eleventyConfig) {
  // copy files
  eleventyConfig.addPassthroughCopy("./src/assets/");

  // override default config
  return {
    dir: {
      input: "src",
      output: "dist"
    }
  };
};
```

Eleventy va maintenant copier ce dossier ainsi que tout ce qu'il contient.

### Ignorer certains dossiers et fichiers

Par défaut, Eleventy va ignorer le dossier `node_modules` ainsi que les dossiers, fichiers et globs spécifiés dans notre éventuel fichier `.gitignore`.

Nous pouvons également créer un fichier `.eleventyignore` et spécifier un dossier, fichier ou glob par ligne pour dire à Eleventy de les ignorer dans notre dossier source.

**.eleventyignore**
```txt
./node_modules/
./dist/
```

### Assets pipeline et outils de build

Eleventy ne possède pas d'assets pipeline ou d'outil de build par défaut. Il est donc intéressant d'intégrer Eleventy à un outil de build, que ce soit des scripts NPM, Gulp, Webpack ou toute autre alternative.

Lorsque vous commencez à utiliser des outils de build pour vos assets, vous devrez modifier votre configuration de `addPassthroughCopy` et probablement ignorer les dossiers d'assets qui ne dépendent plus d'Eleventy puisque ce sont alors vos outils et scripts de build qui vont les générer dans votre dossier `dist`.

A titre d'exemple, si un script NPM génère notre fichier CSS à partir de fichiers Sass, il suffit de faire les modifications suivantes:

```js
module.exports = function(eleventyConfig) {
  // copy files
  eleventyConfig.addPassthroughCopy("./src/assets/js/");
  eleventyConfig.addPassthroughCopy("./src/assets/fonts/");
  eleventyConfig.addPassthroughCopy("./src/assets/img/");

  // override default config
  return {
    dir: {
      input: "src",
      output: "dist"
    }
  };
};
```

**.eleventyignore**
```txt
./node_modules/
./dist/
./src/assets/scss/
```

De cette façon, Eleventy va complètement ignorer le dossier `./src/assets/scss/` qui sera entièrement géré par l'outil de build, ainsi que son output.

Personellement, j'utilise Gulp en combinaison avec Webpack dans la plupart de mes projets et Eleventy est très facile à intégrer à ce genre de workflow.

## 3. Définir et structurer vos données

Eleventy permet de travailler avec deux grandes sources de données:

1. **des fichiers Markdown** (pour le contenu principal) et YAML front matter (pour le reste de la data structure) qui peuvent être facilement convertis en collections.
2. **des fichiers JSON et/ou JS** qui peuvent soit être statiques soit dynamiques (provenant d'une API).

Ces deux sources de données ne sont pas mutuellement exclusives et sont généralement utilisées simultanément dans tout projet. Voyons cela plus en détail.

### Collections

#### Markdown et YAML front matter

Les fichiers Markdown couplés à un YAML front matter permettent d'utiliser de simples fichiers textes comme source de données structurées. C'est un classique avec la plupart des SSG.

La partie en Markdown représente le contenu principal de vos données et est généralement simplement convertie en HTML. Le YAML front matter permet de créer votre data structure à l'aide de différents types de données (strings, tableaux, objets, etc.).

Si vous devez construire un blog, vos blogposts seront représentés par des fichiers Markdown avec un YAML front matter qui pourrait ressembler à ceci:

**./src/blog/2019-07-22-markdown-yaml-front-matter.md**

```md
---
title: "This is the title"
intro: "This is an introductory paragraph for a blogpost"
imageSmall: "testimage_600.jpg"
imageMedium: "testimage_1024.jpg"
imageBig: "testimage_1500.jpg"
imageAlt: "Alternative text for picture"
categories:
- front-end
- JAMstack
- Eleventy
---

## Level 2 title

Lorem ipsum dolor sit amet, consectetur adipisicing elit. Recusandae voluptatibus reiciendis dignissimos accusantium illo, voluptates consequuntur fugit amet quo sed nisi facere animi incidunt assumenda exercitationem, nam omnis perspiciatis praesentium.
```

Chaque membre de l'équipe éditoriale pourrait être représenté par un fichier ayant la structure suivante:

**./src/projects/jerome-coupe.md**

```md
---
name: "Jérôme"
surname: "Coupé"
image: "jerome_coupe.jpg"
twitter: "https://twitter.com/jeromecoupe"
github: "https://github.com/jeromecoupe"
website: "https://www.webstoemp.com"
---

Jérôme Coupé is a looney front-end designer and teacher from Brussels, Belgium. When not coding, he might have been seen drinking a few craft beers with friends.
```

#### Collection API

Pour qu'Eleventy groupe tous ces fichiers dans un tableau et vous permette de les manipuler dans vos templates, il suffit les déclarer comme faisant partie d'une [une collection](https://www.11ty.io/docs/collections/). N'importe quel élément de contenu peut faire partie d'une ou de plusieurs collections.

Pour créer une collection, vous pouvez assigner le même `tag` à différents éléments de contenu. Personnellement, je préfère utiliser l'API de collections et le fichier `eleventy.js`.

Cette API vous offre [différentes méthodes pour déclarer des collections](https://www.11ty.io/docs/collections/#collection-api-methods) qui ont chacune leur utilité. Celle que j'utilise personnellement le plus est `getFilteredByGlob(glob)` qui vous permet de grouper dans une collection tous les éléments de contenus qui correspondent au même glob pattern.

Si tous vos fichiers Markdown sont placés dans un dossier `./src/blog/`, créer une collection les rassemblant tous est assez simple. Il vous suffit d'ajouter le code suivant dans votre fichier `eleventy.js`

```js
module.exports = function(eleventyConfig) {
  // blogposts collection
  eleventyConfig.addCollection("blogposts", function(collection) {
    return collection.getFilteredByGlob("./src/blog/*.md");
  });

  // team collection
  eleventyConfig.addCollection("team", function(collection) {
    return collection.getFilteredByGlob("./src/team/*.md");
  });

  // override default config
  return {
    dir: {
      input: "src",
      output: "dist"
    }
  };
};
```

Vous pouvez maintenant accéder à vos collections en utilisant `collections.blogposts` et `collections.team` dans vos templates. Nous y reviendrons dans le chapitre consacrée au templating.

Il me reste à signaler qu'Eleventy créé par défaut une collection contenant tous vos élements de contenus, c'est à dire tous les fichiers gérés par Eleventy. Cette collection spéciale est adressable via `colections.all`.

Lorsqu'une collection est créée, les key suivantes sont automatiquement créées. Les keys créées dans votre front matter sont accessibles via les keys suivantes:

- `inputPath`: le chemin complet vers le fichier source (inclus le chemin vers le dossier d'input d'Eleventy)
- `fileSlug`: une transformation en slug du nom de fichier du fichier source. Utile dans la construction de permalinks. En savoir plus sur `fileslug` [dans la documentation](https://www.11ty.io/docs/data/#fileslug).
- `outputPath`: le chemin complet vers le fichier d'output de cet élément de contenu
- `url`: l'URL utilisée pour lier vers un élément de la collection. En général cette valeur est basée sur celle de la key `permalink`
- `date`: la date utilisée pour le classement. Pour en savoir plus sur les [dates des éléments de contenus](https://www.11ty.io/docs/dates/), référez-vous à la documentation.
- `data`: toutes les données pour cet élément de contenu. Se réfère aux champs du YAML front-matter et aux données héritée des layouts.
- `templateContent`: le contenu de ce template une fois rendu par Eleventy. N'inclus pas les templates étendus.

#### Classer et filtrer vos collections

Lorsque vous créez une collection avec l'API d'Eleventy, les items de cette collection sont automatiquement classés en ordre ascendant en utilisant:

1. La date renseignée dans le nom de fichier ou dans le YAML front matter du fichier source ou, a defaut, la date de création de celui-ci.
2. Si certains fichiers source ont une date identique, le chemin complet (y compris le nom de fichier) est pris en compte

Si un classement par date correspond à ce que vous souhaitez, vous pouvez éventuellement inverser celui-ci en utilisant le filtre `reverse` de Nunjucks.

Si vous souhaitez classer alphabétiquement les membres de votre équipe sur base de la key `surname`, le code suivant classerait ces membres de l'équipe par ordre ascendant.


```js
module.exports = function(eleventyConfig) {

  // ... more configuration here .../

  // Team collection
  eleventyConfig.addCollection("team", function(collection) {
    return collection.getFilteredByGlob("./src/team/*.md").sort((a, b) => {
      let nameA = a.data.surname.toUpperCase();
      let nameB = b.data.surname.toUpperCase();
      if (nameA < nameB) return -1;
      if (nameA > nameB) return 1;
      return 0;
    });
  });
};
```

Si vous devez par contre filtrer une collection pour exclure certaines données, il vous faudra utiliser la méthode `filter` en JavaScript. Vous pouvez par exemple inclure seulement les blogposts ayant une key `draft` dont la valeur n'est pas `true` ou qui ont une date de publication inférieure à la date de génération du site.

```js
const now = new Date();

module.exports = function(eleventyConfig) {

  // ... more configuration here .../

  // blogposts collection
  eleventyConfig.addCollection("blogposts", function(collection) {
  return collection.getFilteredByGlob("./src/blogposts/*.md").filter((item) => {
    return item.data.draft !== true && item.date <= now;
  });
};
```

### Data files (JS ou JSON)

Outre les collections, l'autre grande source de données pour Eleventy sont les fichiers de data. Ceux-ci peuvent être statiques ou dynamiques.

Ces fichiers doivent par défaut être stockés dans le dossier `./src/_data/`. Cet emplacement des fichiers data peut être modifié dans votre fichier de configuration `.eleventy.js`.

```js
module.exports = function(eleventyConfig) {

  // ... more configuration here .../

  // override default config
  return {
    dir: {
      input: "src",
      output: "dist",
      data: "_data"
    }
  };
};
```

#### Fichiers data statiques

Les fichiers de data statiques sont simplement des fichiers JSON ou JS contenant des paires key / value.

**./src/_data/site.js**
```js
module.exports = {
  title: "Title of the site",
  description: "Description of the site",
  url: "https://www.mydomain.com",
  baseUrl: "/",
  author: "Name Surname",
  authorTwitterUrl: "https://twitter.com/jeromecoupe",
  authorTwitterHandle: "@twitterhandle",
  buildTime: new Date()
};
```

Les données sont accessibles dans vos templates à l'aide du nom de fichier utilisé comme key. Par exemples les données contenues dans le fichier `./src/_data/site.js` sont accessibles dans vos templates via la variable `site`.

#### Fichiers data dynamiques

Etant donné que les fichiers de data sont rédigés en JavaScript, rien ne vous empèche de [vous connecter à une API dans l'un de ces fichiers](https://www.webstoemp.com/blog/headless-cms-graphql-api-eleventy/) en utilisant [`node-fetch`](https://www.npmjs.com/package/node-fetch) ou [`axios`](https://www.npmjs.com/package/axios) par exemple.

A chaque fois que votre site est généré, Eleventy exécutera ce script et traitera le JSON retourné par l'API comme un fichier de données statique pour générer vos pages.

### Permalinks et URLs

Par défaut, Eleventy utilise votre structure de dossiers et vos fichiers et dans votre directory source pour générer des fichiers statiques dans votre dossier de sortie.

- `./src/index.html` va générer `./dist/index.html` avec comme URL `/`
- `./src/test.html` va générer `./dist/test/index.html` avec comme URL `/test/`
- `./src/subdir/index.html` va générer `./dist/subdir/index.html` avec comme URL `/subdir/`

Cela peut être surdéterminé par l'utilisation d'une variable `permalink` statique ou dynamique dans vos fichiers de contenus ou dans vos templates.

Pour créer un blog, vous aller avoir besoin d'une page d'index `/blog/index.html`. Votre template Nunjucks `./src/pages/blog.njk` pourra donc avoir cette URL comme valeur de la key `permalink` dans son YAML front matter.

```text
---
permalink: "/blog/index.html"
---
```

Vous pouvez également utiliser des variables pour créer vos valeurs de `permalink`.

```text
---
permalink: "/blog/{{ page.fileSlug }}/index.html"
---
```

Si vous avez une collection pour présenter votre équipe mais que vous n'avez pas de page de détail, vous pouvez simplement utiliser la valeur `false` pour `permalink` et Eleventy ne générera alors pas de pages de détail. Dans la plupart des cas, vous n'aurez alors pas besoin de layout non plus.

```text
---
permalink: false
layout: false
---
```

Nous verrons plus loin que vous aurez alors besoin d'utiliser `templateContent` pour afficher le contenu de vos fichiers Markdown.

#### Valeurs par défaut et fichiers de données liés aux dossiers

Plutôt que de spécifier une valeur YAML front matter identique dans tous les fichiers d'une collection, Eleventy vous offre la possibilité de spécifier des valeurs identiques pour tous les fichiers contenus dans un directory en utilisant des [directory data files](https://www.11ty.io/docs/data-template-dir/) en JS ou en JSON.

Si vous devez par exemple spécifier une valeur pour `layout` et `permalink` identiques pour tous vos blogposts, vous pouvez simplement les spécifier dans un fichier `.src/blogposts/blogposts.json`, `.src/blogposts/blogposts.11data.json` ou `.src/blogposts/blogposts.11data.js`. Eleventy appliquera ces valeurs à tous les fichiers du directory ou des directories enfants.

**./src/blogposts/blogposts.json** ou **./src/blogposts/blogposts.11tydata.json**
```json
{
  "layout": "layouts/blogpost.njk",
  "permalink": "blog/{{ page.fileSlug }}/index.html"
}
```

**./src/blogposts/blogposts.11tydata.js**
```js
module.exports = {
  layout: "layouts/blogpost.njk",
  permalink: "blog/{{ page.fileSlug }}/index.html"
};
```

## 4. Templating avec Eleventy et Nunjucks

Eleventy vous permet de travailler avec différents langages de templating. Nunjucks de Mozilla est assez puissant et facile d'utilisation, c'est donc celui avec lequel je travaille d'habitude. La [documentation](https://mozilla.github.io/nunjucks/) étant assez bien faite, je ne vais pas ici rentrer dans le détail mais simplement réaliser les templates dont nous avons besoin pour notre projet de blog.

### Principaux tags de Nunjucks

Nunjucks possède trois grands types de tags

- tags de commentaires: `{# this is a comment #}`
- tags d'affichage: `{{ variable }}`
- tags de logique: `{% logique %}`

#### Tags de commentaires

Nunjucks possède un tag de commentaire: `{# Ceci est un commentaire #}`. Ceux-ci ne sont pas affiché lorsque le template est rendu.

#### Tags d'affichage, variables et propriétés

Ces tags vous permettent d'afficher des chaines de caractères, nombres, booléens, tableaux et objects dans vos templates. La plupart du temps, vous afficherez des variables créées par vous ou par Eleventy. Une notation pointée permet d'accéder aux propriétés de ces variables.

Exemples:

- `{{ "Hello World" }}`: affiche la chaîne de caractères "Hello World"
- `{{ data.title }}`: affiche la valeur de la key `title` dans l'objet `data`
- `{{ 8 + 2 }}`: affiche `10`

#### Filtres

Les filtres sont essentiellement destiné à manipuler des chaines de caractères, nombres, booléens, tableaux et objects tout en les affichants dans vos templates. Nunjucks possède de nombreux filtres par défaut. Voici quelques exemples.

- `{{ "this should be uppercase" | upper }}` va avoir comme output `THIS SHOULD BE UPPERCASE`.
- `{{ [1,2,3,4,5] | reverse }}` va avoir comme output `5,4,3,2,1` ce tag est particulièrement utile combiné avec des classements par date dans Eleventy.
- `{{ collections.blogposts | length }}` va afficher le nombre d'items présents dans votre collection de blogposts.

##### Filtres custom dans Eleventy

Eleventy vous permet de créer vos propres filtres en JavaScript à l'aide du fichier `.eleventy.js` ces filtres peuvent ensuite être utilisés dans le languge de te:plating que vous aurez choisi. Nunjucks ne possède pas de filtre permettant de formatter les dates, nous pouvons donc en créer facilement un dans Eleventy à l'aide de la librairie [`moment.js`](https://momentjs.com/).

```js
// required packages
const moment = require("moment");

module.exports = function(eleventyConfig) {

  // ... more configuration here .../

  /**
   * Date filter
   * @param {Date} date
   * @param {string} format - moment.js date formatting string
   */
  eleventyConfig.addFilter("date", function(date, format) {
    return moment(date).format(format);
  });
};
```

Une fois créé, vous pourrez utiliser ce filtre dans vos templates comme suit:

```njk
<p><time datetime="{{ page.date | date('YYYY-MM-DD') }}">{{ page.date | date("MMMM Do, YYYY") }}</time></p>
```

#### Tags de logique

Ces tags permettent l'exécution d'opérations et sont utilisés pour créer des variables, réaliser des boucles, créer des structures de contrôle, etc.

##### Assignation de variable

Le code ci-dessous va assigner tous les blogposts de la collection à une variable `blogposts` disponible dans la suite du template.

```njk
{% set blogposts = collections.blogposts %}
```

##### Structures de contôle

Nunjucks permet d'utiliser les structure de contrôle traditionelles telles que `if` et `else`, les opérateurs de comparaison tels que `===`, `!==`, `>`, etc. ainsi que les opérateurs logique comme `and`, `or` et `not`.

```njk
{% if collections.blogposts | length %}
  <p>There is at least one blogpost in this collection</p>
{% endif %}
```

```njk
{% if not collections.blogposts | length %}
  <p>There is no blogpost in this collection</p>
{% endif %}
```

##### Boucle `for`

Lorsque vous devrez afficher des données, qu'elles proviennent d'API ou de fichiers Markdown, vous devrez parcourir des tableaux ou des dictionnaires avec des boucles `for`. Voici par exemple comment afficher dans une liste les titres et intro de tous vos blogposts.

```njk
{% set blogposts = collections.blogposts %}
{% for entry in blogposts %}
  <ul>
    <h2><a href="{{ entry.url }}">{{ entry.data.title }}</a></h2>
    <p>{{ entry.data.intro }}</p>
  </ul>
{% else %}
  <p>No blogposts found</p>
{% endfor %}
```

### DRY: layout et includes

En plus d'un tag `{% include %}`, Nunjucks utilise l'héritage de template comme modèle de layout avec `{% extends %}`. Ce modèle permet de définir des blocks dans un template que les templates enfants vont venir surdéterminer. Les chaînes de templates peuvent être aussi longues que souhaitées.

Les includes comme l'héritage de templates sont utilisables avec Eleventy. La seule particularité est que les templates à étendre comme les fichiers à inclure doivent impérativement tous se trouver dans le dossier d'includes que vous avez spécifié dans le fichier de configuratioin `.eleventy.js`. Par défaut ce dossier est `_includes` et le chemin est relatif à votre dossier source.

**.eleventy.js**
```js
module.exports = function(eleventyConfig) {
  // override default config
  return {
    dir: {
      input: "src",
      output: "dist",
      // path is relative to the input directory
      // "_includes" is th default value
      includes: "_includes"
    }
  };
};
```

Lorsqu'un template parent est étendu par un template enfants, les variables définies dans le template enfant sont accessible dans le template parent. Outre `{% extends %}` et `{% include %}`, Nunjucks vous permet d'utiliser des [macros](https://mozilla.github.io/nunjucks/templating.html#macro) qui sont de petits bouts de code réutilisables auxquel des variables peuvent être passées. Eleventy possède un concept similaire avec la possibilités de créer des [shortcodes](https://www.11ty.io/docs/shortcodes/).

Pour en revenir à notre blog, voici les layouts dont nous aurons besoin.

#### Layouts

**./src/_includes/layouts/base.njk**
```njk
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>{{ metaTitle }} - {{ site.title }}</title>
  <meta name="description" content="{{ metaDescription }}">

  <link rel="stylesheet" href="/css/main.min.css?ref={{ site.buildTime | date('X') }}">

  <!-- Feed -->
  <link rel="alternate" href="/feed.xml" title="{{ site.title }} - Blog RSS" type="application/atom+xml">

  <!-- Twitter + Open Graph -->
  <meta name="twitter:card" content="summary" />
  <meta name="twitter:creator" content="{{ site.authorTwitter }}" />
  <meta property="og:url" content="{{ site.url ~ page.url }}" />
  <meta property="og:title" content="{{ metaTitle }}" />
  <meta property="og:description" content="{{ metaDescription }}" />
  <meta property="og:image" content="{{ metaImage }}" />

  <!-- Apple icon (minimal) -->
  <link rel="apple-touch-icon" href="/apple-touch-icon.png">

</head>
<body>
  {% include "partials/siteheader.njk" %}

  {% block content %}{% endblock %}

  {% include "partials/sitefooter.njk" %}
  <script src="/js/main.bundle.js?ref={{ site.buildTime | date('X') }}"></script>
</body>
</html>
```

Voici un exemple simple de fichier inclus utilisé pour le footer.

**./src/_includes/partials/blogpost.njk**
```njk
<div class="c-sitefooter">
  <p>&copy; {{ site.buildTime | date("Y") }} - La casa productions</p>
</div>
```

Pour ce qui est des blogposts, il nous faut un layout un peu particulier qui va venir étendre notre layout de base. Ce layout de blogpost va être utilisé par tous les fichiers Markdown de notre collection pour afficher les page de détail.

**./src/_includes/layouts/blogpost.njk**
```njk
{% extends "layouts/base.njk" %}

{% set metaTitle = title %}
{% set metaDescription = intro %}
{% set metaImage = site.url ~ "/assets/img/blogposts/" ~ imageSmall %}

{% block content %}
  <main>
    <article class="c-blogpost">
      <div class="c-blogpost__media">
        <picture>
            <source srcset="/assets/img/blogposts/{{ imageMedium }} 1024w,
                            /assets/img/blogposts/{{ imageBig }} 1500w"
                    sizes="(min-width: 1140px) 1140px,
                           100vw"
                    media="(min-width: 500px)">
            <img src="/assets/img/blogposts/{{ imageSmall }}"
                 class="o-fluidimage"
                 alt="{{ imageAlt }}">
        </picture>
      </div>

      <div class="c-blogpost__body">

        <header>
          <p class="c-suptitle  c-suptitle--dark"><time datetime="{{ page.date | date('YYYY-MM-DD') }}">{{ page.date | date("MMMM Do, YYYY") }}</time></p>
          <h1 class="c-h1">{{ title }}</h1>
          <div class="c-blogpost__intro">
            <p>{{ excerpt }}</p>
          </div>
        </header>

        <div class="u-markdown">
          {{ content | safe }}
        </div>

      </div>
    </article>
  </main>
{% endblock %}
```

#### Pages

Voici un exemple de template pour la page about.

**./src/pages/about.njk**
```njk
---
permalink: /about/index.html
---
{% extends "layouts/base.njk" %}

{% set metaTitle = "About us" %}
{% set metaDescription = "Meet the team behind the blog" %}
{% set metaImage = site.url ~ "/img/meta/team.jpg" %}

{% block content %}
  <h1 class="h1">Meet our awesome team</h1>

  {% set team = collections.team %}
  {% for member in team %}
    {% if loop.first %}<ul class="c-teamlist">{% endif %}
      <li class="c-teamlist__item">
        <article class="c-teammember">
          <img class="o-fluidimage" src="/assets/img/team/{{ member.image }}" >
          <h2 class="c-teammember__title">{{ member.name }} {{ member.surname }}</h2>
          <div class="c-teammember__bio">{{ member.templateContent | safe }}</div>
          {% if member.twitter or member.github or member.website %}
            <ul class="u-hlist  u-hlist--xs">
              {% if member.twitter %}<li><a href="{{ member.twitter }}">{% include "svg/icon-twitter.svg" %}</a></li>{% endif %}
              {% if member.github %}<li><a href="{{ member.github }}">{% include "svg/icon-github.svg" %}</a></li>{% endif %}
              {% if member.website %}<li><a href="{{ member.website }}">{% include "svg/icon-website.svg" %}</a></li>{% endif %}
            </ul>
          {% endif %}
        </article>
      </li>
    {% else %}
      <p>No team member found</p>
    {% if loop.last %}</ul>{% endif %}
  {% endfor %}
{% endblock %}
```

Pour la page d'archive de notre blog, nous allons utiliser la fonction de pagination d'Eleventy. Celle fonctionne en spécifiant quelles sont les données à paginer, combien d'items doivent être affichés par page et quel alias doit être utilisés pour les données une fois paginées.

**./src/pages/blog.njk**
```njk
---
pagination:
  data: collections.blogposts
  size: 12
  alias: posts
permalink: blog{% if pagination.pageNumber > 0 %}/page{{ pagination.pageNumber + 1}}{% endif %}/index.html
---
{% extends "layouts/base.njk" %}

{% set metaTitle = "Blog archives" %}
{% set metaDescription = "A blog about Belgium" %}
{% set metaImage = site.url ~ "/img/meta/default.jpg" %}

{% block content %}
  <h1 class="h1">A blog about Belgium</h1>

  {% for post in posts %}
    {% if loop.first %}<ul class="l-grid  l-grid--fluid">{% endif %}
      <li>
        <article class="c-blogpost">
          <a class="c-blogpost__link" href="{{ post.url }}">
            <img class="c-blogpost__image  o-fluidimage" src="/assets/img/blogposts/{{ post.imageSmall }}" >
            <h2 class="c-blogpost__title">{{ post.title }}</h2>
            <p class="c-blogpost__intro">{{ post.intro }}</p>
          </a>
        </article>
      </li>
    {% else %}
      <p>No blogpost found</p>
    {% if loop.last %}</ul>{% endif %}
  {% endfor %}

  {# pagination #}
  {% set totalPages = pagination.hrefs | length %}
  {% set currentPage = pagination.pageNumber + 1 %}
  {% if totalPages > 0 %}
    <ul class="c-pagination">
      {% if pagination.previousPageHref %}<li class="c-pagination__item  c-pagination__item--first"><a class="c-pagination__link" href="{{ pagination.firstPageHref }}">First</a></li>{% endif %}
      {% if currentPage > 1 %}<li class="c-pagination__item"><a class="c-pagination__link" href="{{ pagination.hrefs[pagination.pageNumber - 1] }}">{{ currentPage - 1 }}</a></li>{% endif %}
      <li class="c-pagination__item"><span class="c-pagination__current" href="{{ pagination.hrefs[pagination.pageNumber] }}">{{ currentPage }}</span></li>
      {% if currentPage < totalPages %}<li class="c-pagination__item"><a class="c-pagination__link" href="{{ pagination.hrefs[pagination.pageNumber + 1] }}">{{ currentPage + 1 }}</a></li>{% endif %}
      {% if pagination.nextPageHref %}<li class="c-pagination__item  c-pagination__item--last"><a class="c-pagination__link" href="{{ pagination.lastPageHref }}">Last</a></li>{% endif %}
    </ul>
  {% endif %}

{% endblock %}
```

### Pagination

La [fonction de pagination d'Eleventy](https://www.11ty.io/docs/pagination/) est bien plus puissante qu'elle n'y parait au premier abord. Si les données pour nos blogposts provenaient d'une API, nous pourrions utiliser ce fichier `./src/_data/_blogposts.js` et la même fonction de pagination pour générer toutes les pages de détail. Il suffit de spécifier une valeur de `1` pour le paramètre `size` et de préciser un pattern de `permalink` correspondant aux URL souhaitées.

Voici un exemple de template simplifié:

```njk
---
pagination:
  data: blogposts
  size: 1
  alias: blogpost
permalink: blog/{{ blogpost.slug }}/index.html
---
{% extends "layouts/base.njk" %}

{% set metaTitle = blogpost.title %}
{% set metaDescription = blogpost.intro %}
{% set metaImage = site.url ~ "/assets/img/blogposts/" ~ blogpost.imageSmall %}

{% block content %}

  <h1>{{ blogpost.title }}</h1>
  <p><time datetime="{{ blogpost.date | date('Y-M-DD') }}">{{ blogpost.date | date("MMMM Do, Y") }}</time></p>
  <p>{{ blogpost.intro }}</p>
  {{ blogpost.body | safe }}

{% endblock %}
```


### Exercice à faire ensemble

Partir des templates statiques fournis pour:

- Configurer Eleventy et créer un filtre de date et un filtre de limite
- Créer une page d'accueil listant les 6 derniers blogposts
- Créer une page d'archive paginée pour le blog
- Créer les pages de détail pour tous les blogposts
- Créer une page about avec les members de l'équipe
- Créer une navigation avec un fichier data
