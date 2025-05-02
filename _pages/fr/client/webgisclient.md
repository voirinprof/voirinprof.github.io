---
title: "QGis et Python"
permalink: /fr/webgis/
lang: fr
date: 2025-04-25T10:00:00+00:00
toc: true
toc_sticky: true
toc_label: "Dans cette page"
typenav: client
---


> **_NOTE:_** Pour plus de détails sur Vue.js, consultez la [documentation officielle Vue.js 3](https://vuejs.org/).

## Introduction

**Vue.js 3** est un framework JavaScript moderne, rapide et léger pour construire des interfaces web dynamiques.  
En géomatique, Vue.js permet de développer des applications cartographiques interactives, intuitives et performantes, notamment en intégrant des bibliothèques comme **Leaflet**, **OpenLayers** ou **MapLibre**.

---

## Pourquoi utiliser Vue.js en géomatique ?

- **Simplicité de prise en main** : syntaxe accessible pour les débutants.
- **Réactivité** : gestion automatique de la mise à jour de l'interface lorsque les données changent.
- **Composants réutilisables** : idéal pour modulariser des éléments cartographiques (cartes, panneaux latéraux, légendes...).
- **Compatibilité avec l'écosystème SIG web** : facile à intégrer avec des bibliothèques de cartographie JS.
- **Performance** : architecture légère et rapide, même pour de grosses applications géospatiales.

---

## Prérequis pour commencer

Avant d'utiliser Vue.js 3 pour des projets de géomatique :

- Avoir des bases solides en **HTML**, **CSS**, **JavaScript**.
- Comprendre les notions de **REST API** et de **services cartographiques** (WMS, WFS, WMTS).
- Avoir une bonne connaissance des bibliothèques de cartographie web.

---

## Mise en place rapide d'un projet Vue.js 3

### Installation avec Vite (recommandé)

```shell
npm create vite@latest mon-projet-vue -- --template vue
cd mon-projet-vue
npm install
npm run dev
```

Cela vous crée une application Vue.js 3 minimaliste prête à être développée.

---

## Intégrer une carte avec Vue.js

Voici un exemple d'intégration de **Leaflet** dans un composant Vue :

### 1. Installer Leaflet

```shell
npm install leaflet
```

### 2. Exemple de composant `Map.vue`

```vue
<template>
  <div id="map" style="height: 500px;"></div>
</template>

<script setup>
import { onMounted } from 'vue'
import L from 'leaflet'

onMounted(() => {
  const map = L.map('map').setView([48.8566, 2.3522], 13)

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap contributors'
  }).addTo(map)
})
</script>

<style>
#map {
  width: 100%;
  height: 500px;
}
</style>
```

---

## Exemple d'architecture d'une application géomatique Vue.js

```
src/
├── components/
│   ├── CartePrincipale.vue
│   ├── BarreDeRecherche.vue
│   └── ListeResultats.vue
├── services/
│   └── api.js (accès aux services WFS, REST, etc.)
├── views/
│   └── Home.vue
├── App.vue
└── main.js
```

Chaque composant peut représenter une partie fonctionnelle de votre application (carte, formulaire de recherche d'adresse, résultats dynamiques...).

---

## Cas d'usage en géomatique

- **Visualisation de couches spatiales** issues d'un serveur WMS/WFS
- **Recherche et affichage d'adresses** via un service de géocodage
- **Calcul et affichage de routes** avec des API de routage (OSRM, GraphHopper)
- **Affichage d'analyses spatiales** en direct (zones d'influence, isochrones)

---

## Bonnes pratiques

- **Séparer la logique métier** (API, géoservices) des composants d'affichage.
- **Utiliser des stores d'état** (Pinia ou Vuex) pour centraliser les données cartographiques partagées.
- **Optimiser les performances** (limiter les rechargements inutiles de la carte).
- **Utiliser des bibliothèques spécialisées** (vue-leaflet, vue3-openlayers) pour aller plus vite.

---

## Ressources utiles

- [Documentation Vue.js 3](https://vuejs.org/)
- [Vite - Build tool rapide pour Vue.js](https://vitejs.dev/)
- [Leaflet.js - Cartes interactives](https://leafletjs.com/)
- [OpenLayers - Framework SIG web avancé](https://openlayers.org/)
- [Pinia - Store officiel pour Vue 3](https://pinia.vuejs.org/)