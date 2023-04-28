* [Installation / mise à jour](#installation--mise-à-jour)
* [Vrac](#vrac)
   * [Utiliser l'API fetch dans node](#utiliser-lapi-fetch-dans-node)

# Installation / mise à jour

cf. [mes notes sur npm](./npm.md).


# Vrac

## Utiliser l'API fetch dans node

[L'API fetch](https://developer.mozilla.org/fr/docs/Web/API/Fetch_API) n'était pas disponible sous node.js jusqu'à une version récente = [la version 18.0](https://nodejs.org/dist/latest-v18.x/docs/api/globals.html#fetch), qui est [sortie en avril 2022](https://nodejs.org/en/blog/announcements/v18-release-announce).


Pour tester si l'API fetch est dispo dans node, rien de plus facile :

```js
const response = await fetch("https://api.openbrewerydb.org/v1/breweries?by_city=san_diego&per_page=3")
console.log(await response.json());
// node 16 = Uncaught ReferenceError: fetch is not defined
// node 18 = OK
```
