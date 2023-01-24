* [Notes vrac](#notes-vrac)
   * [vue.js vs. angular/react](#vuejs-vs-angularreact)
   * [vue.js et typescript](#vuejs-et-typescript)
   * [Keypoints](#keypoints)
   * [bootstrap d'un projet vue.js avec npm init](#bootstrap-dun-projet-vuejs-avec-npm-init)

# Notes vrac

**Contexte** = mai 2022, j'essaye vue.js en tant que framework simple pour avoir des visus propres et modulaires sur mes projets persos.

## vue.js vs. angular/react

TL;DR : vue.js plus adapté aux devs js inexpérimentés sur de petits projets déjà existants → c'est tout à fait mon cas.

https://www.codeinwp.com/blog/angular-vs-vue-vs-react/

> Vue is generally more suited to smaller, less complex apps and is easier to learn from scratch compared to React. Vue
> can be easier to integrate into new or existing projects and many feel its use of HTML templates along with JSX is an
> advantage.
>
> Overall, Vue might be the best choice if you’re a newer developer and not as familiar with advanced JavaScript
> concepts, while React is quite well suited for experienced programmers and developers who have worked with
> object-oriented JavaScript, functional JavaScript, and similar concepts.
>
> In most cases, you probably wouldn’t be deciding between only Angular and Vue. They are vastly different libraries
> with very different feature sets and learning curves. Vue is the clear choice for less experienced developers, and
> Angular would be preferred for those working on larger apps.

## vue.js et typescript

https://vuejs.org/about/faq.html

> While Vue itself is implemented in TypeScript and provides first-class TypeScript support, it does not enforce an opinion on whether you should use TypeScript as a user.


## Keypoints

https://vuejs.org/guide/introduction.html#what-is-vue

En gros, vue.js simplifie la mise à jour du HTML via js :

- Declarative Rendering: Vue extends standard HTML with a template syntax that allows us to declaratively describe HTML output based on JavaScript state
- Reactivity: Vue automatically tracks JavaScript state changes and efficiently updates the DOM when changes happen.

Plusieurs façons d'utiliser vue.js :

- Enhancing static HTML without a build step
- Embedding as Web Components on any page
- Single-Page Application (SPA)
- Fullstack / Server-Side-Rendering (SSR)
- Jamstack / Static-Site-Generation (SSG)
- Targeting desktop, mobile, WebGL or even the terminal

**Progressive Framework** = on peut commencer sur des besoins simples, mais gagner en puissance avec le temps.

**SFC = Single-File Component** = décrire l'intégralité d'un composant (HTML+CSS+JS) en un seul fichier :

- _In most build-tool-enabled Vue projects, we author Vue components using an HTML-like file format called Single-File Component (also known as `*.vue` files, abbreviated as SFC)._
- _A Vue SFC, as the name suggests, encapsulates the component's logic (JavaScript), template (HTML), and styles (CSS) in a single file._

Attention que les SFC ne sont disponibles que si on utilise un build-tool :
> SFC is a defining feature of Vue, and is the recommended way to author Vue components if your use case warrants a build setup.

Du coup, sur le principe, on dirait qu'on créée des composants, de deux façons possibles :
- **Options API** = on définit déclarativement un objet, avec des magic-properties (data, methods, mounted, ...)
- **Composition API** = c'est un peu plus des free-floating objects ou fonctions.

Les deux sont deux APIs différentes pour parler au core unique de vue.js Options API est un peu plus simple, et beginner-friendly : le composant est un objet, qu'on peut manipuler avec "this"

- For learning purposes, go with the style that looks easier to understand to you.
- For production use: Go with Options API if you are not using build tools, or plan to use Vue primarily in low-complexity scenarios, e.g. progressive enhancement.

## bootstrap d'un projet vue.js avec npm init

À noter que dans le répertoire créé par `npm init`, il y a un fichier caché `.gitignore` (e.g. pour gitignorer `node_modules/` ou `dist/`).

Analyse du projet installé automatiquement par "npm init" :

- définit des sous-composants dans `src/components/`
    - `components/HelloWorld.vue`
    - `components/TheWelcome.vue`
    - `components/WelcomeItem.vue`
- définit une `src/App.vue` qui les utilise :
- chaque composant semble suivre le même pattern :
    - on commence par un tag `<script setup>` si on veut importer d'autres composants, ou définir des données
    - puis, on a un tag `<template>` pour définir le HTML du composant
    - enfin, on a un tag `<style>` pour styliser le template
- à noter que certains composants sont des icônes !
    - leur HTML est une simple balise SVG qui encode l'icône
- le composant principal est celui importé par `main.js` :
    ```js
    import App from './App.vue'
    createApp(App).mount('#app')
    ```
- le template du composant principal définit le squelette de la page :
    ```
    <template>
      <header>
        <div class="wrapper">
          <HelloWorld msg="You did it!" />
        </div>
      </header>
      <main>
        <TheWelcome />
      </main>
    </template>
    ```
- le p'tit bout de code ci-dessus montre :
    - qu'on peut utiliser un composant comme du pseudo-html
    - qu'on peut passer des données au composant

À noter que pour servir `dist/`, je dois passer par un serveur web (plutôt que de charger directement les fichiers avec le browser via `file:///`) à cause de la politique CORS :

```
python3 -m http.server --directory=dist/ 8787
```

**CONCLUSION** = si je me donne la peine de regarder d'un peu plus près, ce petit bout de code a déjà plein de trucs intéressants !
