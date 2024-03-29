* [Différence entre SELECT et SELECT GROUP BY](#différence-entre-select-et-select-group-by)
* [Exemple avec une base de test](#exemple-avec-une-base-de-test)
* [Cas-réel d'utilisation de GROUP BY](#cas-réel-dutilisation-de-group-by)

# Différence entre `SELECT` et `SELECT GROUP BY`

**TL;DR** : chaque résultat d'un `SELECT GROUP BY` représente un GROUPE de records
- avec un `SELECT` classique, chaque ligne renvoyée correspond à un *record*, et on SELECT des champs d'un record donné
- avec un `SELECT GROUP BY`, chaque ligne renvoyée correspond à un *groupe de record*, et on SELECT des propriétés d'un groupe de record

Les records sont regroupés s'ils partagent une propriété identique, définie par la clause `GROUP BY`.

Conséquence : on ne peut sélectionner QUE des propriétés qui ont du sens sur un groupe de record :
- soit un champ **homogène** sur tous les records du groupe (donc un champ sur lequel on a fait le `GROUP BY`)
- soit une aggrégation calculée sur tous les records du groupe (cf. [doc sur les aggrégations](https://www.postgresql.org/docs/9.5/functions-aggregate.html))

note rigolote : les deux requêtes suivantes produiront la MÊME sortie :

```sql
SELECT customer_name FROM bills GROUP BY customer_name;
SELECT DISTINCT customer_name FROM bills;
```

# Exemple avec une base de test

La base de test :

```sql
CREATE DATABASE mytest;
CREATE TABLE bills (bill_id Integer, customer_name String, bill_amount Integer);
INSERT INTO bills VALUES(1, "HSBC", 300);
INSERT INTO bills VALUES(2, "BNP", 100);
INSERT INTO bills VALUES(3, "BNP", 250);
INSERT INTO bills VALUES(4, "BNP", 250);
INSERT INTO bills VALUES(5, "LCL", 50);
INSERT INTO bills VALUES(6, "LCL", 850);
INSERT INTO bills VALUES(7, "LCL", 100);
INSERT INTO bills VALUES(7, "LCL", 400);
INSERT INTO bills VALUES(8, "LaBanquePostale", 100);
INSERT INTO bills VALUES(9, "SocieteGenerale", 115);

SELECT bill_id, customer_name, bill_amount FROM bills;
--  bill_id |  customer_name  | bill_amount
-- ---------+-----------------+-------------
--        1 | HSBC            |         300
--        2 | BNP             |         100
--        3 | BNP             |         250
--        4 | BNP             |         250
--        5 | LCL             |          50
--        6 | LCL             |         850
--        7 | LCL             |         100
--        7 | LCL             |         400
--        8 | LaBanquePostale |         100
--        9 | SocieteGenerale |         115
```

L'exemple :

```sql
SELECT customer_name FROM bills GROUP BY customer_name;
--   customer_name
-- -----------------
--  BNP
--  LCL
--  HSBC
--  SocieteGenerale
--  LaBanquePostale


SELECT customer_name, bill_amount FROM bills GROUP BY customer_name;
-- ERREUR : il n'est pas possible d'attribuer une valeur aux différents groupes de records pour la propriété "bill_amount"
-- dit autrement : pour un groupe de quelques records, quelle valeur attribuer à "bill_amount" ?


SELECT customer_name, SUM(bill_amount) FROM bills GROUP BY customer_name;
--   customer_name  | sum
-- -----------------+------
--  BNP             |  600
--  LCL             | 1400
--  HSBC            |  300
--  SocieteGenerale |  115
--  LaBanquePostale |  100

-- Ici, pour chaque groupe de records, on peut (i.e. ça a du sens de) calculer SUM(bill_amount)


SELECT customer_name, ARRAY_AGG(bill_amount) FROM bills GROUP BY customer_name;
--   customer_name  | array_agg
-- -----------------+-----------
--  BNP             | {2,3,4}
--  LCL             | {5,6,7,7}
--  HSBC            | {1}
--  SocieteGenerale | {9}
--  LaBanquePostale | {8}

-- Ici, pour chaque groupe de records, on construit un tableau contenant les bill_ids
```

# Cas-réel d'utilisation de GROUP BY

BESOIN = dans une table de sections (int_id, node_from, node_to), lister les couples (node_from, node_to) qui correspondent à plusieurs int_id.

Première façon de faire, avec une jointure :

```sql
-- étape 1 = on construit une table qui liste les couples (node_from, node_to) qui représentent plusieurs sections :
WITH duplicates AS (
    SELECT node_from, node_to
    FROM sections
    GROUP BY node_from, node_to
    HAVING COUNT(*) > 1
)

-- étape 2 = à partir de cette première table, jointure pour lister les int_ids :
SELECT s.id
FROM sections s, duplicates d
WHERE s.node_from = d.node_from AND s.node_to = d.node_to;
```

Deuxième façon de faire, en utilisant `array_agg` et `unnest` :

```sql
-- étape 1 = on construit une table de tableaux de sections_ids d'un même couple (node_from, node_to) :
WITH duplicates AS (
    SELECT array_agg(int_id) AS duplicated_sections
    FROM sections
    GROUP BY node_from, node_to
    HAVING COUNT(*) > 1
)

-- étape 2 = à partir de cette table, on concatène chaque tableau pour obtenir une liste de int_ids :
SELECT unnest(duplicated_sections)
FROM duplicates;
```

Comparaison : `EXPLAIN` semble être plus simple dans le second cas (sans jointure), mais la différence de perf a pas l'air flagrante non plus.
