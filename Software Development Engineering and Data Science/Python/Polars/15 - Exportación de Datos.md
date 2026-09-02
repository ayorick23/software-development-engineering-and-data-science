---
tags: [polars, python, data-science, io, export, cheat-sheet]
---

# 15 — Exportación de Datos

> Continúa de [[14 - Streaming y Datasets Grandes]]. El complemento simétrico de [[03 - Creación y Carga de Datos]].

## `write_csv()`

```python
df.write_csv("salida.csv")
df.write_csv("salida.csv", separator=";")
```

## `write_parquet()` — formato recomendado para datos intermedios

```python
df.write_parquet("datos.parquet")
df.write_parquet("datos.parquet", compression="zstd")     # zstd: buen balance velocidad/compresión, default moderno
```

Igual que en Pandas (ver [[Python/Pandas/14 - Exportación de Datos#to_parquet() — formato recomendado para datos intermedios|Python/Pandas]]), Parquet es el formato preferido para persistir resultados intermedios de un pipeline — preserva dtypes exactos y es mucho más rápido de leer/escribir que CSV en volúmenes medianos-grandes. Con Polars la ventaja es aún mayor porque Parquet es nativamente columnar, el mismo formato de memoria que Polars usa internamente (Arrow).

## `sink_parquet()` — escribir directamente desde un LazyFrame, sin materializar en memoria

```python
(
    pl.scan_csv("archivo_enorme.csv")
    .filter(pl.col("monto") > 100)
    .sink_parquet("resultado.parquet")     # escribe DIRECTO a disco, en streaming, sin pasar por collect() primero
)
```

`sink_parquet()` (y sus equivalentes `sink_csv()`, `sink_ipc()`) son exclusivos de la API lazy: en vez de `.collect()` (que materializa el resultado completo en memoria) seguido de `.write_parquet()`, `sink_*()` transmite el resultado directo a disco en modo streaming — la forma correcta de procesar y guardar un dataset más grande que la RAM sin nunca tener el resultado completo en memoria a la vez.

## `write_json()` / `write_ndjson()`

```python
df.write_json("datos.json")
df.write_ndjson("datos.ndjson")     # JSON delimitado por líneas — una fila por línea, más eficiente para archivos grandes
```

## `write_excel()`

```python
df.write_excel("reporte.xlsx", worksheet="Ventas")     # requiere el paquete 'xlsxwriter' instalado
```

## `write_database()`

```python
df.write_database(table_name="ventas_procesadas", connection="postgresql://user:pass@host/db", if_table_exists="replace")
```

Equivalente a `to_sql()` de Pandas (ver [[Python/Pandas/14 - Exportación de Datos#to_sql()|Python/Pandas]]).

## Ver también

- [[03 - Creación y Carga de Datos]]
- [[14 - Streaming y Datasets Grandes]]
- [[Python/Pandas/14 - Exportación de Datos|Python/Pandas]]
