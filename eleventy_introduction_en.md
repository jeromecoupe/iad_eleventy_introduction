# Eleventy (11ty) by Zach Leatherman

## 1. Introduction

[Eleventy](https://www.11ty.io) is a Static Site Genertor created and maintained by [Zach Leatherman](https://www.zachleat.com/). Eleventy allows you to develop websites based on templates and data or content files (YAML / Markdown / HTML / JSON / JS) in a source directory. Based on those source files, Eleventy will generate a fully functional static site in a destination folder. You will then be able to deploy that site on any web server capable of serving static files.

The [stated goal of Eleventy](https://www.11ty.io/docs/) is to be an alternative to [Jekyll](https://jekyllrb.com/), written in Node rather than in Ruby. Like Jekyll, 11ty is a very approachable and simple SSG to use, once the basic principles are well understood.

Node being quite fast, Eleventy is a performant SSG. It is also very flexible. Since it is written in Node, 11ty allows you to use the NPM ecosystem to extend its functionalities. You can also pick your favourite in a [long list of templating languages](https://www.11ty.io/docs/languages/). In this course we will use [Nunjucks](https://mozilla.github.io/nunjucks/) by [Mozilla](https://www.mozilla.org) for all code samples.

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

At this point, Eleventy will only create a `node_modules` folder and install itself, nothing more. To see it in action, we will have to create at least one file. Let's create a basic `index.html` file that Eleventy will copy in the default `_site` destination folder.

Once we have created that file, we can learn some basic Eleventy commands:

- `npx eleventy`: to run Eleventy
- `npx eleventy --serve`: to run Browsersync and have a local web server that will reload the site in your browser when the site changes
- `npx eleventy --help`: to explore the list of available commands and flags

Now if we type `npx eleventy` in our terminal, Eleventy will create a `_site` folder and copy our `index.html` file into it. Pretty impressive, right?

Let's configure Eleventy to better suit our needs.

### Configuration

We will make a basic project architecture and configure Eleventy by creating a `.eleventy.js` configuration file at the root of our project.

- Remove Eleventy's default destination `./site` folder.
- Create a `./src` folder and move `index.html` into it.
- Create an `.eleventy.js` file at the root of the project.

Let's start by specifying source and destination directories for Eleventy:

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

Now when we run the `npx eleventy` command in our terminal from the root of our project, Eleventy will generate a `./dist` folder and will copy our trusty `index.html` file to it.

### Tell Eleventy to copy some folders and files

We also can use this configuration file to tell Eleventy to copy any file or folder from the source directory to the destination directory. In order to instruct Eleventy to do so, we are going to use [`addPassthroughCopy`](https://www.11ty.io/docs/copy/). Good candidates to copy are static assets like images and font files.

- Create a `./src/assets/img/` directory and drop a few optimized images in there.
- Create a `./src/assets/fonts/` directory and drop a few font files in there.
- Create a `./src/assets/js/` directory and drop a JavaScript file in there.
- Create a `./src/assets/css/` directory and drop a CSS file in there.

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

By default, Eleventy will ignore the `node_modules` directory as well as the folders, files and globs specified in your `.gitignore` file.

We can also create a `.eleventyignore` file at the root of our project and specify a file, directory or glob pattern per line to explicitly tell Eleventy to ignore all matching files and directories. I have painfully learned that you always want to be as explicit as possible, and that's true for many other things than code, actually. Let's do this.

**.eleventyignore**
```txt
node_modules/
dist/
```

### Assets pipeline and build tools

Eleventy doesn't offer an asset pipeline or a build tool by default. I always almost use Eleventy with build tools, whether with NPM scripts, Gulp, Webpack or any other alternatives.

When you start using build tools to create an assets pipeline, you will likely have to modify your `addPassthroughCopy` and ignore assets directories that will not need to be handled by Eleventy because your build tools and scripts will take care of them and generate what you need in your `./dist` directory.

If, for example, you are using NPM scripts to build your CSS from Sass files or if you use Webpack to handle your JavaScript pipeline, you will have to make the following modifications:

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

Eleventy will now completely ignore the `./src/assets/scss/` and `./src/assets/js/` directories, while build tools and scripts will generate the required outputs in your `./dist/` directory.

Personally, I use Gulp combined with Webpack for most of my projects. Eleventy can very easily be integrated in this kind of workflow.

## 3. Define and structure your data

Eleventy allows you to work with two data sources:

1. **Markdown files** (for the main content) and YAML front matter (for the rest of the data structure) that can easily be turned into collections (more on that later).
2. **JSON and/or JS data files** that can either be static or dynamic (fetched from an API).

These two types of data sources are not mutually exclusive and are, in fact, used simultaneously in most projects.
Let's dive in.

### Collections

Collections in Eleventy allow you to group content items in interesting ways and to use them in your templates.

#### Markdown and YAML front matter

Markdown files coupled with a YAML front matter allow you to use text files as structured data sources. It's a staple feature of most SSG out there.

The Markdown part of the file generally represents the main content of your data and is generally converted to HTML. The YAML front matter allows you to create a data structure with different types of data (strings, arrays, objects, etc.).

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

Let's say you also need to model a data structure for a team. Each team member could be represented by a file like this one:

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

Jérôme Coupé is a looney front-end designer and teacher from Brussels, Belgium. When not designing or coding, he might have been seen downtown, drinking a few craft beers with friends.
```

#### Collection API

For Eleventy to group those files in an array that will allow you to manipulate it in your templates, you have to create a [collection](https://www.11ty.io/docs/collections/). Any content item can be part of one or more collections.

To create a collection, you can assign the same `tag` to various content items. Personally, I would much rather use the collection API and our trusty `.eleventy.js` file.

This API offers you [different methods to declare your collections](https://www.11ty.io/docs/collections/#collection-api-methods) that each have their use. My favourite and most used one by far is `getFilteredByGlob(glob)` that allows you create a collection from all files matching a defined glob pattern.

If all your Markdown files for your blogposts are in a `./src/blog/` directory, grouping them into a collection is quite easy. You have to add the following code to your `.eleventy.js` configuration file. While we are at it, we will also add our `team` collection.

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

- `inputPath`: the full path to the source input file (including the path to the input directory).
- `fileSlug`: Mapped from the input file name, useful for permalinks. Read more about [`fileslug`](https://www.11ty.io/docs/data/#fileslug).
- `outputPath`: the full path to the output file to be written for this content.
- `url`: url used to link to this piece of content. In general, this is based on the `permalink` key defined for your content items.
- `date`: the resolved date used for sorting. Read more about [Content Dates](https://www.11ty.io/docs/dates/) in the documentation.
- `data`: all data for this piece of content. Contains the keys/values defined in the YAML front matter and the data inherited from layouts.
- `templateContent`: the rendered content of this template. Does not include layout wrappers.

#### Sort and filter your collections

When you create a collection in Eleventy, its items are automatically sorted in ascending order using:

1. The date specified in the filename or in the YAML front matter of the source file or the creating date on the filesystem as a fallback.
2. If some files have an identical date, the full path of the file (including the filename) will be taken into account as well.

If what you need is to sort your collection items by date, you are covered. You can also use the `reverse` filter in Nunjucks if needed.

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

If you need to filter a collection to exclude some data, you can use the JavaScript `filter` method. You can for example only include the blogposts that do not have a `draft` key set to `true` in their front matter and that have a publication date less recent than site generation date.

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

### Data files (JS or JSON)

Aside from collections, the other data source you can use with Eleventy are data files. Those can be static or dynamic (fetched from an API).

By default, these files have to be stored in the `./src/_data/` directory. This can be modified using the `.eleventy.js` configuration file.

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

Static data files are JSON or JS files containing key/value pairs.

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

Those values can be accessed in your templates by using the filename as a key. Data contained in a `./src/_data/site.js` file will thus be accessible in your templates using the `site` variable.

#### Dynamic data files

Because data files can be JavaScript files, nothing is preventing you from [connecting to an API](https://www.webstoemp.com/blog/headless-cms-graphql-api-eleventy/) in one of those files by using [`node-fetch`](https://www.npmjs.com/package/node-fetch) or [`axios`](https://www.npmjs.com/package/axios) for example.

Each time you generate your site, Eleventy will execute that script and treat the JSON file returned by the API like a static one to generate your pages or views.

### Permalinks and URLs

By default, Eleventy will use the folders and files structure in your source directory to generate static files in your output folder.

- `./src/index.html` will generate `./dist/index.html` with `/` as the URL.
- `./src/test.html` will generate `./dist/test/index.html` with `/test/` as the URL.
- `./src/subdir/index.html` will generate `./dist/subdir/index.html` with `/subdir/` as the URL.

This default behaviour can be changed by using a static or dynamic `permalink` variable in your content files or in your templates.

For example, to create a blog, you will need an index page at the following URL `/blog/index.html`. Your Nunjucks template `./src/pages/blog.njk` should have that as the value of the `permalink` key in its YAML front matter.

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

If you have a collection to group all your team members but you do not need a detail page for each member, you can use `false` as the value of the `permalink` key. Eleventy will not generate detail pages. In most of those cases, you won't need a dedicated `layout` either.

```text
---
permalink: false
layout: false
---
```

We will see later that you will then need to use the `templateContent` key to display the content of these Markdown files.

#### Default values and directory data files

Instead of having to specify the same YAML front matter key / value pair for a bunch of files, Eleventy allows you to specify the same key/value pair for all the files in a directory by using JS or JSON [directory data files](https://www.11ty.io/docs/data-template-dir/).

If you want to specify the same key/value pair for `permalink` and `layout` for all of your blogposts, you can add a `./src/blog/blog.json`, `./src/blog/blog.11data.json` or `./src/blog/blog.11data.js` directory data file in the `./src/blog` directory and specify them there. Eleventy will apply those values to all the files in that directory or in its subdirectories.

**./src/blog/blog.json** or **./src/blog/blog.11tydata.json**
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

## 4. Templating with Eleventy and Nunjucks

Eleventy allows you to work with several templating languages. Nunjucks from Mozilla is powerful and easy to use, so that's generally my default choice. The [documentation](https://mozilla.github.io/nunjucks/) is quite well done so we don't need to cover everything. We will learn enough to create the templates we need for our small blog project, and you can expand your knowledge on your own later on.

### Main Nunjucks tags

Nunjucks has three main types of tags:

- comments: `{# this is a comment #}`
- display: `{{ variable }}`
- logic: `{% logic %}`

#### Comment tags

This one is pretty easy: `{# This is a comment #}`. Comments never appear in the rendered code.

#### Display tags: variables and properties

This tags allow you to display variables like strings, numbers, booleans, arrays and objects in your templates. Most of the time, you will display variables created by you or by Eleventy when it runs. You can access properties using dot syntax, like in JavaScript. Those tags also allow you to perform maths operations or string concatenation.

Examples:

- `{{ "Hello World" }}`: displays the string "Hello World"
- `{{ site.title }}`: displays the value of the `title` key of the `site` object
- `{{ site.title ~ " - is an awesome site" }}` will concatenate the value of the key `title` of the site object with the string next to it
- `{{ 8 + 2 }}`: displays `10`

#### Filters

Filters are essentially devoted to manipulate strings, numbers, booleans, arrays and objects while displaying them in your templates. Nunjucks makes a bunch of [built-in filters](https://mozilla.github.io/nunjucks/templating.html#builtin-filters) available. Here are some examples.

- `{{ "this should be uppercase" | upper }}` will output `THIS SHOULD BE UPPERCASE`.
- `{{ [1,2,3,4,5] | reverse }}` will output `5,4,3,2,1`. This filter is quite useful when combined with date sorted collections in Eleventy.
- `{{ collections.blogposts | length }}` will display the number of items in your `blogposts` collection.

##### Custom filters in Eleventy

Eleventy allows you to create your own filters using JavaScript and the `.eleventy.js` configuration file. these filters can then be used in most templating languages you choose to use.

For example, Nunjucks does not have a built-in date formatting filter. We can easily create one in Eleventy using the popular [`moment.js`](https://momentjs.com/) library.

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

Once created, we can use that filter in our templates like any native one.

```njk
<p><time datetime="{{ page.date | date('YYYY-MM-DD') }}">{{ page.date | date("MMMM Do, YYYY") }}</time></p>
```

#### Logic tags

These tags allow you to execute operations and can be used to create variables, for loops, control structures, etc.

##### Assign variables

The code below will assign all blogposts in the `blogposts` collection to an `allBlogposts` variable and use the built-in `reverse` filter to sort them by reverse date order.

```njk
{% set blogposts = collections.blogposts | reverse %}
```

##### Control structures

Nunjucks will allow you to use traditional control structures like `if` and `else` as well as comparison operators like `===`, `!==`, `>` or logical operators like `and`, `or` and `not`.

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

##### `for` loop

When you have to display data, whether they come from an API or from Markdown files, you will have to walk through arrays or objects using `for` loops. Let's display title and introductions for all our blogposts in an HTML list.

```njk
{% set blogposts = collections.blogposts | reverse %}
{% for entry in blogposts %}
  {% if loop.first %}<ul>{% endif %}
    <article>
      <h2><a href="{{ entry.url }}">{{ entry.data.title }}</a></h2>
      <p>{{ entry.data.intro }}</p>
    </article>
  {% if loop.last %}</ul>{% endif %}
{% else %}
  <p>No blogposts found</p>
{% endfor %}
```

You will notice that we can use `loop.first` and `loop.last` to only spit out the `<ul>` and `</ul>` in the first and last iterations of the loop, respectively. In Nunjucks, an `else` clause is executed when no data is returned, which allows us to display a warning in HTML.

### Layouts, includes, macros and shortcodes

On top of offering you an `{% include %}` tag, Nunjucks uses template inheritance as its layout model with `{% extends %}`. This allows you to define blocks with `{% block blockname %}` in a template and then to override the content of those blocks with child templates that extends the parent one. These chains of templates can be as long as you wish.

Includes as well as template inheritance can be used with Eleventy. The only quirk is that templates to extend as well as template to include must all be located in the includes directory specified in your `.eleventy.js` configuration file. By default, this directory is `_includes` and the path you specify in your config file is relative to your source directory.

**.eleventy.js**
```js
module.exports = function(eleventyConfig) {
  // override default config
  return {
    dir: {
      input: "src",
      output: "dist",
      // path is relative to the input directory
      // "_includes" is the default value
      includes: "_includes"
    }
  };
};
```

When a parent template is extended by a child template, variables defined in the child template are accessible in the parent template.

Apart from `{% extends %}` and `{% include %}`, Nunjucks allows you to use [macros](https://mozilla.github.io/nunjucks/templating.html#macro) that are reusable code snippets to which you can pass variables. Eleventy has a similar concept with the [shortcodes](https://www.11ty.io/docs/shortcodes/).

Let's come back to our project and create the templates we need using everything we have learned so far.

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

Here is an included file for the footer

**./src/_includes/partials/sitefooter.njk**
```njk
<div class="c-sitefooter">
  <p>&copy; {{ site.buildTime | date("Y") }} - La casa productions</p>
</div>
```

As far as blogposts go, we need a special layout which will extend our base layout. This blogpost layout will be used by all the Markdown files in our collection to generate detail pages. This layout is the one specified for all our blogposts using a directory data file (see above).

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

Here is an example of template for the about page, where will will display a list of our team members. It extends our base layout and sets several variables that will be available in the base layout.

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

### Pagination

For the archive page of our blog, we will use the [pagination](https://www.11ty.io/docs/pagination/) provided by Eleventy. Pagination specifies what `data` must be paginated (this can be any iterable), how many items must be displayed per page with `size` and which `alias` must be used for the paginated data.

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
    {% if loop.last %}</ul>{% endif %}
  {% else %}
    <p>No blogpost found</p>
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

The pagination function in Eleventy is much more powerful than it seems at first sight. If the data for our blogposts came as a big JSON file returned by an API, we could use the `./src/_data/_blogposts.js` file as a data source and use the same pagination function to generate all the detail pages with a single file. In order to accomplish that, we have to specify a value of `1` for `size` and to craft a dynamic permalink pattern that corresponds to the URLs we want.

Here is a simplified template we could use:

**./src/pages/blogpost_entry.njk**
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

### Workshop exercise

Start with the provided static templates to create a fully functional blog.

- Configure Eleventy and create a date and a limit filter
- Generate a homepage with the list of the latest 6 blogposts using the `limit` filter
- Generate a paginated archive for all our blogposts
- Generate detail pages for blogposts
- Create an about page listing team members and providing contact details
- Create a navigation using a data file (home, blog, about). The navigation must highlight the current section.
