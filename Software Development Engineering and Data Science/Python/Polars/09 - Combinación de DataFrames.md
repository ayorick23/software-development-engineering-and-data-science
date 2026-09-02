---
tags: [polars, python, data-science, join, cheat-sheet]
---

# 09 — Combinación de DataFrames

> Continúa de [[08 - Agrupación y Agregación]].

## `join()` — combinar por columnas clave

```python
clientes.join(pedidos, on="cliente_id", how="inner")
clientes.join(pedidos, on="cliente_id", how="left")
clientes.join(pedidos, on="cliente_id", how="outer")
clientes.join(pedidos, how="cross")                     # producto cartesiano
```

| `how` | Filas resultantes |
|---|---|
| `"inner"` | Intersección de claves |
| `"left"` | Todas las de la tabla izquierda |
| `"outer"` | Unión de ambas, con `null` donde falte |
| `"semi"` | Filas de la izquierda que SÍ tienen match — sin traer columnas de la derecha |
| `"anti"` | Filas de la izquierda que NO tienen match — el opuesto de `"semi"` |
| `"cross"` | Producto cartesiano |

`"semi"` y `"anti"` no tienen un equivalente de una sola palabra en `pd.merge()` de Pandas — replican el patrón SQL `WHERE EXISTS`/`WHERE NOT EXISTS` (ver [[SQL/Subqueries|SQL]]) directamente como un tipo de join, evitando tener que hacer un `left join` seguido de un filtro por nulos para lograr el mismo resultado.

```python
# Semi-join: clientes que SÍ tienen al menos un pedido (sin traer columnas de pedidos)
clientes.join(pedidos, on="cliente_id", how="semi")

# Anti-join: clientes que NO tienen ningún pedido
clientes.join(pedidos, on="cliente_id", how="anti")
```

## Claves con nombres distintos

```python
clientes.join(pedidos, left_on="id", right_on="cliente_id")
clientes.join(pedidos, on=["cliente_id", "region"])     # múltiples columnas clave
```

## `join_asof()` — combinar por la coincidencia más cercana (series de tiempo)

```python
precios.join_asof(operaciones, on="timestamp", by="ticker", strategy="backward")
```

`join_asof()` combina filas basándose en el valor **más cercano** de una columna ordenada (típicamente tiempo), no en una coincidencia exacta — el caso de uso clásico es unir el precio de una acción con la operación más reciente conocida en ese momento, sin requerir timestamps idénticos. No existe un equivalente de una sola función tan directo en `pd.merge()` de Pandas (Pandas tiene `pd.merge_asof()`, con sintaxis muy similar).

## `concat()` — apilar DataFrames

```python
pl.concat([df_enero, df_febrero, df_marzo])                     # apilar verticalmente (default)
pl.concat([df_a, df_b], how="horizontal")                          # pegar columnas lado a lado
pl.concat([df_enero, df_febrero], how="diagonal")                     # apila verticalmente incluso con columnas distintas, rellenando con null
```

`how="diagonal"` no tiene equivalente directo en `pd.concat()` de Pandas para el caso de columnas que no coinciden exactamente entre los DataFrames — resuelve automáticamente la unión de esquemas, rellenando con `null` donde una columna no existía en uno de los DataFrames originales.

## Validar la cardinalidad de un join

```python
clientes.join(pedidos, on="cliente_id", how="inner", validate="1:m")
```

Igual que `validate=` en `pd.merge()` (ver [[Python/Pandas/09 - Combinación de DataFrames#validate e indicator — detectar problemas de cardinalidad|Python/Pandas]]) — lanza un error explícito si la relación real de cardinalidad no coincide con la esperada, en vez de inflar filas silenciosamente.

## Ver también

- [[08 - Agrupación y Agregación]]
- [[10 - Reshaping]]
- [[11 - Series de Tiempo]]
- [[Python/Pandas/09 - Combinación de DataFrames|Python/Pandas]]
