# Eleventy (11ty) par Zach Leatherman

## 1. Introduction

Eleventy est un static site generator créé et maintenu par Zach Leatherman. Comme tel, il n'utilise pas de base de données. Cet outil vous permet de développer des sites basés sur des templates (codés avec Nunjucks dans notre cas) et des fichiers de données (YAML / Markdown / HTML / JSON / JS) présents dans un dossier source. Sur base de ces fichiers, Eleventy va générer un site entièrement statique dans un dossier de destination. Vous pourrez ensuite déployer ce site sur n'importe quel serveur web.

Le but avoué d'Eleventy est d'être une alternative à Jekyll écrite en JavaScript plutôt qu'en Ruby. Tout comme Jekyll, c'est un SSG simple à utiliser et à configurer une fois les principes de base bien compris.

Node étant assez rapide, Eleventy est un SSG performant et rapide. Il est également très flexible. Tour d'abord, Eleventy est écrit en Node et vous permet donc d'utiliser facilement l'ensemble de l'écosystème NPM pour en étendre les fonctionnalités. Il vous permet également d'utiliser une large gamme de languages de templating. Nous utiliserons Nunjucks par Mozilla.

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

Nous pouvons ensuite installer Eleventy comme dépendance lcoale pour notre projet

```text
npm install --save-dev @11ty/eleventy
```

Eleventy va simplement installer un dossier `node_modules` mais rien d'autre. Pour pouvoir utiliser Eleventy, nous allons devoir le configurer et créer au moins un fichier. Pour le moment, nous allons simplement créer un fichier `index.html` statique qu'Eleventy devrait copier tel quel dans le dossier de destination qui est `_site` par defaut.

Une fois ce fichier créé, nous pouvons tester les commandes principales d'Eleventy

- `npx eleventy`: pour faire tourner Eleventy
- `npx eleventy --serve`: pour faire tourner Browsersync afin d'avoir un serveur web local avec du hot reloading
- `npx eleventy --help`: pour avoir la liste des commandes existantes

Vous connaissez maintenant les commande de base, nécessaires pour commencer à travailler. Si vous tapez `npx eleventy`, Eleventy devrait créer pour vous un dossier `_site` et dedans une copie de votre fichier `index.html`. Voyons maintenant comment rendre Eleventy un peu plus utile.

### Configuration

Nous allons commencer par créer une architecture de projet un peu plus efficace et configurer Eleventy grâce au fichier de configuration `.eleventy.js`:

- Supprimer le dossier `./site`
- Créer un dossier `./src` et y placer notre fichier `index.html`
- Créer un fichier `.eleventy.js` dans la racine de notre projet

Nous allons commener par spécifier les dossiers source et de destination pour Eleventy:

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

Si nous exécutons la commande `npx eleventy` dans notre terminal, Eleventy va générer un dossier `./dist` et y copier notre fichier `index.html`.

### Passtrough copy

Nous pouvons également utilisre ce fichier de configuration pour demander à Eleventy de copier n'importe quel fichier ou dossier depuis le dossier source jusqu'au dossier de destination via `addPassthroughCopy`. De bons candidats sont les assets statiques de notre projet comme les images et les fichiers de polices.

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

### Ignorer certains dosiers et fichiers

Par defaut, Eleventy va ignorer le dossier `node_modules` ainsi que les dossiers, fichiers et globs spécifiés dans notre fichier `.gitignore`. Nous pouvons également créer un fichier `.eleventyignore` et spécifier un dossier, fichier ou glob par ligne pour dire à Eleventy de les ignorer dans notre dossier source. A part les fichiers de polices et les images déjà copiés par Eleventy, le dossier `./src/assets` ne contiendra que des fichiers CSS/SCSS/JS pris en charge par notre outil de build (Gulp dans ce cas-ci). Nous pouvons donc simplement dire à Eleventy d'ignorer ce dossier dans son ensemble:

`.eleventyignore`

```txt
./src/assets/
```

Nous avons maintenant une solide configuration de base pour la suite de notre projet. Nous reviendrons à ce fichier de configuration lorsque nous aborderons les collections dans le chapitre suivant.

### Intégrer Eleventy et Gulp

Eleventy ne possède pas d'asset pipeline pour vous permettre d'utiliser Sass ou Webpack. Il est donc intéressant d'intégrer Eleventy à un outil de build, que ce soit des scripts NPM, Gulp ou toute autre alternative.

Personellement, j'utilise Gulp et Webpack. Voici une version simplifiée d'un fichier `gulpfile.js` standard:

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
const browsersync = require("browser-sync").create();

// browsersync server
function server(done) {
  browsersync.init({
    server: "./dist/",
    files: ["./dist/assets/css/*.css", "./dist/assets/js/*.js", "./dist/*.{html, xml}", "./dist/**/*.{html, xml}"],
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

//
function watchFiles() {
  gulp.watch("./src/assets/scss/**/*", stylesBuild);
  gulp.watch("./src/assets/js/**/*", scriptsBuild);
  gulp.watch(["./.eleventy.js", "./src/**/*", "!./src/assets/js/**/*", "!./src/assets/scss/**/*"], eleventyBuild);
}

const watch = gulp.parallel(server, watchFiles);
const build = gulp.series(clean, gulp.parallel(stylesBuild, scriptsBuild, eleventyBuild));

exports.build = build;
exports.watch = watch;
```

Eleventy est maintenant intégré à notre workflow Gulp et les commandes `gulp build` et `gulp watch` intègrent toutes deux Eleventy.

## 3. Définir et structurer vos données

Eleventy permet de travailler avec deux grandes sources de données:

- des fichiers Markdown (pour le contenu principal) et YAML front matter (pour le reste de la data structures) qui peuvent être facilement convertis en collections.
- des fichiers JSON et/ou JS qui peuvent soit petre statiques soit dynamiques (provenant d'une API).

Ces deux sources de données ne sont pas mutuellement exclusives et sont généralement utilisées simultanément dans tout projet. Voyons cela plus en détail.

#### Collections

##### Markdown et YAML front matter

Les fichiers Markdown couplés à un YAML front matter permettent d'utiliser de simples fichiers textes comme source de données structurées. C'est un classique avec la plupart des SSG.

La partie en Markdown représente le contenu principal de vos données et est généralement simplement converti en HTML. Le YAML front matter permet de créer votre data structure à l'aide de différents types de données (tableaux, strings, objets, etc.).

Si vous devez par exemple construire un blog, vos blogposts seront chacun représenté par un fichier Markdown avec un YAML front matter qui ressemble à ceci.

**./src/blog/2019-07-22--markdown-yaml-front-matter.md**

```txt
---
title: "This is the title"
intro: "This is an introductory sentence for a blogpost"
categories: ["front-end", "JAMstack", "Eleventy"]
---
This is the main content of my blogpost
```

##### Collection API

Pour qu'Eleventy groupe tous ces fichiers dans un tableau et vous permette de les manipuler dans vos templates, il suffit les déclarer comme faisant partie d'une [une collection](https://www.11ty.io/docs/collections/). N'importe quel élément de contenu peut faire partie d'une oou plusieurs collections.

Pour créer une collection, vous pouvez assigner le même `tag` à différents éléments de contenu. Personellement, je préfère utiliser l'API collection et le fichier `eleventy.js`.

Cette collection API vous offre [différentes méthodes pour déclarer des collections](https://www.11ty.io/docs/collections/#collection-api-methods) qui ont chacune leur utilité. Celle que j'utilise personnellement le plus est `getFilteredByGlob(glob)` qui vous permet de grouper dans une collection tous les éléments de contenus qui correspondent au même glob pattern.

Si tous vos fichiers markdown sont placé dans un dossier `./src/blog/`, créer une collection les rassemblant tous est assez simple. Il vous suffit d'ajouter le code suivant dans votre fichier `eleventy.js`

```js
module.exports = function(eleventyConfig) {
  // blogposts collection
  eleventyConfig.addCollection("blogposts", function(collection) {
    return collection.getFilteredByGlob("./src/blog/*.md");
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

Vous pouvez maintenant accéder à votre collection en utilisant `collections.blogposts`. Nous y reviendrons dans le chapitre consacrée au templating.

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

Lorsque vous crééez une collection avec l'API d'Eleventy, les items de cette collection sont automatiquement classés en ordre ascendant en utilisant:

1. La date renseignée dans le nom de fichier ou dans le YAML front matter du fichier source ou, a defaut, la date de création de celui-ci.
2. Si certains fichiers source ont une date identique, le chemin complet (y compris le nom de fichier) est pris en compte

Si un classement par date correspond à ce que vous souhaitez, vous pouvez évntuyellement inverser celui-ci en utilisant le filtre `reverse` de Nunjucks.

Pour classer les élements d'une collection sur d'autres critères, il vous faudra simplement utiliser JavaScript et la methode `sort`. Admetons par exemple que vous deviez classer des membres de l'équipe alphabétiquement sur base de leurs noms. Chaque team member à une key `surname` dans le YAML front matter de son fichier Markdown. Le code suivant classera ces team members par ordre ascendant sur base de leurs noms.

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

Si vous devez par contre filtrer un collection pour exclure certaines données, il vous faura utiliser la méthode `filter` en JavaScript. Vous pouvez par exemple exclure de la collection `blogposts` les items ayant une key `draft` dont la valeur est `true`.

```js
// blogposts collection
eleventyConfig.addCollection("blogposts", function(collection) {
  return collection.getFilteredByGlob("./src/blog/*.md").filter((item) => {
    return item.data.draft !== true;
  });
});
```

##### Valeurs par defaut et fichiers données liés aux dossiers

@TODO

#### Data files (JS ou JSON)

- fichiers statiques
- fichiers dynamiques connectés à un endpoint API
- données accessibles en utilisant le nom de fichier comme key
- exemple avec des valeurs en key / value pairs (site.js)
- exemple avec un tableau d'objets en provenance d'une API

### Permalinks et URLs

- simple permalink
- permalink false
- utilisation de variables dans les permalinks

## 4. Templating

### Les bases de Nunjucks

- tags de logique
- tags d'affichage

### DRY: layout et includes

- layouts et héritage avec Nunjucks

- includes (header et footer)

### Boucles et structures de contrôle

- for et loop

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

- conditionnels if / else

### Filtres

- filtres nunjucks
- filtres custom avec Eleventy (date et limit)

### Macros

- macros (parallèle avec fonctions)

### Pagination

- listes paginées
- pages de détail et pagination
