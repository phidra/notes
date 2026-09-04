#Installation

J'ai suivi la doc : https://opencode.ai/docs/en/

```sh
curl -fsSL https://opencode.ai/install | bash
```

# Config

## Fichier de config

```sh
~/.config/opencode/opencode.json   # json classique
~/.config/opencode/opencode.jsonc  # json avec commentaires
```

## Débugger la config

Pour afficher la configuration effectivement utilisée (qui résulte du merge du fichier de conf, des plugins, des envvars, etc.) :

```sh
opencode debug config
```

## Afficher les lignes copiées dans le prompt

Ajouter ceci au fichier de config :

```json
{
  "experimental": {
    "disable_paste_summary": true
  }
}
```

# Choix des modèles

## Connaître les modèles disponibles

La clé à renseigner dans le fichier de conf est donnée par :

```sh
# lister tous les modèles :
opencode models

# lister les modèles d'un provider en particulier :
opencode models opencode-go
```


## Configurer les modèles des agents primaires

Dans le fichier de config :

```json
"agent": {
  "plan": {
    "model": "opencode-go/glm-5.2"
  },
  "build": {
    "model": "opencode-go/deepseek-v4-pro"
  }
}

```


# Plugins

**Créer un plugin** = il suffit de dropper un fichier javascript `myplugin.js` (ou `myplugin.ts` en typescript) dans un de ces répertoires :

- GLOBAL = `~/.config/opencode/plugins/`
- LOCAL  = `/path/to/myproject.opencode/plugins/`

Derrière, inutile d'activer le plugin ou de le déclarer, ça fonctionne automatiquement.


## Jouer un son quand mon intervention est requise

Comme player, j'utilise `pw-play`, dispo via pipewire :

```
sudo apt install pipewire
sudo apt install wireplumber
systemctl --user restart wireplumber
```

Derrière, je créé un plugin `~/.config/opencode/plugins/sound.js` :


```js
export const SoundPlugin = async ({ $ }) => {
  return {
    event: async ({ event }) => {
      if (event.type === "session.idle") {
        await $`pw-play /usr/share/sounds/freedesktop/stereo/complete.oga`.quiet()
      }
      if (event.type === "permission.asked") {
        await $`pw-play /usr/share/sounds/freedesktop/stereo/window-attention.oga`.quiet()
      }
    },
  }
}
```


# Shortcuts utiles

Tout est configurable : https://opencode.ai/docs/en/keybinds/

Beaucoup de commandes repose sur un **leader** (par défaut, `CTRL-x`).

Aide = `LEADER-p(` (pour voir les commandes accessibles)

Shortcuts utiles :

- `LEADER-down` pour voir les agents
    - `Up` (sans LEADER) pour revenir à la vue normale
- `LEADER-b` pour toggle la sidebar
    - utile pour copier-coller des lignes dans le terminal (mais dsi c'est pour le dernier message, `LEADER-y` est mieux !)
- `LEADER-y` copier dans le presse-papiers le dernier message du LLM
- `LEADER-x` exporter dans un fichier l'ensemble de la session
- `LEADER-e` ouvrir un éditeur sur un fichier temporaire
    - possiblement utile pour crafter un prompt (même si je préfère faire ça dans une autre fenêtre
- `LEADER-l` ouvrir la vue des sessions (depuis laquelle on peut en supprimer avec `Ctrl-d` et en renommer avec `Ctrl+r`)

Sinon :

- `CTRL+j` = sauter une ligne
- `CTRL+c` :
    - vider le prompt actuel s'il est non-vide
    - sortir d'opencode sinon
- `CTRL+a` = déplacer le curseur au DÉBUT de la ligne
- `CTRL+e` = déplacer le curseur à la FIN de la ligne
- `CTRL+k` = effacer depuis le curseur jusqu'à la FIN de la ligne
- `CTRL+u` = équivalent de `CTRL+k` mais vers le DÉBUT de la ligne : effacer la ligne depuis le curseur jusqu'au DÉBUT.


# Commandes utiles

`/undo` et `/redo` = annuler/restaurer le dernier prompt (ou d'autres)

`/fork` pour forker la session à partir d'une de mes questions passées (tout se passe comme si je venais d'entrer la question dans le prompt, sans l'avoir envoyée).

`/sessions` pour ouvrir la vue des sessions

# Tips

## Ouvrir une session "temporaire"

Le plus simple est de forker puis supprimer le fork :

- forker toute la session : `/fork` → choisir "Full session"
- entrer mes questions "temporaires", et échanger librement
- quand j'ai fini, ouvrir l'éditeur de sessions (`/sessions` ou `LEADER-l`), sélectionner la session forkée (suffixée `(fork #1)`), et la supprimer avec `Ctrl-d` deux fois, puis revenir à la session d'avant

# MCP

## Configuration

En dehors du coding-agent, commande pour ajouter le MCP (soit globally, soit pour le projet) :

```sh
opencode mcp add
# répondre aux questions
# notamment, si c'est un MCP remote, il demandera l'URL
```

On identifie un MCP server par son nom.

## Authentication

Certains MCP servers nécessitent une authentication.

Si besoin, on peut consulter les MCP qui nécessite une authentication :

```
opencode mcp auth list
```

Et authentifier un MCP :

```sh
opencode mcp auth
# ┌  MCP OAuth Authentication
# │
# ◇  Select MCP server to authenticate
# │  ✗ my-atlassian (not authenticated)
# │
# ◇  Authentication successful!
# │
# └  Done
```

Dans mon cas, un browser s'est ouvert (a priori vers un server local créé par le process d'authentication) dans lequel je me suis authentifié, et l'auth a été propagée à opencode.

## Utilisation

Les MCP doivent être enabled/disabled.

On peut consulter les MCP et les enable/disable avec la commande `/mcps`.

Derrière, on peut lui demander explicitement d'utiliser un MCP :

> Using the my-atlassian tools, can you list all the tickets in this board that are affected to me ?

Et voir alors l'agent utiliser le tool MCP dynamiquement, yay

# Consulter les logs

Les logs sont dans :

```
~/.local/share/opencode/log/
```
