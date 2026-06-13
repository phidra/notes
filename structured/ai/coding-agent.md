Ici, des notes communes à tous les coding agents.

---

# Mode PLAN vs. BUILD

`TAB` pour alterner entre mode PLAN et mode BUILD (et plus généralement, entre tous les agents primaires).

**IMPORTANT** = utiliser un modèle intelligent pour planifier, et un modèle moins cher pour builder.

Idéalement, BUILD est réservé aux itérations courtes : dès lors qu'on veut faire du complexe, mieux vaut PLAN d'abord.

## PLAN

En Plan-mode, le coding-agent ne fait aucune modification, et n'utilise que des outils read-only.

Son objectif est de préparer un plan d'action à partir de mon prompt, sans prendre aucune action.

Quand on veut faire des modifications complexes, mieux vaut d'abord utiliser plan-mode AVANT de faire les modifs, pour éviter de devoir itérer.

(le coding agent bascule a priori tout seul en plan-mode si on lui demande d'explorer ou de prévoir comment il va faire les choses)

Une fois le plan défini, l'agent l'affiche, et me demande quoi en faire, en me laissant l'opportunité de l'exécuter, ou de le modifier avant exécution.

## BUILD

Exécution d'une tâche.

# Tips

Utiliser `@` pour faire référence à un fichier dans le prompt sans taper son chemin complet (l'autocomplétion est fuzzy).

Être clair dans le prompt sur le success criteria qui permet à l'agent de vérifier qu'il a bien fait le taf.

si Claude fait continuellement la même erreur, plutôt que de le corriger à chaque fois, inclure la correction dans le fichier `CLAUDE.md`.

# SubAgent

Ressource opencode = https://opencode.ai/docs/agents/

Deux types d'agents :
- **primaires** = ceux avec lesquels l'utilisateur peut interagir (par défaut il y en a deux : Plan et Build)
- **secondaires** = ceux destinés à être subagents = appelés depuis un agent primaire

## C'est quoi

Un subagent a ses propres :

- system prompt
- contexte
- leurs propres permissions (p.ex. le subagent Plan n'aura le droit que d'utiliser des outils read-only)

NOTE : quand on switch entre le mode Plan et le mode Build, en fait on switche entre des sous-agents primaires !

Claude et opencode ont chacun des subagents prédéfinis :

- exemple pour claude = `explore` = recherche rapidement des trucs dans la codebase
- [doc des subagents opencode](https://opencode.ai/docs/en/agents/#built-in) (ma config opencode spécifie d'ailleurs quels modèles chaque subagent doit utiliser)

## Utilisation explicite

Ils sont utiliser automatiquement en fonction du prmopt, mais on peut aussi les utiliser explicitement :

- manuellement : _Review the changes I just made using a subagent focused on code quality._
- avec `@mysubagent` : _@explore Where is the code that handles feature XXX ?_

## Créer ses propres subagents

On peut créer ses propres sous-agents, avec des prompts spécialisés :
- soit directement dans la config opencode
- soit via un fichier markdown :
    - GLOBAL      = `~/.config/opencode/agents/`
    - PER-PROJECT = `.opencode/agents/`

Il faut un front-matter qui décrit l'agent, lui donne un nom, le modèle à utiliser, les outils auxquels il a droit, etc.

Derrière, le contenu du markdown est le prompt system du subagent.

## Exemple opencode
Pour créer un subagent, j'ai fait :

```
opencode agent create
```

NOTE : le nom du fichier est le nom de l'agent.

Puis j'ai raffiné manuellement (et même : via opencode) le fichier créé dans :

```
~/.config/opencode/agents/
```

En plus de raffiner son prompt système, j'ai notamment rajouté manuellement :

```
mode: subagent
model: azure/gpt-5.3-codex
steps: 3
temperature: 0.1
```

La **temperature** règle le niveau d'aléas, depuis des réponses déterministes (températures basses), à des niveaux plus aléatoires (températures hautes).

Le **prompt système** est un prompt "racine", toujours présent, non-overridable par les prompts de l'utilisateur. Il définit le rôle, les règles à respecter, donne du contexte, etc.

Chaque subagent a son propre system prompt, qui lui donne son rôle spécialisé.

