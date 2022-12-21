**Contexte** : mai 2022, j'essaye d'y voir plus clair sur la place de node/npm dans le dev frontend ; j'en profite pour m'intéresser aux build-tools. J'ai fini par faire [un blogpost qui résume l'essentiel sur le sujet](https://phidra.github.io/blog/2022-05-26-node-frontend/), et ces notes sur les notes vrac que j'avais prises dans la phase d'analyse.

En résumé :

- node.js et npm sont des outils de dev : du point de vue du navigateur qui au final exécute le HTML+CSS+JS de l'app, ces outils n'existent même pas.
- dans le dev frontend moderne, on n'écrit plus directement le HTML/CSS/JS exécuté par le navigateur au final (surtout le js), on écrit du code, et celui-ci est **buildé** pour générer le HTML/CSS/JS à exécuter
- par rapport à écrire son HTML/CSS/JS directement, les build-tools apportent les features suivantes :
    - TRANSPILING = e.g. on dev en typescript (langage typé) et le build-tool génère le js à exécuter par le navigateur
    - NOM INCONNU ? = dans le fichier HTML, au lieu de gérer les tags `<script>` à la main, on liste ses librairies dans un `package.json`, et c'est le buil-tool qui produit les tags `<script>` des librairies, et de leurs dépendances récursives.
    - MINIFYING = diminuer le poids du js, suppression de commentaire ou whitespaces, raccourcissement de noms de variables, etc.
    - BUNDLING = agréger les divers scripts js (y compris des librairies utilisées) en un unique js, prêt à l'emploi par le navigateur.
    - TREESHAKING = dans le js bundlé, supprimer le dead-code js jamais utilisé dans l'application.


* [place de node.js dans le dev frontend](#place-de-nodejs-dans-le-dev-frontend)
* [build tools](#build-tools)

# place de node.js dans le dev frontend

https://www.section.io/engineering-education/nodejs-frontend-backend/

> Node.js is a runtime environment used for executing server-side code with higher efficiency

NDM : node.js est bien backend, mais c'est en quelque sorte la VM js qui servira à exécuter les tools, donc l'équivalent de l'interpréteur python.

node.js n'est JAMAIS exécuté côté frontend (c'est plutôt la VM native du navigateur qui fait ça), mais node.js peut servir au dev à travailler sur son code frontend :

- soit directement = node.js exécute en backend le code visant à tourner côté frontend (p.ex. pour faire tourner des tests unitaires dessus)
- soit indirectement = node.js sert à exécuter des outils (e.g. linter : ESLint utilise node.js) qui s'applique au code qui tournera côté frontend

Les outils utiles au dev frontend qui peuvent nécessiter node.js :

- code-preprocessor = un preprocessor CSS permet d'écrire du CSS dans un langage plus convenient (e.g. langage SASS, qui sera préprocessé en CSS). Il existe aussi des préprocesseurs HTML (e.g. langage HAML).
- code-linter = vérifier statiquement le code
- Module bundlers = Module bundlers are programs that take in various code files and bundles them into a single file. Such programs are usually included with Web frameworks like React. (e.g. vue-cli permet de bundle)
- Styling = écrire son code HTML+CSS dans un langage spécifique (pour le transpiler en HTML+CSS)
- Package = utiliser des third-party modules ; Building the frontend is as simple as collecting a bunch of required components and stitching them together to create a beautiful UI.

Par ailleurs, côté backend, node.js reste utilisé pour tout ce dont on peut avoir besoin côté back :

- Network and API calls
- Database Integration
- Operating System-Level Control over Application
- Real-Time Applications

CLARIFICATION : pour moi, ces outils s'exécutent tous côté backend (grâce à nodejs), mais travaillent sur le code frontend, pour le raffiner.
- donc quand on dit que node.js "est utilisé côté front-end", c'est un abus de langage, il est juste utilisé POUR BOSSER SUR DU CODE FRONT-END
- par exemple, la conclusion de l'article me paraît fausse/confusante : "Node.js developed as a server-side runtime environment can be used extensively in the frontend as well."
- disons que "node.js est également utile au devs frontend", mais node.js n'est PAS utilisé côté frontend

NDM : beaucoup d'outils pour au final éviter d'écrire le HTML+CSS, et faire de la génération de code à la place `(o_O')`... En gros, HTML+CSS is the new assembly, personne ne veut l'écrire soi-même.

# build tools

Du coup, "build-tools" = des outils pour créer son application frontend, i.e. générer le triplet HTML+CSS+JS que le navigateur exécutera.

---

https://developerexperience.io/practices/javascript-front-end-build-tools

Here are some of the most popular build tools:

- npm: Package manager.
- Webpack: Module bundler.
- Gulp: Runs and automatizes tasks.
- ESLint: Code analyzer.
- Grunt: Runs and automatizes tasks.

Most build tools are built on top of Node and npm (Node Package Management). For this reason, most of the front-end developers try to use these two tools as much as possible and then install additional tools. Npm comes preinstalled with Node.js.

Common Pitfalls of Javascript Front-End Build Tools
- Too much information
- Complexity
- Documentation sucks

^ Je ne peux pas être plus d'accord `('^_^)`

---

Je peux maintenant revenir comprendre la FAQ vue.js sur les build-tools :

https://developer.mozilla.org/en-US/docs/Glossary/Tree_shaking

> Tree shaking is a term commonly used within a JavaScript context to describe the removal of dead code.
>
> It relies on the import and export statements in ES2015 to detect if code modules are exported and imported for use between JavaScript files.
>
> In modern JavaScript applications, we use module bundlers (e.g., webpack or Rollup) to automatically remove dead code when bundling multiple JavaScript files into single files.

TL;DR : si l'application frontend est construite avec un buildtool (e.g. un bundler), le tool peut détecter le code inutilsé et le dégager = c'est le tree-shaking.

https://vuejs.org/about/faq.html#is-vue-lightweight

> When using Vue without a build tool, we not only lose tree-shaking, but also have to ship the template compiler to the browser.

^ Je comprends maintenant cette phrase = si je construis le HTML+CSS+JS qui sera interprété par le browser manuellement (par opposition à "avec un bundler"), alors je n'ai plus le tree-shaking (et par ailleurs, je déduis de la phrase que le build-tool compile le template côté serveur -> si je n'utilise pas le build-tool, il faut fournir au navigateur le code permettant de le faire lui-même)

---

https://vue-community.org/guide/ecosystem/build-tools.html

> To be used [in browser], the application needs to be transformed by a various tools like Babel, PostCSS, Sass, Typescript to name a few.

^ ça résume bien le principe des build-tools = l'app n'est plus nativement écrite en un HTML+JS+CSS directement interprétable par le navigateur, elle est BUILDÉE par tout plein d'outils.

Le reste de la page donne quelques build-tools...

Vue CLI est le tool officiel vue (pour pas se prendre la tête avec les build-tools et se concentrer sur l'app en elle-même) = https://cli.vuejs.org/

---

https://www.freecodecamp.org/news/making-sense-of-front-end-build-tools-3a1b3a87043b/

Un bon article pour y voir plus clair. Il est splitté en 8 "concepts", mais il ne faut pas s'arrêter aux titres, plein d'infos intéressantes sont disséminées !

- Concept #1 — The core dichotomy of build tools is “installing vs. doing”
    - e.g. npm (package manager) INSTALLE, webpack (qui bundle le code) FAIT des trucs
- Concept #2 — The grandparent of all build tools is Node and npm
    - Node and npm install and run all these build tools, so there is always a trace of them in your project.
    - C'est un point très confusant, et la raison pour laquelle on dit (à tort selon moi) que node est utilisé côté frontend.
    - Ma façon de voir les choses = node.js est simplement l'interpréteur du code (js) de ces outils, c'est un peu l'équivalent de l'interpréteur python.
    - KEYPOINT = If you need a place to start learning, start with Node+npm, and stay there for a while → inutile d'aller chercher des build-tools évolués, commencer par node+npm ce sera déjà pas mal
- Concept #3 — A build is just a production ready version of your app
    - KEYPOINT = tous ces outils servent à construire un build = le code qui sera réellement interprété par le navigateur.
    - il faut voir les choses comme un process de compilation, pour faire le parallèle avec C++ :
        - C++ = le code sur lequel je dev (en C++) n'est pas le code interprété au runtime (assembleur) : le code au runtime est produit par un build-tool
        - js = le code sur lequel je dev (sass, typescript, ...) n'est pas le code interprété par le navigateur (html+css+js) : le code interpété par le navigateur est construit par le build-tool
    - notamment, phase de bundling importante = rassembler tout le code en un seul fichier, pour éviter que le navigateur ait trop de requêtes à faire
    - KEYPOINT = inutile pour de petites apps : "If you’re just learning development, or only making sites with very low traffic, generating a build might not be worth your time."
- Concept #4 — The lines between “install” and “do” can be blurry
    - p.ex. npm peut faire des choses également comme lancer des scripts
- Concept #5 — There is no one right combination of tools
- Concept #6 — Build tools have a steep learning curve, so only learn what’s necessary
- Concept #7— All build tools share the same goal: to make you happy by automating a lot of menial task
    - So how much effort should you put into configuring and setting up your build tool? Simple: stop when you’re happy with what it’s doing for you.
- Concept #8 — It’s not just you. The documentation often is terrible.

---

https://medium.com/@trevorpoppen/modern-front-end-the-tools-and-build-process-explained-36641b5c1a53

  Why is Front End “Built” Now? The different steps:
- TRANSPILING = le dev développe dans un langage donné (e.g. js récent, voire typescript ou coffescript), et après transpiling, le code transpilé est dans un vanilla js compatible avec le navigateur (js plus ancien)
- BUNDLING = au lieu de gérer les tags `<script>` du fichier HTML à la main, on maintient des listes de dépendances dans des fichiers de requirements, et le bundler s'assure de produire les tags `<script>` nécessaire, ou bien d'inclure directement le bon js.
    - Original Problem:
        - Importing via Script tags isn’t scalable or manageable
        - New packages become available daily
        - Per points 1 and 2: As more and more JavaScript packages came to be, the script tag region of index.html became unmanageable. As packages are updated and new versions are released, those script tags need updated. And if some of your imported packages relied on other packages, and those packages had cross dependencies, the script tag ordering and versioning gets messy.
    - So now instead of using Script tags and having libraries be globally available, we are importing specific libraries into our modularized files. How do we turn this into a deliverable, browser compatible JavaScript file? Well, by Bundling of course.
    - Bundling is the process of taking all the “import” or “require” statements, finding the matching JavaScript snippets/packages/libraries, adding them to the appropriate scope, and packaging it all into one big JavaScript file. That file will be one of the few, if not only, script tags in your index.html file.
- MINIFYING = diminuer le poids du js, suppression de commentaire ou whitespaces, etc.
- PACKAGING = générer quelque part le build final (un unique fichier js, p.ex.), prêt à l'emploi par le navigateur.
- NdM = j'ajoute TREESHAKING = virer du dead-code jamais utilisé.

La page donne un exemple utilisant :

- Package Manager: npm
- Transpiler: babel
- Minifier: UglifyJS
- Bundler: webpack

---

https://apprendre-nodejs.fr/v1/chapter-09/

- Quel rapport entre Node et les navigateurs web ?
- La réponse courte est : nous n’exécutons pas Node dans un navigateur.
- Et voici la réponse longue : Node est utilisé pour assembler du code, le transformer et le rendre fonctionnel dans une paire de balises `<script></script>`. Ce code peut aussi aussi bien être fourni par des bibliothèques tierces installées via npm (jQuery, React ou d3 par exemple) que par de l’outillage (optimiseurs, suite de tests, orchestration de tâches) ou encore par le code réutilisable de notre propre application web.

---

https://vuejs.org/guide/quick-start.html#with-build-tools

du coup, comme l'app qui tourne au final n'est PAS le code sur lequel je développe, pour tester mes devs, il faut que je fasse tourner l'env de dev avec "npm run dev" (il s'occupe d'utiliser les dépendances, p.ex.)

on dirait qu'il y a DEUX build-tools officiels : create-vue (utilisant vite) et vue cli (utilisant webpack)


