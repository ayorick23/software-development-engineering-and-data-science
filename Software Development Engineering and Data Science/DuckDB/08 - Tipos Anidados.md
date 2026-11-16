---
tags: [duckdb, list, struct, map, json, tipos-anidados, cheat-sheet]
---

# 08 — Tipos Anidados

> Continúa de [[07 - Friendly SQL]]. A diferencia del SQL relacional clásico, DuckDB trata las **listas, los structs, los maps y el JSON** como tipos de primera clase. Así puedes trabajar con datos semiestructurados (APIs, logs, eventos) sin aplanarlos antes en Python, y usar listas como herramienta analítica, por ejemplo para agregar a listas y transformarlas con lambdas.

## Panorama de tipos

| Tipo | Literal | Analogía en Python | Uso típico |
|---|---|---|---|
| `LIST` (`INTEGER[]`) | `[1, 2, 3]` | `list` de longitud variable | Tags, items de un pedido, historial |
| `ARRAY` (`FLOAT[3]`) | `[1.0, 2.0, 3.0]::FLOAT[3]` | `list` de longitud **fija** | Embeddings, vectores |
| `STRUCT` | `{'a': 1, 'b': 'x'}` | `dict` con claves fijas / dataclass | Objetos anidados (dirección, usuario) |
| `MAP` | `MAP {'k1': 1, 'k2': 2}` | `dict` con claves variables | Atributos dinámicos, conteos |
| `UNION` | `union_value(num := 2)` | `int \| str` | Valores de tipos heterogéneos |
| `JSON` | `'{"a":1}'::JSON` | `str` con JSON válido | Payloads sin esquema fijo |

```sql
CREATE TABLE pedidos (
    id        INTEGER,
    cliente   STRUCT(nombre VARCHAR, pais VARCHAR),
    items     STRUCT(sku VARCHAR, qty INTEGER, precio DECIMAL(10,2))[],
    tags      VARCHAR[],
    atributos MAP(VARCHAR, VARCHAR),
    embedding FLOAT[384]
);
```

---

## LIST

### Crear

```sql
SELECT [1, 2, 3]                     AS lista,
       list_value('a', 'b')          AS otra,
       ['x', NULL, 'z']              AS con_nulos,
       [[1, 2], [3]]                 AS anidada,
       range(5)                      AS r;              -- [0,1,2,3,4]

-- Agregar filas a una lista (lo inverso de UNNEST)
SELECT cliente_id, list(producto ORDER BY fecha) AS historial   -- alias array_agg
FROM ventas GROUP BY ALL;
```

### Acceder

```sql
SELECT l[1]        AS primero,      -- índices desde 1
       l[-1]       AS ultimo,
       l[2:3]      AS slice,
       len(l)      AS n,
       list_contains(l, 5)  AS tiene_5,     -- alias: list_has
       list_position(l, 5)  AS pos_5,
       list_has_any(l, [1, 9]),
       list_has_all(l, [1, 2])
FROM (SELECT [5, 1, 2, 9] AS l);
```

### Funciones de lista

```sql
SELECT list_sort([3,1,2]),                  -- [1,2,3]
       list_reverse_sort([3,1,2]),
       list_distinct([1,1,2]),
       list_unique([1,1,2]),                -- conteo de distintos: 2
       list_concat([1], [2,3]),             -- también [1] || [2,3]
       flatten([[1,2],[3]]),                -- [1,2,3]
       list_slice([1,2,3,4], 2, 3),
       list_aggregate([1,2,3], 'sum'),      -- 6 (cualquier agregado por nombre)
       list_sum([1,2,3]), list_avg([1,2,3]), list_max([1,2,3]),
       list_zip([1,2], ['a','b']),          -- [{'': 1, '': 'a'}, ...]
       array_to_string(['a','b'], '-'),     -- 'a-b'
       string_split('a,b,c', ',');          -- ['a','b','c']
```

### Lambdas y list comprehensions

```sql
-- Transformar cada elemento
SELECT list_transform([1, 2, 3], x -> x * 10);            -- [10,20,30]
-- Sintaxis estilo Python (1.3+), recomendada a futuro:
SELECT list_transform([1, 2, 3], lambda x: x * 10);

-- Filtrar
SELECT list_filter([1, 5, 8, 12], x -> x > 4);            -- [5,8,12]

-- Reducir
SELECT list_reduce([1, 2, 3, 4], (acc, x) -> acc + x);    -- 10

-- Con índice (segundo parámetro)
SELECT list_transform(['a','b'], (x, i) -> i || ':' || x); -- ['1:a','2:b']

-- List comprehension al estilo Python
SELECT [x * 2 FOR x IN [1, 2, 3, 4] IF x % 2 = 0];        -- [4, 8]
```

Caso práctico: limpiar una lista de tags dentro de la consulta.

```sql
SELECT id,
       list_distinct(
         list_filter(
           list_transform(string_split(tags_raw, ','), t -> lower(trim(t))),
           t -> t <> ''
         )
       ) AS tags
FROM productos;
```

---

## UNNEST: de listas a filas

```sql
-- Una fila por elemento
SELECT id, unnest(tags) AS tag FROM productos;

-- Varias listas en paralelo (se alinean por posición y rellenan con NULL)
SELECT unnest([1,2,3]) AS n, unnest(['a','b']) AS l;

-- Con posición: generate_subscripts
SELECT id, unnest(items) AS item, generate_subscripts(items, 1) AS pos FROM pedidos;

-- Desanidar listas de structs en columnas, de forma recursiva
SELECT id, unnest(items, recursive := true) FROM pedidos;
-- columnas: id, sku, qty, precio

-- Desanidar N niveles
SELECT unnest([[1,2],[3]], max_depth := 2);    -- 1,2,3
```

Patrón clásico **explode → agregar → reagrupar**:

```sql
-- Conteo de tags más frecuentes
SELECT tag, COUNT(*) AS n
FROM (SELECT unnest(tags) AS tag FROM productos)
GROUP BY ALL ORDER BY n DESC LIMIT 10;
```

---

## STRUCT

```sql
SELECT {'nombre': 'Ana', 'edad': 30}               AS persona,
       struct_pack(nombre := 'Ana', edad := 30)    AS persona2,
       row('Ana', 30)                              AS sin_nombres;

-- Acceso
SELECT persona.nombre, persona['edad'], struct_extract(persona, 'nombre')
FROM (SELECT {'nombre': 'Ana', 'edad': 30} AS persona);

-- Expandir todos los campos a columnas
SELECT persona.* FROM t;
SELECT unnest(persona) FROM t;          -- equivalente para structs

-- Una fila entera como struct (útil para to_json o para comparar filas)
SELECT t AS fila FROM clientes t;

-- Modificar o añadir campos
SELECT struct_insert(persona, pais := 'PE') FROM t;
```

## MAP

```sql
SELECT MAP {'color': 'rojo', 'talla': 'M'}           AS m1,
       map(['a', 'b'], [1, 2])                       AS m2,
       map_from_entries([('k', 1), ('j', 2)])        AS m3;

SELECT m['color'],               -- valor (NULL si no existe)
       map_extract(m, 'color'),  -- devuelve lista [valor]
       map_keys(m), map_values(m), map_entries(m),
       cardinality(m),
       map_contains(m, 'talla')
FROM (SELECT MAP {'color': 'rojo', 'talla': 'M'} AS m);
```

**STRUCT vs. MAP:** usa un STRUCT cuando las claves son **fijas y conocidas**, porque cada campo es una columna tipada y eficiente. Usa un MAP cuando las claves **varían de fila a fila**, como atributos de producto o parámetros de un evento.

## ARRAY: longitud fija y vectores

```sql
CREATE TABLE docs (id INT, texto VARCHAR, emb FLOAT[3]);
INSERT INTO docs VALUES (1, 'gato', [0.1, 0.9, 0.0]), (2, 'perro', [0.2, 0.8, 0.1]);

SELECT id, texto,
       array_cosine_similarity(emb, [0.1, 0.85, 0.05]::FLOAT[3]) AS sim
FROM docs
ORDER BY sim DESC
LIMIT 5;
-- también: array_distance (euclídea), array_inner_product, array_cosine_distance
```

Con la extensión `vss` se puede crear un índice HNSW para búsqueda vectorial aproximada, y DuckDB funciona como vector store embebido para RAG. Ver [[12 - Extensiones#Búsqueda vectorial con vss|12 - Extensiones (vss)]] y [[10-IA-Generativa-LLMs-y-Agentes]].

---

## JSON como tipo

### Leer y extraer

```sql
SELECT j->'$.user'              AS user_json,       -- JSON
       j->>'$.user.name'        AS nombre,          -- VARCHAR
       j->'$.items[0]'          AS primer_item,
       j->>'$.items[*].sku'     AS skus,            -- lista de todos los sku
       json_extract(j, ['$.a', '$.b'])  AS varios,
       json_type(j->'$.items'),                    -- 'ARRAY'
       json_valid(j),
       json_keys(j)
FROM (SELECT '{"user":{"name":"Ana"},"items":[{"sku":"A"},{"sku":"B"}]}'::JSON AS j);
```

### Inferir esquema y convertir a tipos nativos

```sql
-- ¿Qué estructura tiene este JSON?
SELECT json_structure(payload) FROM eventos LIMIT 1;

-- Convertir a STRUCT tipado (más rápido de consultar que extraer con ->>)
SELECT json_transform(payload, '{"user": {"name": "VARCHAR"}, "monto": "DOUBLE"}') AS p
FROM eventos;

-- Alternativa con cast directo
SELECT payload::STRUCT(user STRUCT(name VARCHAR), monto DOUBLE) FROM eventos;
```

### Construir JSON

```sql
SELECT to_json({'id': 1, 'tags': ['a','b']});
SELECT json_object('id', id, 'nombre', nombre) FROM clientes;
SELECT json_group_array(nombre) FROM clientes;              -- '["Ana","Luis"]'
SELECT json_group_object(id, nombre) FROM clientes;         -- '{"1":"Ana",...}'

-- Exportar cada fila como un objeto JSON
SELECT to_json(c) FROM clientes c;
```

### ¿Mantener JSON o convertir?

| Situación | Recomendación |
|---|---|
| Esquema estable | Convierte a STRUCT/columnas en la capa *silver* |
| Esquema variable o desconocido | Guarda como `JSON` en la capa *bronze* y extrae lo que necesites |
| Consultas frecuentes sobre un campo | Extráelo a una columna propia (más rápido y comprimible) |

---

## UNION (tipo suma)

```sql
CREATE TABLE mediciones (valor UNION(num DOUBLE, txt VARCHAR));
INSERT INTO mediciones VALUES (1.5), ('sin dato');
SELECT union_tag(valor), valor.num, valor.txt FROM mediciones;
```

Es poco común; aparece sobre todo al leer JSON muy heterogéneo.

## Ejemplo integrador: eventos de una API

```sql
-- eventos.ndjson: {"ts":"...","user":{"id":7,"plan":"pro"},"cart":[{"sku":"A","qty":2,"price":10.5}, ...]}
WITH base AS (
    SELECT ts::TIMESTAMP AS ts, "user".id AS user_id, "user".plan AS plan, cart
    FROM 'eventos.ndjson'
),
lineas AS (
    SELECT ts, user_id, plan, unnest(cart, recursive := true)
    FROM base
)
SELECT plan,
       date_trunc('day', ts)        AS dia,
       SUM(qty * price)             AS ingreso,
       list(DISTINCT sku)           AS skus_vendidos,
       arg_max(sku, qty * price)    AS sku_top
FROM lineas
GROUP BY ALL
ORDER BY dia, plan;
```

`"user"` va entre comillas dobles porque `user` es palabra reservada.

## Ver también

- [[04 - Leer y Escribir CSV y JSON]]: cómo cargar el JSON
- [[07 - Friendly SQL]]
- [[09 - Analítica Avanzada]]
- [[14 - JSON and YAML]] (en `Python/`)
- [[28-SQL-Profesional-Nivel-3-Modelado]] (en `Machine Learning/`): modelado relacional frente a datos anidados
