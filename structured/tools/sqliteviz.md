**C'est quoi** : un viewer de BDD sqlite (et fichiers CSV) sympa.

Le github : [sqliteviz](https://github.com/lana-k/sqliteviz).

Une version est directement utilisable [depuis le site](https://sqliteviz.com/app/), mais pensé offline first.

# Utiliser une version locale

```Dockerfile
FROM node:18.20.8-bookworm AS build
ARG SQLITEVIZ_REF=master
RUN apt-get update && apt-get install -y --no-install-recommends git ca-certificates && rm -rf /var/lib/apt/lists/*
RUN git clone --depth 1 --branch "${SQLITEVIZ_REF}" https://github.com/lana-k/sqliteviz.git /app
WORKDIR /app
RUN npm ci && npm run build
FROM nginx:alpine
RUN cat <<'EOF' > /etc/nginx/conf.d/default.conf
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
EOF
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
```

Builder :

```sh
docker build -t sqliteviz-local .
```

Runner :

```sh
docker run --rm -p 8080:80 sqliteviz-local
```

# ASTUCE plotter des valeurs loggées avec timestamp

Par exemple, si je greppe des logs dans ce genre :

```
2022-06-13 00:00:31,870	INFO	[pouet] 12345 this is my log, that took 138 ms
```

Je peux transformer une liste de tels logs en CSV (à deux colonnes : datetime + mesure) avec un coup de vim :

```
2022-06-13 00:00:31,138
```

Avec sqliteviz, je peux importer mon CSV, stocker le datetime dans une colonne, et la valeur dans l'autre :

```sql
SELECT datetime(col1) AS dt, col2 FROM "times"
```

Derrière, il faut cliquer sur "play" pour exécuter la requête et charger les données, et je peux alors tracer des histogrammes ou des courbes.

Bonus = si je veux plotter des _différences de timestamp_ entre deux lignes de log :
- je fais un coup de vim pour avoir un CSV où chaque ligne à 3 colonnes :
    - datetime avant (sans les millisecondes, inutilisable par la fonction `datetime` de sqlite)
    - datetime après
    - (la mesure)
- exemple de ligne :
    ```
    2022-06-13 00:43:18,2022-06-13 00:44:07,2945
    ```
- derrière, pour plotter sous sqlite le diff de deux colonnes datetime :
    ```sql
    SELECT datetime(col1) AS premier, datetime(col2) AS deuxieme, ROUND((JULIANDAY(col2) - JULIANDAY(col1)) * 86400) AS diff FROM "times"
    ```
