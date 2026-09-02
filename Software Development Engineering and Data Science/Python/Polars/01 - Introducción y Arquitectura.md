---
tags: [polars, python, data-science, dataframe, cheat-sheet]
---

# 01 — Introducción y Arquitectura

> Complementa la sección `### Polars` de [[Machine Learning/04-Ingenieria-de-Datos#Polars|Machine Learning/04 - Ingeniería de Datos]] con la profundidad práctica de sintaxis.

**Polars** es una librería de manipulación de DataFrames escrita en **Rust**, diseñada desde cero (no es una evolución de Pandas) para resolver limitaciones estructurales conocidas de [[Python/Pandas/01 - Introducción y Arquitectura Interna|Pandas]]: paralelismo automático multi-núcleo, uso de memoria más eficiente, y un motor de consultas que puede **optimizar** el trabajo antes de ejecutarlo.

```python
import polars as pl
```

## Las tres decisiones de diseño que la diferencian de Pandas

### 1. Motor en Rust, no en Python/C

Pandas está construida sobre NumPy (arrays de C); Polars está escrita en Rust desde su núcleo, con enlaces (bindings) a Python. Esto le da paralelización multi-hilo **automática** en la mayoría de operaciones, sin que el usuario tenga que hacer nada especial — Pandas, en cambio, es single-threaded por default.

### 2. Apache Arrow como formato de memoria nativo

```python
df.to_arrow()          # conversión sin copia (zero-copy) hacia el formato Arrow
```

Polars usa el formato columnar de **Apache Arrow** como su representación interna — el mismo formato que usan DuckDB, y que Pandas solo soporta parcialmente vía el backend opcional `dtype_backend="pyarrow"` (ver [[Python/Pandas/15 - Rendimiento y Optimización#Backend PyArrow (dtype_backend)|Python/Pandas]]). Esto permite interoperar con otras herramientas del ecosistema Arrow sin copiar datos.

### 3. Sin índice (`Index`)

```python
# Pandas: cada DataFrame tiene un Index explícito, con su propio dtype y semántica de alineación
df_pandas.index

# Polars: NO existe el concepto de índice — cada fila se identifica solo por su posición
df_polars[0]     # simplemente la primera fila, no hay "etiquetas" de fila
```

Esta es, en la práctica, la diferencia conceptual más profunda con Pandas: no hay `Index`, no hay alineación automática por etiqueta en operaciones aritméticas, y no hay `.loc`/`.iloc` con la misma semántica — ver el detalle completo de por qué esto simplifica (y a la vez limita) la API en [[17 - Buenas Prácticas, Errores Comunes y Comparativa]].

## `DataFrame` — la estructura básica

```python
df = pl.DataFrame({
    "producto": ["A", "B", "C"],
    "precio": [10.5, 22.0, 7.25],
    "stock": [100, 50, 200],
})
print(df)
```

Visualmente similar a un DataFrame de Pandas, pero sin la columna de índice a la izquierda — solo las columnas de datos reales.

## Eager vs Lazy: el otro pilar de la arquitectura

```python
pl.DataFrame(...)        # API "eager" — ejecuta cada operación inmediatamente, como Pandas
pl.LazyFrame(...)          # API "lazy" — construye un PLAN de ejecución, optimiza, y ejecuta solo al final con .collect()
```

Esta distinción es tan central que tiene su propio archivo dedicado — ver [[02 - Eager vs Lazy API]].

## Ver también

- [[02 - Eager vs Lazy API]]
- [[04 - Expresiones (pl.col)]]
- [[16 - Integración con el Ecosistema]]
- [[Python/Pandas/01 - Introducción y Arquitectura Interna|Python/Pandas]]
- [[Machine Learning/04-Ingenieria-de-Datos#Polars|Machine Learning/04 - Ingeniería de Datos]]
