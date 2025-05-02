---
title: "Vue.js 3"
permalink: /fr/vuejs-intro/
lang: fr
date: 2025-04-28T22:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: client
---

> **_NOTE:_** Cette page survole le framework Vue.js, toutefois cela peut vous montrer la différence entre JavaScript classique et Vue.js. J'utilise Vue.js en géomatique car elle est très facile à utiliser et elle dispose de beaucoup d'extensions pour développer des applications spatiales (ex: leaflet, openlayers, ...). 

## Introduction à Vue.js 3

Cette page vous permet de découvrir les **bases de Vue.js 3** qui est un framework JavaScript permettant la construction d'interfaces utilisateur interactives. Vue est facile à prendre en main et puissant pour créer des applications web modernes. On peut noter qu'il existe d'autres frameworks comme ReactJS (présent dans le MapStore de GeoNode) et Angular.

Vue.js change la manière de développer les applications en géomatique. Traditionnellement les outils WebGIS sont développés autour de jQuery et du JavaScript classique. Ce genre de frameworks oblige les développeurs à changer leurs habitudes. Toutefois c'est pour mettre en place des applications plus réactives. Il faut noter qu'il est important de disposer de :
- Connaissances de base en HTML, CSS, et JavaScript
- Node.js installé (version recommandée : ≥ 16)

## Structure d'un projet

```bash
.
├── index.html           # Point d'entrée HTML
├── package.json         # Dépendances et scripts
├── vite.config.js       # Configuration Vite
└── src/
    ├── main.js          # Démarrage de l'application Vue
    ├── App.vue          # Composant principal
    └── components/
        └── HelloWorld.vue  # Exemple de composant
```

Comme un projet Vue.js est basé sur node.js, il y aura la présence d'un fichier package.json listant les librairies à utiliser dans le projet.

En général, le projet va comprendre des composants _réutilisables_ avec une structure particulière (comprenant du script, un agencement HTML et un style).

## Concepts de base

### Créer une application Vue 3

```javascript
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

### Template, Script, Style

Un fichier `.vue` est composé de 3 parties :

```vue
<template>
  <h1>{{ message }}</h1>
</template>

<script>
export default {
  data() {
    return {
      message: 'Bonjour Vue 3 !'
    }
  }
}
</script>

<style>
h1 {
  color: teal;
}
</style>
```

### Directives

- `v-if`: afficher un élément si une condition est vraie
- `v-for`: répéter un élément pour chaque item d'une liste
- `v-model`: liaison bidirectionnelle avec un champ de formulaire
- `@click`: gérer un clic utilisateur

Exemple :

```html
<input v-model="name" placeholder="Votre nom">
<button @click="saluer">Dire bonjour</button>
<p v-if="name">Bonjour, {{ name }} !</p>
```

### Composants

Un composant est un bloc réutilisable avec ses propres données et comportements.

```vue
<!-- components/HelloWorld.vue -->
<template>
  <h2>Hello {{ name }}</h2>
</template>

<script>
export default {
  props: ['name']
}
</script>
```

Puis dans `App.vue` :

```vue
<HelloWorld name="Alice" />
```

---

## Ressources utiles

- [Documentation officielle de Vue.js 3](https://vuejs.org/)
- [Tutoriels Vue 3 (Vue Mastery)](https://www.vuemastery.com/)
- [Vite (build tool recommandé)](https://vitejs.dev/)