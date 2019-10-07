# Eleventy (11ty) par Zach Leatherman

## 1. Introduction

Eleventy est un static site generator créé et maintenu par Zach Leatherman. Comme tel, il n'utilise pas de base de données. Cet outil vous permet de développer des sites basés sur des templates (codés avec Nunjucks dans notre cas) et des fichiers de données (YAML / Markdown / HTML / JSON / JS) présents dans un dossier source. Sur base de ces fichiers, Eleventy va générer un site entièrement statique dans un dossier de destination. Vous pourrez ensuite déployer ce site sur n'importe quel serveur web.

Le but avoué d'Eleventy est d'être une alternative à Jekyll écrite en JavaScript plutôt qu'en Ruby. Tout comme Jekyll, c'est un SSG simple à utiliser et à configurer une fois les principes de base bien compris.

Node étant assez rapide, Eleventy est un SSG performant. Il est également très flexible. Tour d'abord, Eleventy est écrit en Node et vous permet donc d'utiliser facilement l'ensemble de l'écosystème NPM pour en étendre les fonctionnalités. Il vous permet également d'utiliser une large gamme de langages de templating. Nous utiliserons Nunjucks par Mozilla.

## 2. Installation and configuration

### Installation

Commençons d'abord par créer un dossier pour notre projet

```text
mkdir eleventy-portfolio
cd eleventy-portfolio
```

L'installation se fait à l'aide d'un module NPM. Pour installer Eleventy, il faut commencer par créer dans notre projet un fichier `package.json` qui reprendra toutes nos dépendances Node.

```text
npm init
```

Dans un premier temps, vous pouvez simplement accepter l'ensemble des réponses aux questions posées. Vous pourrez toujours y revenir par après en éditant votre fichier `package.json`. Voici la [documentation complète](https://docs.npmjs.com/files/package.json) pour vous aider à comprendre quelles sont les paramètres et options disponibles.

Nous pouvons ensuite installer Eleventy comme dépendance locale pour notre projet

```text
npm install --save-dev @11ty/eleventy
```

Eleventy va simplement installer un dossier `node_modules` mais rien d'autre. Pour pouvoir utiliser Eleventy, nous allons devoir le configurer et créer au moins un fichier. Pour le moment, nous allons simplement créer un fichier `index.html` statique qu'Eleventy devrait copier tel quel dans le dossier de destination qui est `_site` par défaut.

Une fois ce fichier créé, nous pouvons tester les commandes principales d'Eleventy:

- `npx eleventy`: pour faire tourner Eleventy
- `npx eleventy --serve`: pour faire tourner Browsersync afin d'avoir un serveur web local avec du hot reloading
- `npx eleventy --help`: pour avoir la liste des commandes existantes

Vous connaissez maintenant les commandes de base, nécessaires pour commencer à travailler. Si vous tapez `npx eleventy`, Eleventy devrait créer pour vous un dossier `_site` et dedans une copie de votre fichier `index.html`. Voyons maintenant comment configurer Eleventy en fonction de nos besoins.

### Configuration

Nous allons commencer par créer une architecture de projet et configurer Eleventy grâce au fichier de configuration `.eleventy.js`:

- Supprimer le dossier `./site`
- Créer un dossier `./src` et y placer notre fichier `index.html`
- Créer un fichier `.eleventy.js` dans la racine de notre projet

Commençons par spécifier les dossiers source et de destination pour Eleventy:

```js
module.exports = function(eleventyConfig) {
  // override default config
  return {
    dir: {
      input: "./src",
      output: "./dist"
    }
  };
};
```

Si nous exécutons la commande `npx eleventy` dans notre terminal, Eleventy générera un dossier `./dist` et y copier notre fichier `index.html`.

### Copy some files

Nous pouvons également utiliser ce fichier de configuration pour demander à Eleventy de copier n'importe quel fichier ou dossier depuis le dossier source jusqu'au dossier de destination via `addPassthroughCopy`. De bons candidats sont les assets statiques de notre projet comme les images et les fichiers de polices.

- créer un dossier `./src/assets/img/` et y placer un fichier image
- créer un dossier `./src/assets/fonts/` et y placer quelques fichiers de polices

Modifier le fichier `.eleventy.js` comme suit:

```js
module.exports = function(eleventyConfig) {
  // copy files
  eleventyConfig.addPassthroughCopy("./src/assets/img/");
  eleventyConfig.addPassthroughCopy("./src/assets/fonts/");

  // override default config
  return {
    dir: {
      input: "./src/",
      output: "./dist/"
    }
  };
};
```

Eleventy va maintenant copier ces dossiers ainsi que tout ce qu'ils contiennent.

### Ignorer certains dossiers et fichiers

Par défaut, Eleventy va ignorer le dossier `node_modules` ainsi que les dossiers, fichiers et globs spécifiés dans notre fichier `.gitignore`. Nous pouvons également créer un fichier `.eleventyignore` et spécifier un dossier, fichier ou glob par ligne pour dire à Eleventy de les ignorer dans notre dossier source. A part les fichiers de polices et les images déjà copiés par Eleventy, le dossier `./src/assets` ne contiendra que des fichiers CSS/SCSS/JS pris en charge par notre outil de build (Gulp dans ce cas-ci). Nous pouvons donc simplement dire à Eleventy d'ignorer ce dossier dans son ensemble:

**.eleventyignore**

```txt
./src/assets/
```

Nous avons maintenant une solide configuration de base pour la suite de notre projet. Nous reviendrons à ce fichier de configuration lorsque nous aborderons les collections dans le chapitre suivant.

### Intégrer Eleventy et Gulp

Eleventy ne possède pas d'asset pipeline pour vous permettre d'utiliser Sass ou Webpack. Il est donc intéressant d'intégrer Eleventy à un outil de build, que ce soit des scripts NPM, Gulp ou toute autre alternative.

Personnellement, j'utilise Gulp et Webpack. Voici une version simplifiée d'un fichier `gulpfile.js` standard:

```js
// required packages
const sass = require("gulp-sass");
const postcss = require("gulp-postcss");
const autoprefixer = require("autoprefixer");
const cssnano = require("cssnano");
const rename = require("gulp-rename");
const gulp = require("gulp");
const webpack = require("webpack");
const webpackConfig = require("./webpack.config.js");
const childProcess = require("child_process");
const del = require("del");
const browserSync = require("browser-sync").create();

// browsersync server
function server(done) {
  browserSync.init({
    server: "./dist/",
    files: [
      "./dist/assets/**/*",
      "./dist/*.{html, xml}",
      "./dist/**/*.{html, xml}"
    ],
    port: 3000,
    open: false
  });
  done();
}

// clean dist folder
function clean() {
  return del(["./dist/"]);
}

// build styles (sass)
function stylesBuild() {
  return gulp
    .src("./src/assets/scss/main.scss")
    .pipe(sass({ outputStyle: "expanded" }))
    .pipe(gulp.dest("./dist/assets/css/"))
    .pipe(rename({ suffix: ".min" }))
    .pipe(postcss([autoprefixer(), cssnano()]))
    .pipe(gulp.dest("./dist/assets/css/"));
}

// JS scripts (webpack: transpile, lint, minify)
function scriptsBuild() {
  return new Promise((resolve, reject) => {
    webpack(webpackConfig, (err, stats) => {
      const info = stats.toJson();

      if (err) {
        return reject(err);
      }

      if (stats.hasErrors()) {
        return reject(info.errors);
      }

      if (stats.hasWarnings()) {
        return reject(info.warnings);
      }

      console.log(
        stats.toString({
          chunks: false,
          colors: true
        })
      );

      resolve();
    });
  });
}

// Run Eleventy
function eleventyBuild() {
  return childProcess.spawn("npx", ["eleventy", "--quiet"], {
    stdio: "inherit"
  });
}

// Watch files
function watchFiles() {
  gulp.watch("./src/assets/scss/**/*", stylesBuild);
  gulp.watch("./src/assets/js/**/*", scriptsBuild);
  gulp.watch(
    [
      "./.eleventy.js",
      "./src/**/*",
      "!./src/assets/js/**/*",
      "!./src/assets/scss/**/*"
    ],
    eleventyBuild
  );
}

const watch = gulp.parallel(server, watchFiles);
const build = gulp.series(
  clean,
  gulp.parallel(stylesBuild, scriptsBuild, eleventyBuild)
);

exports.build = build;
exports.watch = watch;
```

Eleventy est maintenant intégré à notre workflow Gulp et les commandes `gulp build` et `gulp watch` vont toutes deux faire tourner Eleventy.

## 3. Définir et structurer vos données

Eleventy permet de travailler avec deux grandes sources de données:

- des fichiers Markdown (pour le contenu principal) et YAML front matter (pour le reste de la data structures) qui peuvent être facilement convertis en collections.
- des fichiers JSON et/ou JS qui peuvent soit être statiques soit dynamiques (provenant d'une API).

Ces deux sources de données ne sont pas mutuellement exclusives et sont généralement utilisées simultanément dans tout projet. Voyons cela plus en détail.

#### Collections

##### Markdown et YAML front matter

Les fichiers Markdown couplés à un YAML front matter permettent d'utiliser de simples fichiers textes comme source de données structurées. C'est un classique avec la plupart des SSG.

La partie en Markdown représente le contenu principal de vos données et est généralement simplement converti en HTML. Le YAML front matter permet de créer votre data structure à l'aide de différents types de données (tableaux, strings, objets, etc.).

Si vous devez construire un blog, vos blogposts seront chacun représenté par un fichier Markdown avec un YAML front matter qui pourrait ressembler à ceci:

**./src/blog/2019-07-22-markdown-yaml-front-matter.md**

```md
---
title: "This is the title"
intro: "This is an introductory paragraph for a blogpost"
imageSmall: "cover_600.jpg"
imageMedium: "cover_1024.jpg"
imageBig: "cover_1500.jpg"
imageAlt: "Alternative text for cover picture"
categories:
- front-end
- JAMstack
- Eleventy
---

## Level 2 title

Lorem ipsum dolor sit amet, consectetur adipisicing elit. Recusandae voluptatibus reiciendis dignissimos accusantium illo, voluptates consequuntur fugit amet quo sed nisi facere animi incidunt assumenda exercitationem, nam omnis perspiciatis praesentium.
```

**./src/projects/2018-01-01-my-project-name.md**

```md
---
client: "My Myself and I"
title: "A splendid project title"
imageSmall: "cover_800.jpg"
imageMedium: "cover_1024.jpg"
projectUrl: "https://www.webstoemp.com"
---

Lorem ipsum dolor sit amet, consectetur adipisicing elit. Recusandae voluptatibus reiciendis dignissimos accusantium illo, voluptates consequuntur fugit amet quo sed nisi facere animi incidunt assumenda exercitationem, nam omnis perspiciatis praesentium.
```

##### Collection API

Pour qu'Eleventy groupe tous ces fichiers dans un tableau et vous permette de les manipuler dans vos templates, il suffit les déclarer comme faisant partie d'une [une collection](https://www.11ty.io/docs/collections/). N'importe quel élément de contenu peut faire partie d'une oou plusieurs collections.

Pour créer une collection, vous pouvez assigner le même `tag` à différents éléments de contenu. Personnellement, je préfère utiliser l'API collection et le fichier `eleventy.js`.

Cette collection API vous offre [différentes méthodes pour déclarer des collections](https://www.11ty.io/docs/collections/#collection-api-methods) qui ont chacune leur utilité. Celle que j'utilise personnellement le plus est `getFilteredByGlob(glob)` qui vous permet de grouper dans une collection tous les éléments de contenus qui correspondent au même glob pattern.

Si tous vos fichiers Markdown sont placé dans un dossier `./src/blog/`, créer une collection les rassemblant tous est assez simple. Il vous suffit d'ajouter le code suivant dans votre fichier `eleventy.js`

```js
module.exports = function(eleventyConfig) {
  // blogposts collection
  eleventyConfig.addCollection("blogposts", function(collection) {
    return collection.getFilteredByGlob("./src/blog/*.md");
  });

  // projects collection
  eleventyConfig.addCollection("projects", function(collection) {
    return collection.getFilteredByGlob("./src/projects/*.md");
  });

  // override default config
  return {
    dir: {
      input: "./src",
      output: "./dist"
    }
  };
};
```

Vous pouvez maintenant accéder à vos collection en utilisant `collections.blogposts` et `collections.projects` dans vos templates. Nous y reviendrons dans le chapitre consacrée au templating.

Il me reste à signaler qu'Eleventy créé par défaut une collection contenant tous vos élements de contenus, c'est à dire tous les fichiers gérés par Eleventy. Cette collection spéciale est adressable via `colections.all`.

Lorsqu'une collection est créée, les key suivantes sont automatiquement créées. Les keys créées dans votre front matter sont accessibles via `data`.

- `inputPath`: the full path to the source input file (including the path to the input directory)
- `inputPath`: le chemin complet vers le fichier source (inclus le chemin vers le dossier d'input d'Eleventy)
- `fileSlug`: une transformation en slug du nom de fichier du fichier source. Utile dans la construction de permalinks. En savoir plus sur `fileslug` [dans la documentation](https://www.11ty.io/docs/data/#fileslug).
- `outputPath`: le chemin complet vers le fichier d'output de cet élément de contenu
- `url`: l'URL utilisée pour lier vers un élément de la collection. En général cette valeur est basée sur celle de la key `permalink`
- `date`: la date utilisée pour le classement. Pour en savoir plus sur les [dates des éléments de contenus](https://www.11ty.io/docs/dates/), référez-vous à la documentation.
- `data`: toutes les données pour cet élément de contenu. Se réfère aux champs du YAML front-matter et aux données héritée des layouts.
- `templateContent`: le contenu de ce template une fois rendu par Eleventy. N'inclus pas les templates étendus.

##### Classer et filtrer vos collections

Lorsque vous créez une collection avec l'API d'Eleventy, les items de cette collection sont automatiquement classés en ordre ascendant en utilisant:

1. La date renseignée dans le nom de fichier ou dans le YAML front matter du fichier source ou, a defaut, la date de création de celui-ci.
2. Si certains fichiers source ont une date identique, le chemin complet (y compris le nom de fichier) est pris en compte

Si un classement par date correspond à ce que vous souhaitez, vous pouvez éventuellement inverser celui-ci en utilisant le filtre `reverse` de Nunjucks.

Un cas de figure courant pourrait être de classer des membres d'une équipe alphabétiquement sur base de leurs noms. Chaque personne aurait une key `surname` dans le YAML front matter de son fichier Markdown. Le code suivant classerait ces membres de l'équipe par ordre ascendant sur base de leurs noms.

```js
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
```

Si vous devez par contre filtrer une collection pour exclure certaines données, il vous faudra utiliser la méthode `filter` en JavaScript. Vous pouvez par exemple exclure de la collection `blogposts` les items ayant une key `draft` dont la valeur est `true`.

```js
// blogposts collection
eleventyConfig.addCollection("blogposts", function(collection) {
  return collection.getFilteredByGlob("./src/blog/*.md").filter((item) => {
    return item.data.draft !== true;
  });
});
```

#### Data files (JS ou JSON)

Outre les collections, l'autre grande source de données pour Eleventy sont les fichiers data. Ceux-ci peuvent être statiques ou dynamiques.

Ces fichiers doivent par défaut être stockés dans le dossier `./src/_data/`. Cet emplacement des fichiers data peut être modifié dans votre fichier de configuration `.eleventy.js`.

```js
module.exports = function(eleventyConfig) {
  // override default config
  return {
    dir: {
      input: "./src/",
      output: "./dist/",
      data: "./_data/"
    }
  };
};
```

##### Fichiers data statiques

Les fichiers de data statiques sont simplement des fichiers JSON ou JS contenant des key et des values.

`./src/_data/site.js`

```js
module.exports = {
  title: "Title of the site",
  description: "Description of the site",
  url: "https://www.mydomain.com",
  baseUrl: "/",
  author: "Name Surname",
  authorTwitter: "@twitterhandle",
  buildTime: new Date()
};
```

Les données sont accessibles dans vos templates à l'aide des noms de fichier utilisé comme key. Par exemples les données contenues dans le fichier `./src/_data/site.js` sont accessibles dans vos templates via la variable `site`.

##### Fichiers data dynamiques

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
permalink: "blog/index.html"
---
```

Vous pouvez également utiliser des variables pour créer vos valeurs de `permalink`.

```text
---
permalink: "blog/{{ page.fileSlug }}/index.html"
---
```

Si vous avez une collection pour présenter vos projets mais que vous n'avez pas de page de détail, vous pouvez simplement utiliser la valeur `false` pour `permalink` et Eleventy ne générera alors pas de pages de détail. Dans la plupart des cas, vous n'aurez alors pas besoin de layout non plus.

```text
---
permalink: false
layout: false
---
```

Nous verrons plus loin que vous aurez alors besoin d'utiliser `templateContent` pour afficher le contenu de vos fichiers Markdown.

##### Valeurs par défaut et fichiers de données liés aux dossiers

Plutôt que de spécifier une valeur YAML front matter identique dans tous les fichiers d'une collection, Eleventy vous offre la possibilité de spécifier des valeurs identiques pour tous les fichiers contenus dans un directory en utilisant des [directory data files](https://www.11ty.io/docs/data-template-dir/) en JS ou en JSON.

Si vous devez par exemple spécifier une valeur pour `layout` et `permalink` identiques pour tous vos blogposts, vous pouvez simplement les spécifier dans un fichier `.src/blogposts/blogposts.json`, `.src/blogposts/blogposts.11data.json` ou `.src/blogposts/blogposts.11data.js`. Eleventy appliquera ces valeurs à tous les fichiers du directory ou des directory enfants.

**./src/blogposts/blogposts.json` ou `./src/blogposts/blogposts.11tydata.json**

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

## 4. Templating

Eleventy vous permet de travailler avec différents langages de templating. Je trouve Nunjucks de Mozilla puissant et facile d'utilisation, c'est donc celui avec lequel je travaille d'habitude. La documentation étant assez bien faite, je ne vais pas ici rentrer dans le détail mais simplement réaliser les templates dont nous avons besoin pour notre projet de blog.

### Les bases de Nunjucks

- tags de logique
- tags d'affichage

### DRY: layout et includes

- layouts et héritage avec Nunjucks
- includes (header et footer)

### Boucles et structures de contrôle

#### `for` et `loop`

```twig
{% for item in collections.blogposts | reverse %}
  {% if loop.first %}<ul>{% endif %}
    <li>
      <article>
        <h2><a href="{{ item.url }}">{{ item.title }}</h2>
        <p>{{ item.data.intro }}</p>
      </article>
    </li>
  {% if loop.last %}</ul>{% endif %}
{% else %}
  <p>No blogpost found</p>
{% endfor %}
```

#### conditionnels

### Filtres

- filtres nunjucks
- filtres custom avec Eleventy (date et limit)

### Macros

- macros (parallèle avec fonctions)

### Pagination

- listes paginées
- pages de détail et pagination
