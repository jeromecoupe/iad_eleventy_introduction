# Eleventy (11ty) by Zach Leatherman

## 1. Introduction

[Eleventy](https://www.11ty.io) is a Static Site Genertor created and maintained by [Zach Leatherman](https://www.zachleat.com/). Eleventy allows you to develop websites based on templates and data or content files (YAML / Markdown / HTML / JSON / JS) in a source directory. Based on those source files, Eleventy will generate a fully functional static site in a destination folder. You will then be able to deploy that site on any web server capable of serving static files.

The [stated goal of Eleventy]((https://www.11ty.io/docs/)) is to be an alternative to [Jekyll](https://jekyllrb.com/) written in Node rather than in Ruby. Just as Jekyll it is a very approachable and simple to use SSG once the basic principles are well understood.

Node being quite fast, Eleventy is a performant SSG. It is also very flexible. Since it is written in Node, Eleventy allows you to use the NPM ecosystem to build extends its functionalities. You can also pick your favourite in a [long list of templating languages](https://www.11ty.io/docs/languages/). We will use [Nunjucks](https://mozilla.github.io/nunjucks/) by [Mozilla](https://www.mozilla.org) for all the code samples in this course.

## 2. Installation and configuration

### Installation

Let's start by creating a folder for our new project:

```text
mkdir eleventy-portfolio
cd eleventy-portfolio
```

You will need [Node](https://nodejs.org) installed on your machine. You can then install Eleventy as an NPM module. The best way to proceed is to create a `package.json` that will manage all our Node dependencies.

```text
npm init -y
```

That will skip a bunch of questions and create a standard file. You can come back to it and edit your `package.json` as you see fit. Here is the [official documentation](https://docs.npmjs.com/files/package.json) to help you find out about the available parameters and options.

We can then install Eleventy as a local dependency for our project:

```text
npm install --save-dev @11ty/eleventy
```

At this point, Eleventy will simply create a `node_modules` folder and install itself, but nothing more. To see it in action, we will have to create at lest one file. Let's simply create a basic `index.html` file and that Eleventy will copy in the default `_site` destination folder.

Once we have created that file, we can learn the basic Eleventy command

- `npx eleventy`: to run Eleventy
- `npx eleventy --serve`: to run BrowserSync and have a local web server that will reload the site in your browser when the site changes
- `npx eleventy --help`: to explore the list of available commands and flags

If we now type `npx eleventy` in our terminal, Eleventy will create a `_site` folder and copy our `index.html` file in it. Pretty impressive, right ?

Let's now configure Eleventy to better suit our needs.

### Configuration

We will make a simple project architecture and configure Eleventy by creating a `.eleventy.js` configuration file at the root of our project.

- Delete the `./site` folder created by Eleventy
- Create an `./src` folder and move `index.html` to it
- Create an `.eleventy.js` file at the root of our project

Let's start by specify source and destination folders for Eleventy:

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

When we now run the `npx eleventy` command in our terminal while being at the root of our project, Eleventy will generate a `./dist` folder and will copy our trusty `index.html` file to it.

### Tell Eleventy to copy some folders and files

We can also use this configuration file to tell Eleventy to copy any file or folder from the source directory to the destination directory. In order to instruct Eleventy to do so, we are going to use [`addPassthroughCopy`](https://www.11ty.io/docs/copy/). Good candidates to copy are static assets like images and font files.

- create a `./src/assets/img/` directory and drop a few optimised images in there
- create a `./src/assets/fonts/` directory and drop a few font files in there
- create a `./src/assets/js/` directory and drop a JavaScript file in there
- create a `./src/assets/css/` directory and drop a CSS files in there

Let's modify our `.eleventy.js` file as follows:

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

Eleventy will now copy the `./src/assets/` directory and everything it contains to the output directory while preserving the directories structure.

### Ignore directories and files

By default, Eleventy will ignore the `node_modules` directory as well as the folder, files and globs specified in your `.gitignore` file.

We can also create a `.emeventyignore` folder at the root of our project and spécify a file, directory or glob pattner per line to explicitly tell Eleventy to ignore all matching files and directories. I have painfully learned that you always want to be as explict as possible, not ony in your code actually. Let's do this.

**.eleventyignore**
```txt
node_modules/
dist/
```

### Assets pipeline and build tools

Eleventy doesn't have an asset pipeline or a build tool by default. I almost always integrate it with build tools, be it with NPM scripts, Gulp, Webpack or any other alternative.

When you start using build tools to create an assets pipeline, you will likely have to modify your `addPassthroughCopy` and ignore assets directories that will not need to be handled by Eleventy because your build tools and scripts will take care of them and generate what you need in your `./dist` directory.

If, for example, you are using NPM scripts to build your CSS from Sass files or you use Webpack to handle your JavaScript pipeline, you will have to make the following modifications:

**.eleventy.js**
```js
module.exports = function(eleventyConfig) {
  // copy files
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
node_modules/
dist/
src/assets/scss/
src/assets/js/
```

Eleventy will now completely ignore the `./src/assets/scss/` and `./src/assets/js/` directories. Your build tools and scripts will generate the required outputs in your `./dist` directory.

Personnally, I use Gulp combined with Webpack for most of my projects. Eleventy can very easily be integrated to this kind of workflow.

## 3. Define and structure your data

Eleventy allows you to work with two data sources:

1. **Markdown files** (for the main content) and YAML front matter (for the rest of the data structure) that can easily be turned into collections (more on that later).
2. **JSON and/or JS data files** that can either be static or dynamic (fetched from an API).

Thse two types of data sources are not mutually exlcusive and are, in fact, used simultaneously in most projects. Let's dive in.

### Collections

Collections in Eleventy allow you to group content items in interesting ways and to use them in your templates.

#### Markdown and YAML front matter

Markdown files coupled with a YAML front matter allow you tyo use simple etxt files as structured data sources. It's a staple feature of most SSG out there.

The Mardown part of the file generally represents the main content of your data and is generally converted to HTML. The YAML front matter allow you to create a data structure with diffrent types of data (strings, arrays, objects, etc.).

If you want to build a blog, your blogposts are going to be represented by Markdown files with a YAML front matter that could look something like the following:

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

Let's say you also need to model a data structire for a team. Each team member could be represented by a file like this one:

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

Jérôme Coupé is a looney front-end designer and teacher from Brussels, Belgium. When not coding, he might have been seen downtown, drinking a few craft beers with friends.
```

#### Collection API

For Eleventy to group those files in an array that will allow you to manipulate that array in your templates, you just have to create a [collection](https://www.11ty.io/docs/collections/). Any content item can be part of one or more collections.

To create a collection, you can assign the same `tag` to various content items. Personally, I would much rather use the collection API and our trusty `.eleventy.js` file

This API offers you [different methods to declare your collections](https://www.11ty.io/docs/collections/#collection-api-methods) that each have their use. My favourite and most used one by far is `getFilteredByGlob(glob)` that allows you create a collection from all files matching a defined glob pattern.

If all your Markdown files for your blogposts are in a `./src/blog/` directory, grouping them into a collection couldn't be simpler. You just have to add the following code to your `.eleventy.js` configuration file. While we are at it, we will also add our `team` collection.

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

You can now access your collections using `collections.blogposts` and `collections.team` in your templates. We will come back to them when we get to templating.

It is also important to know that, by default, Eleventy creates a collection that contains all your content items. This special collection can be accessed in your templates by using `collections.all`.

When a collection is created, the following keys are automatically created:

- `inputPath`: the full path to the source input file (including the path to the input directory)
- `fileSlug`: Mapped from the input file name, useful for permalinks. Read more about [`fileslug`](https://www.11ty.io/docs/data/#fileslug).
- `outputPath`: the full path to the output file to be written for this content
- `url`: url used to link to this piece of content. In general, this is based on the `permalink` key defined for your content items.
- `date`: the resolved date used for sorting. Read more about [Content Dates](https://www.11ty.io/docs/dates/) in the documentation.
- `data`: all data for this piece of content. Contains the keys/values defined in the YAML front-matter and the data inherited from layouts.
- `templateContent`: the rendered content of this template. Does not include layout wrappers.

#### Sort and filter your collections

When you create a collection in Eleventy, the items in it are automatically sorted in ascending order using:

1. The date specified in the filename or in the YAML front matter of the source file or will use the creating date on the filesystem as a fallback.
2. If some files have an identicaly date, the full path of the file (including the filename) will be taken into account as well.

If what you need is to sort your collection items by date, you are covered. Just use the `reverse` filter in Nunjucks if you need it.

By contrast, if you need to sort your team members alphabetically using the value of their `surname` key, you will need to use the JavaScript `sort` method.

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

If you need to filter a collection to exclude some data, you can use the JavaScript `filter` method. You can for example only include only the blogposts that do not have a `draft` key set to `true` in their front matter and that have a publication date smaller than the date at which the site is generated.

```js
const now = new Date();

module.exports = function(eleventyConfig) {

  // ... more configuration here .../

  // blogposts collection
  eleventyConfig.addCollection("blogposts", function(collection) {
  return collection.getFilteredByGlob("./src/blog/*.md").filter((item) => {
    return item.data.draft !== true && item.date <= now;
  });
};
```

### Data files (JS ou JSON)

Aside from collections, the other data source you can use with Eleventy are data files. Those can be static or dynamic (retrieved from an API).

By default, these files have to be stored in the `./src/_data/` directory. This can be modified uing the `.eleventy.js` configuration file.

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

#### Static data files

Static data files are simply JSON or JS files containing key/value pairs.

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

Those values can be accessed in your templates by using the filename as a key. For exemple, data contained in a `./src/_data/site.js` file will be accessible in your templates using the `site` variable.

#### Dynamic data files

Because data files can be JavaSCript files, nothing is preventing you from [connecting to an API](https://www.webstoemp.com/blog/headless-cms-graphql-api-eleventy/) in one of those files by using [`node-fetch`](https://www.npmjs.com/package/node-fetch) or [`axios`](https://www.npmjs.com/package/axios) for example.

Each time you generate your site, Eleventy will execute that script and treat the JSOn file returned by the API like a static one to generate your pages or views.

### Permalinks and URLs

By default, Eleventy will use the folders and files structure in your source directory to generate static files in your output folder.

- `./src/index.html` will generate `./dist/index.html` with `/` as the URL
- `./src/test.html` will generate `./dist/test/index.html` with `/test/` as the URL
- `./src/subdir/index.html` will generate `./dist/subdir/index.html` with `/subdir/` as the URL

This default behaviour can be changed by using a static or dynamic `permalink` variable in your content files or in your templates.

For example, to create a blog, you will need an index page at the following URL `/blog/index.html`. Your Nunjucks template `./src/pages/blog.njk` should have that as the value of the permalink key in its YAML front matter.

```text
---
permalink: "/blog/index.html"
---
```

You can also use variables to create dynamic `permalink` values. This could be in the YAML front matter of all your Markdown blogposts files.

```text
---
permalink: "/blog/{{ page.fileSlug }}/index.html"
---
```

If you have a collection to group all your team memebers but you do not need a detail page for each member, you can use `false` as the value of the `permalink` key. Eleventy will not generate detail pages. In most of those cases, you won't need a dedicated layour either.

```text
---
permalink: false
layout: false
---
```

We will see later that you will then need to use the `templateContent` key to display the content of your Markdown files.

#### Default values and directory data files

Instead of having to specify the same YAML front matter key / value apir for a bunch of files, Eleventy allows you to specify the same key/value pair for all the files in a directory by using JS or JSON [directory data files](https://www.11ty.io/docs/data-template-dir/).

If you want to specify the same key/value pair for `permalink` and `layout` for all of your blogposts, you can simply add an `./src/blog/blog.json`, `./src/blog/blog.11data.json` or `./src/blog/blog.11data.js` directory data file in the `./src/blog` directory and specify them there. Eleventy will apply those values to all the files in that directory or in subdiretories.

**./src/blog/blog.json** ou **./src/blog/blog.11tydata.json**
```json
{
  "layout": "layouts/blogpost.njk",
  "permalink": "blog/{{ page.fileSlug }}/index.html"
}
```

**./src/blog/blog.11tydata.js**
```js
module.exports = {
  layout: "layouts/blogpost.njk",
  permalink: "blog/{{ page.fileSlug }}/index.html"
};
```

-----------------------------
@TODO
-----------------------------

## 4. Templating avec Eleventy et Nunjucks

Eleventy vous permet de travailler avec différents langages de templating. Nunjucks de Mozilla est assez puissant et facile d'utilisation, c'est donc celui avec lequel je travaille d'habitude. La [documentation](https://mozilla.github.io/nunjucks/) étant assez bien faite, je ne vais pas rentrer ici dans le détail mais simplement réaliser les templates dont nous avons besoin pour notre projet de blog.

### Principaux tags de Nunjucks

Nunjucks possède trois grands types de tags

- tags de commentaires: `{# this is a comment #}`
- tags d'affichage: `{{ variable }}`
- tags de logique: `{% logique %}`

#### Tags de commentaires

Nunjucks possède un tag de commentaire: `{# Ceci est un commentaire #}`. Ceux-ci ne sont pas affichés lorsque le template est rendu.

#### Tags d'affichage, variables et propriétés

Ces tags vous permettent d'afficher des chaines de caractères, nombres, booléens, tableaux et objets dans vos templates. La plupart du temps, vous afficherez des variables créées par vous ou par Eleventy. Une notation pointée permet d'accéder aux propriétés de ces variables.

Exemples:

- `{{ "Hello World" }}`: affiche la chaîne de caractères "Hello World"
- `{{ data.title }}`: affiche la valeur de la clef `title` dans l'objet `data`
- `{{ 8 + 2 }}`: affiche `10`

#### Filtres

Les filtres sont essentiellement destiné à manipuler des chaînes de caractères, nombres, booléens, tableaux et objets tout en les affichants dans vos templates. Nunjucks possède de nombreux filtres par défaut. Voici quelques exemples.

- `{{ "this should be uppercase" | upper }}` produira en sortie `THIS SHOULD BE UPPERCASE`.
- `{{ [1,2,3,4,5] | reverse }}` produira en sortie `5,4,3,2,1` ce filtre est particulièrement utile combiné avec des classements par date dans Eleventy.
- `{{ collections.blogposts | length }}` va afficher le nombre d'éléments présents dans votre collection de blogposts.

##### Filtres personnalisés dans Eleventy

Eleventy vous permet de créer vos propres filtres en JavaScript à l'aide du fichier `.eleventy.js` ces filtres peuvent ensuite être utilisés dans le languge de templating que vous aurez choisi. Nunjucks ne possède pas de filtre permettant de formatter les dates, nous pouvons donc en créer facilement un dans Eleventy à l'aide de la librairie [`moment.js`](https://momentjs.com/).

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

Lorsque vous devrez afficher des données, qu'elles proviennent d'API ou de fichiers Markdown, vous devrez parcourir des tableaux ou des dictionnaires avec des boucles `for`. Voici par exemple comment afficher dans une liste les titres et introduction de tous vos blogposts.

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

Lorsqu'un template parent est étendu par un template enfant, les variables définies dans le template enfant sont accessibles dans le template parent. Outre `{% extends %}` et `{% include %}`, Nunjucks vous permet d'utiliser des [macros](https://mozilla.github.io/nunjucks/templating.html#macro) qui sont de petits bouts de code réutilisables auxquels des variables peuvent être passées. Eleventy possède un concept similaire avec la possibilités de créer des [shortcodes](https://www.11ty.io/docs/shortcodes/).

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

Pour la page d'archive de notre blog, nous allons utiliser la fonction de pagination d'Eleventy. Celle fonctionne en spécifiant quelles sont les données à paginer, combien d'éléments doivent être affichés par page et quel alias doit être utilisé pour les données une fois paginées.

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
