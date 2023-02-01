* [Templates](#templates)
   * [mustache tag](#mustache-tag)
   * [v-xxxx directives](#v-xxxx-directives)
   * [v-bind](#v-bind)
   * [synthèse du binding](#synthèse-du-binding)
   * [écouter des DOM events](#écouter-des-dom-events)
* [Reactivity](#reactivity)
   * [data](#data)
   * [methods](#methods)
   * [le rendu du DOM n'est pas instantané](#le-rendu-du-dom-nest-pas-instantané)
   * [Synthèse](#synthèse)
* [Props vs data](#props-vs-data)
* [Communication entre composants](#communication-entre-composants)

# Templates

https://vuejs.org/guide/essentials/template-syntax.html

## mustache tag

```html
<span>Message: {{ msg }}</span>
```

> The mustache tag will be replaced with the value of the msg property from the corresponding component instance. It will also be updated whenever the msg property changes.

**NDM =** RAS, si ce n'est 1. que `msg` sera réactif (le DOM sera mis à jour quand `msg` sera modifié), et 2. que `msg` peut être indifféremment une *data* (ou une *computed*) appartenant au composant, ou bien une *prop* passée par le parent.

## v-xxxx directives

```html
<p>Using v-html directive: <span v-html="supercontent"></span></p>
```

> The v-html attribute you're seeing is called a directive. Directives are prefixed with v- to indicate that they are special attributes provided by Vue, and as you may have guessed, they apply special reactive behavior to the rendered DOM. Here, we're basically saying "keep this element's inner HTML up-to-date with the supercontent property on the current active instance."

Directives = commencent par `v-`, et triggent le re-rendu du DOM quand leur expression change.

Les arguments des directives (e.g. l'id d'attribut pour `v-bind` ou l'event écouté pour `v-on`) peuvent être définis dynamiquement :

```html
<a v-on:[eventName]="doSomething"> ... </a>

<!-- shorthand -->
<a @[eventName]="doSomething">
```

Les directives peuvent avoir des *modifiers* :

```html
<form @submit.prevent="onSubmit">...</form>
```

![directive syntax](./vue.js_DIRECTIVE_SYNTAX.png)

## v-bind

`v-bind` permet d'utiliser une prop/data/computed du composant comme ATTRIBUT d'une balise (plutôt que comme CONTENU)

```html
<div v-bind:id="dynamicId">some irrelevant content</div>
```

> Because v-bind is so commonly used, it has a dedicated shorthand syntax:

```html
<div :id="dynamicId">some irrelevant content</div>
```

On peut binder plusieurs attributs d'un coup :

```html
<div v-bind="objectOfAttrs">some irrelevant content</div>
```

On peut binder des expressions plutôt que de simples variables :

```
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}
<div :id="`list-${id}`"></div>
```

> Caveat : Each binding can only contain one single expression. An expression is a piece of code that can be evaluated to a value. A simple check is whether it can be used after return.

## synthèse du binding

En résumé :

- **{{ mustache }}** = utiliser une prop/data en tant que CONTENU `textContent` d'une balise (donc text-only)
- **v-html** = utiliser une prop en tant que CONTENU `innerHTML` d'une balise (attention au XSS !)
- **v-bind** (aka `:`) = utiliser une prop comme contenu d'un ATTRIBUT d'une balise
- dans les trois cas, il y a binding : quand la prop/data référencée change, le rendu dans le DOM sera mis à jour

## écouter des DOM events

`v-on` permet d'appeler une méthode du composant comme callback sur un event (un event du DOM, ou un event émis par un enfant)

```html
<a v-on:click="doSomething"> ... </a>

<!-- shorthand -->
<a @click="doSomething"> ... </a>
```

# Reactivity

https://vuejs.org/guide/essentials/reactivity-fundamentals.html

## data

> With Options API, we use the data option to declare reactive state of a component. The option value should be a function that returns an object. Vue will call the function when creating a new component instance, and wrap the returned object in its reactivity system. Any top-level properties of this object are proxied on the component instance (this in methods and lifecycle hooks):

L'état du composant est stocké dans son attribut `data`, cet état est réactif.

La réactivité (binding au DOM : on re-rend le DOM quand la propriété est modifiée) est gérée à l'initialisation du composant.

## methods

> To add methods to a component instance we use the methods option. This should be an object containing the desired methods: Vue automatically binds the `this` value for methods so that it always refers to the component instance.

**NDM** : les méthodes du composant peuvent être utilisées comme si le composant était un objet.

> Just like all other properties of the component instance, the methods are accessible from within the component's template. Inside a template they are most commonly used as event listeners:

```html
<button @click="increment">{{ count }}</button>
```

## le rendu du DOM n'est pas instantané

> When you mutate reactive state, the DOM is updated automatically. However, it should be noted that the DOM updates are not applied synchronously. Instead, Vue buffers them until the "next tick" in the update cycle to ensure that each component updates only once no matter how many state changes you have made.

## Synthèse

Début de synthèse sur l'Options API :

- `data` = état (réactif) propre au composant
- `props` = état (réactif) du composant, passé par le parent
- `methods` = méthodes du composant
- `computed` = état du composant, résultant d'un calcul automatique à partir d'autres data/props du composant
- `components` = les sous-composants utilisés par le composant
- `watchers` = les callbacks sur des props qui changent (conceptuellement, ça correspond au fait que le parent "signale" l'enfant)

# Props vs data

Si on considère que le composant vue est un objet, les `data` sont ses attributs ; ils sont réactifs (i.e. ils triggent le re-rendu du DOM lorsque modifiés).

De ce que j'en comprends, les `props` jouent deux rôles :

- permettre à un parent de paramétrer un enfant
- permettre à un parent de partager un de ses états avec un enfant ; la prop chez l'enfant est alors une référence (réactive aussi) vers l'attribut du parent.

Chez le composant enfant, `props` et `data` sont tous deux accessibles via `this` ; du coup les deux ne peuvent pas avoir le même nom.

Pour passer une prop d'un **Parent** vers un **Child** :

- côté **Child** :
    ```html
    <template>
      <p> Look, this is my parent's prop : {{ mysuperprop }}</button>
    </template>
    <script>
    export default {
      props: {
        mysuperprop: String
      }
    };
    </script>
    ```
- côté **Parent** :
    ```html
    <template>
        <Child mysuperprop="Pouet"/>
    </template>
    <script>
    import Child from "./components/Child.vue";
    export default {
      name: "App",
      components: {
        Child
      }
    };
    </script>
    ```

# Communication entre composants

Communiquer (dans les deux sens) entre un parent et un enfant est assez facile :

- le parent et le parent peuvent partager des données via les `props` passées par le parent
- le parent peut trigger une callback chez l'enfant si celui-ci `watch` des props
- l'enfant peut trigger une callback chez le parent en émettant des messages

Communiquer de sibling à sibling est déjà un peu plus compliqué ; e.g. il faut qu'un sibling n°1 émette un event en direction du parent + que le parent redescende l'info au sibling n°2.

Communiquer entre des composants plus éloignés (e.g. petit-enfant à grand-parent) est encore plus compliqué... Plusieurs pistes possibles :

- chaîner les events dans toute la hiérarchie (chaque parent écoute l'event de son enfant et le renvoie à son propre parent ; ou idem avec les props) ; danger = [prop-drilling](https://vuejs.org/guide/components/provide-inject.html#prop-drilling) = on pollue toute la hiérarchie intermédiaire juste pour faire communiquer les deux composants extrêmaux
- utiliser `inject` et `provide` ([lien](https://vuejs.org/guide/components/provide-inject.html)) pour qu'un parent communique avec les descendants
- utiliser un **event-bus** ([vue n'en propose pas](https://v3-migration.vuejs.org/breaking-changes/events-api.html#event-bus) mais redirige sur des libairies, comme [mitt](https://github.com/developit/mitt)) = un broker centralisé auprès duquel des composants peuvent s'abonner à des messages ou en publier (risque = le couplage entre les composants est moins clair, on ne sait plus qui communique avec qui)
- utiliser un store = global state managment tel que [pinia](https://pinia.vuejs.org/) ; à ce stade, j'ai du mal à ne pas considérer ça comme un anti-pattern, où on ne sait plus qui dépend de qui

À noter qu'[une page de la doc vue](https://vuejs.org/guide/components/events.html#emitting-and-listening-to-events) mentionne explicitement cette question de la communication entre composants lointains :

> Unlike native DOM events, component emitted events do not bubble. You can only listen to the events emitted by a direct child component. If there is a need to communicate between sibling or deeply nested components, use an external event bus or a global state management solution.
