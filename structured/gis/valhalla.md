# Valhalla

**C'est quoi ?** Un moteur open-source pour faire du routing sur de la donnée OSM.

Il y a [plein d'APIs disponibles](https://valhalla.github.io/valhalla/api/) :

- routing (y compris en mixant des données GTFS avec un graphe point-to-point OSM)
- isochrone
- elevation
- ...

**C'est fait par qui ?** [La doc](https://valhalla.github.io/valhalla/) ne semble pas mentionner l'origine du projet, mais il y a beaucoup de monde de chez mapzen dans [les AUTHORS](https://github.com/valhalla/valhalla/blob/master/AUTHORS), et l'URL de mapzen est donnée dans le header du [repo github](https://github.com/valhalla). Bémol pour cette hypothèse = [cet article de blog](http://www.juliendelabaca.com/innovation-ouverte-et-mobilite-il-est-temps-dentrer-dans-une-nouvelle-ere/) mentionne plutôt mapbox comme auteurs.

Le tool passe pour être compliqué à setup, mais parmi [les third-party tools référencée par la doc](https://valhalla.github.io/valhalla/#related-projects), il y a une [image docker magique](https://github.com/gis-ops/docker-valhalla) pour tout faire d'un coup de docker-compose.

Rien à voir, mais il y a aussi un tool de decoding des [polylines google](https://valhalla.github.io/valhalla/decoding/) en démo : https://valhalla.github.io/demos/polyline/

## routing

TODO

## elevation

valhalla expose [une API](https://valhalla.github.io/valhalla/api/elevation/api-reference) pour récupérer l'altitude d'un point ou d'une route :

```sh
# un unique point = l'altitude de Bourg-Saint-Maurice :
curl https://VALHALLAHOST/height --data '{"shape":[{"lat":45.616973,"lon":6.767273}]}' | jq "."

# une "route" constituée de deux points, en POST :
curl https://VALHALLAHOST/height --data '{"shape":[{"lat":45.768571,"lon":6.531887},{"lat":45.760712,"lon":6.533432}]}' | jq "."

# astuce pour avoir l'équivalent en GET :
JSON=$(python -c "import urllib.parse, sys; print(urllib.parse.quote(sys.argv[1]))" '{"shape":[{"lat":45.768571,"lon":6.531887},{"lat":45.760712,"lon":6.533432}]}')
curl https://VALHALLAHOST/height?json=JSON | jq "."

# ce qui s'expand en :
curl https://VALHALLAHOST/height?json=%7B%22shape%22%3A%5B%7B%22lat%22%3A45.768571%2C%22lon%22%3A6.531887%7D%2C%7B%22lat%22%3A45.760712%2C%22lon%22%3A6.533432%7D%5D%7D
```


Exemple de réponse :

```
{
  "shape": [
    {
      "lat": 45.768571,
      "lon": 6.531887
    },
    {
      "lat": 45.760712,
      "lon": 6.533432
    }
  ],
  "height": [
    1628,
    1660
  ]
}
```

En passant l'option `range=true` à la query, on peut avoir la distance de chaque point depuis l'origine en plus de l'altitude (utile si on passe la polyline d'un trajet, pour calculer la pente de chaque segment).

On peut également lui passer des polylines encodées en remplaçant le paramètre `shape` par le paramètre `encoded_polyline`.




