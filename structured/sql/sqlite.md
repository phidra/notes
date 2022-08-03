NOTE : cf. également [mes POCS](https://github.com/phidra/pocs/tree/235cb236a03feb86e0f3a781144f7fc4dcad539a/sql) (où j'illustre notamment comment analyser un fichier CSV avec sqlite, très pratique)

Pour enregistrer le résultat d'une requête dans un fichier, puis remettre l'affichage sur la console :

```sqlite
.output /tmp/result.txt
SELECT * FROM mytable;
.output stdout
```

Conversion des colonnes CSV : ceci ne marche pas sur sqlite, car on ne peut pas modifier des columns :

```sql
UPDATE stop_times SET stop_sequence = CAST(stop_sequence AS INTEGER);
```

Mais ceci fonctionne :

```sql
ALTER TABLE stop_times ADD COLUMN stop_seq_int INT NOT NULL DEFAULT -1;
UPDATE stop_times SET stop_seq_int = CAST(stop_sequence AS INTEGER);
```

En théorie, je peux également renommer la nouvelle colonne pour prendre le nom de l'ancienne (en pratique, ma version de sqlite est trop ancienne).


Équivalent de `array_agg` = `group_concat` :

```sql
SELECT trip_id, GROUP_CONCAT(stop_seq_int) FROM (SELECT * FROM stop_times ORDER BY stop_seq_int DESC) GROUP BY trip_id LIMIT 2;
```
