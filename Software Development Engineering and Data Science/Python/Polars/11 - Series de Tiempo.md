---
tags: [polars, python, data-science, time-series, cheat-sheet]
---

# 11 — Series de Tiempo

> Continúa de [[10 - Reshaping]].

## Tipos de fecha explícitos: `Date` vs `Datetime`

```python
df.with_columns(pl.col("fecha_str").str.to_date("%Y-%m-%d"))            # solo fecha, sin hora -> dtype Date
df.with_columns(pl.col("timestamp_str").str.to_datetime("%Y-%m-%d %H:%M:%S"))   # fecha + hora -> dtype Datetime
```

A diferencia de Pandas (donde todo es `datetime64`, con o sin componente de hora), Polars distingue explícitamente `pl.Date` de `pl.Datetime` como dtypes separados — más preciso sobre la semántica real de la columna.

## El namespace `.dt`

```python
df.select(pl.col("fecha").dt.year())
df.select(pl.col("fecha").dt.month())
df.select(pl.col("fecha").dt.weekday())
df.select(pl.col("fecha").dt.strftime("%B %Y"))     # formatear como string custom
```

Igual que el accessor `.dt` de Pandas (ver [[Python/Pandas/11 - Series de Tiempo#Componentes de fecha con el accessor .dt|Python/Pandas]]) — la sintaxis y el propósito son prácticamente idénticos, dentro del mismo mecanismo general de expresiones encadenables de Polars.

## `group_by_dynamic()` — el equivalente a `resample()` de Pandas

```python
(
    df.sort("fecha")
    .group_by_dynamic("fecha", every="1mo")
    .agg(pl.col("ventas").sum())
)
```

`group_by_dynamic()` requiere que los datos estén **ordenados** por la columna de tiempo (a diferencia de `resample()` de Pandas, que opera sobre un `DatetimeIndex` ya ordenado por construcción) — agrupa en ventanas de tiempo de tamaño fijo (`every="1mo"`, `"1w"`, `"1d"`, `"1h"`) y aplica una agregación a cada una.

```python
df.group_by_dynamic("fecha", every="1d", period="7d").agg(pl.col("ventas").mean())   # ventana de 7 días, cada 1 día -> equivalente a rolling
```

Con `period` distinto de `every`, `group_by_dynamic()` también puede replicar el comportamiento de una ventana móvil (`rolling`), unificando ambos conceptos (resample y rolling) en una sola función más flexible que su equivalente en Pandas.

## Funciones de ventana móvil directas

```python
df.with_columns(pl.col("ventas").rolling_mean(window_size=7).alias("media_movil_7d"))
df.with_columns(pl.col("ventas").rolling_std(window_size=30).alias("std_movil_30d"))
```

Equivalente a `.rolling().mean()` de Pandas (ver [[Python/Pandas/11 - Series de Tiempo#rolling() — ventanas móviles|Python/Pandas]]) — en Polars, cada tipo de ventana móvil (`rolling_mean`, `rolling_sum`, `rolling_std`) es su propio método de expresión en vez de un objeto intermedio `.rolling()` con método encadenado.

## `shift()`, `diff()`, `pct_change()`

```python
df.with_columns(pl.col("ventas").shift(1).alias("ventas_periodo_anterior"))
df.with_columns(pl.col("ventas").diff().alias("variacion_absoluta"))
df.with_columns(pl.col("ventas").pct_change().alias("variacion_pct"))
```

Mismos nombres y comportamiento que en Pandas — ver [[Python/Pandas/11 - Series de Tiempo#shift(), diff() y pct_change() — comparar contra periodos anteriores|Python/Pandas]].

## Zonas horarias

```python
df.with_columns(pl.col("fecha").dt.replace_time_zone("America/Mexico_City"))
df.with_columns(pl.col("fecha").dt.convert_time_zone("UTC"))
```

Mismo principio que `tz_localize`/`tz_convert` de Pandas: `replace_time_zone` se usa una vez para declarar el timezone de datos naive; `convert_time_zone` para pasar de una zona horaria ya asignada a otra.

## Ver también

- [[10 - Reshaping]]
- [[12 - Texto y Categóricos]]
- [[Python/Pandas/11 - Series de Tiempo|Python/Pandas]]
