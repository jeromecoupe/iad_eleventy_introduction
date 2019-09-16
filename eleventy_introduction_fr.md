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

Par defaut, Eleventy va ignorer le dossier `node_modules` ainsi que les dossiers, fichiers et globs spécifiés dans notre fichier `.gitignore`. Nous pouvons également créer un fichier `.eleventyignore` et placer un dossier, fichier ou glob par ligne pour explicitement spécifier les dossier et fichier à ignorer dans notre dossier source. Nous pouvons par exemple dire à Eleventy d'ignorer le contenu de l'ensemble de notre dossier assets, qui contiendra des fichiers CSS/SCSS/JS pris en charge par notre outil de build (Gulp dans ce cas).

`.eleventyignore`
```txt
./src/assets/
```

## 3. Définir et structurer vos données

- deux types de sources de données

#### Collections (markdown et YAML front matter)

- Markdown et YAML front matter
- collection API avec .eleventy.js
- accessibles à partir de l'object collections
- une collection spéciale: collections.all
- valeurs par defaut et fichiers de directory (JS ou JSON)

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
- conditionnels if / else

### Filtres

- filtres nunjucks
- filtres custom avec Eleventy (date et limit)

### Macros

- macros (parallèle avec fonctions)

### Pagination

- listes paginées
- pages de détail et pagination
