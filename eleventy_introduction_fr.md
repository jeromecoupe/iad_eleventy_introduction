# Eleventy (11ty) par Zach Leatherman

## 1. Introduction

- Ecrit en Node donc permet d'utiliser tout l'écosystème NPM
- Simple à configurer et à utiliser
- Performant, rapide et flexible

## 2. Installation and configuration

### Installation

- installation locale vs globale
- commandes principales

### Configuration

- fichiers de configuration principaux (.eleventy.js et .eleventyignore)
- intégration dans un pipeline de build (scripts NPM ou Gulp)

## 3. Définir et structurer vos données

- deux types de sources de données

#### Collections (markdown et YAML front matter)

- Markdown et YAML front matter
- collection API avec .eleventy.js
- valeurs par defaut et fichiers de directory

#### Data files (JS ou JSON)

- fichiers statiques
- fichiers dynamiques connectés à un endpoint API

## 4. Templating avec Nunjucks

### Les bases de Nunjucks

- tags de logique
- tags d'affichage
- layouts et héritage
- filtres
- macros

### Collections

- accessibles à partir de l'object collections
- une collection spéciale: collections.all

### Data files

- données accessibles en utilisant le nom de fichier comme key
- exemple avec des valeurs en key / value pairs (site.js)
- exemple avec un tableau d'objets en provenance d'une API

### Pagination

- listes paginées
- pages de détail et pagination
