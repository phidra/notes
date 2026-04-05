
[La doc officielle](https://docs.astral.sh/uv/) a plein d'infos.

* [Installation](#installation)
* [Usecase = lancer un script](#usecase--lancer-un-script)
* [Usecase = travailler sur un projet](#usecase--travailler-sur-un-projet)
* [Usecase = créer un venv](#usecase--créer-un-venv)
* [Usecase = uvx](#usecase--uvx)

---

# Installation

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh

# downloading uv 0.11.6 x86_64-unknown-linux-gnu
# installing to /home/myself/.local/bin
#   uv
#   uvx
# everything's installed!

uv --version
# uv 0.11.6 (x86_64-unknown-linux-gnu)
```

# Usecase = lancer un script


```sh
# Sans dépendance :
uv run python mon_script.py myarg

# Avec dépendance :
uv run --with requests --with jsonschema python mon_script.py
```

# Usecase = travailler sur un projet

TODO = documenter


# Usecase = créer un venv

TODO = documenter

# Usecase = uvx

TODO = documenter
