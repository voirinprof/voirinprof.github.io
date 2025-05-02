---
title: "Vue.js 3"  
permalink: /en/vuejs-intro/  
lang: en  
date: 2025-04-28T22:00:00+00:00  
toc: true  
toc_sticky: true  
toc_label: "In this page"  
typenav: client  
---

> **_NOTE:_** This page provides an overview of the Vue.js framework, but it can show you the difference between classic JavaScript and Vue.js. I use Vue.js in geospatial applications because it is very easy to use and has many extensions for developing spatial applications (e.g., leaflet, openlayers, ...). 

## Introduction to Vue.js 3

This page introduces the **basics of Vue.js 3**, a JavaScript framework used for building interactive user interfaces. Vue is easy to get started with and powerful for creating modern web applications. It’s worth noting that there are other frameworks like ReactJS (used in GeoNode's MapStore) and Angular.

Vue.js changes the way we develop applications in geospatial contexts. Traditionally, WebGIS tools are developed around jQuery and classic JavaScript. These kinds of frameworks require developers to adjust their workflows. However, this leads to more reactive applications. It is important to have:
- Basic knowledge of HTML, CSS, and JavaScript
- Node.js installed (recommended version: ≥ 16)

## Project Structure

```bash
.
├── index.html           # HTML entry point
├── package.json         # Dependencies and scripts
├── vite.config.js       # Vite configuration
└── src/
    ├── main.js          # Vue app entry point
    ├── App.vue          # Main component
    └── components/
        └── HelloWorld.vue  # Example of a component
```

As a Vue.js project is based on node.js, there will be a package.json file listing the libraries to be used in the project.

In general, the project will include _reusable_ components with a specific structure (containing scripts, HTML layout, and styling).

## Basic Concepts

### Create a Vue 3 Application

```javascript
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

### Template, Script, Style

A `.vue` file is composed of 3 parts:

```vue
<template>
  <h1>{{ message }}</h1>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello Vue 3!'
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

- `v-if`: display an element if a condition is true
- `v-for`: repeat an element for each item in a list
- `v-model`: two-way binding with a form field
- `@click`: handle a user click

Example:

```html
<input v-model="name" placeholder="Your name">
<button @click="greet">Say hello</button>
<p v-if="name">Hello, {{ name }}!</p>
```

### Components

A component is a reusable block with its own data and behavior.

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

Then in `App.vue`:

```vue
<HelloWorld name="Alice" />
```

---

## Useful Resources

- [Official Vue.js 3 Documentation](https://vuejs.org/)
- [Vue 3 Tutorials (Vue Mastery)](https://www.vuemastery.com/)
- [Vite (recommended build tool)](https://vitejs.dev/)