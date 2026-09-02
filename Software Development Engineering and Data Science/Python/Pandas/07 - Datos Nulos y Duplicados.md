---
tags: [pandas, python, data-science, missing-data, cheat-sheet]
---

# 07 — Datos Nulos y Duplicados

> Continúa de [[06 - Operaciones con Columnas]].

## Detección de nulos

```python
df.isna()                # máscara booleana, True donde hay NaN/None/NaT/pd.NA
df.isna().sum()            # nulos por columna
df.isna().sum(axis=1)      # nulos por fila
df.notna()                 # complemento de isna()
df["precio"].isna().any()  # ¿hay al menos un nulo en esta columna?
```

Pandas trata como "nulo" varios objetos según el contexto: `np.nan` (float), `None` (Python), `pd.NaT` (fechas) y `pd.NA` (el nulo genérico de los Extension Arrays, ver [[Python/Pandas/01 - Introducción y Arquitectura Interna]]).

## `fillna()` — imputación de valores faltantes

```python
df["precio"].fillna(0)                          # valor fijo
df["precio"].fillna(df["precio"].mean())         # imputación con la media
df["categoria"].fillna(df["categoria"].mode()[0])  # imputación con la moda (para categóricas)

df.fillna(method="ffill")    # forward fill: propaga el último valor válido hacia adelante
df.fillna(method="bfill")    # backward fill: propaga el siguiente valor válido hacia atrás

df.fillna({"precio": 0, "categoria": "Desconocido"})   # valores distintos por columna, en una sola llamada
```

`ffill`/`bfill` son especialmente relevantes en series de tiempo, donde "el último valor conocido" suele ser una imputación más razonable que la media global — ver [[11 - Series de Tiempo]].

## `interpolate()` — imputación numérica basada en tendencia

```python
df["temperatura"].interpolate(method="linear")     # interpola linealmente entre valores conocidos
df["temperatura"].interpolate(method="time")        # pondera por la distancia temporal real (requiere DatetimeIndex)
df["temperatura"].interpolate(method="polynomial", order=2)
```

## `dropna()` — eliminar filas/columnas con nulos

```python
df.dropna()                          # elimina cualquier fila con AL MENOS un nulo
df.dropna(how="all")                  # elimina solo filas completamente vacías
df.dropna(subset=["precio", "stock"]) # considera nulos solo en estas columnas
df.dropna(thresh=3)                    # conserva filas con al menos 3 valores NO nulos
df.dropna(axis=1)                      # elimina COLUMNAS con nulos, en vez de filas
```

## Estrategia de decisión para nulos

| Situación | Estrategia recomendada |
|---|---|
| Pocos nulos (<5%), aleatorios | `dropna()` de esas filas |
| Muchos nulos en una columna poco relevante | eliminar la columna (`drop(columns=...)`) |
| Numérica, distribución simétrica | `fillna(mean())` |
| Numérica, con outliers/asimétrica | `fillna(median())` |
| Categórica | `fillna(mode())` o categoría explícita `"Desconocido"` |
| Serie de tiempo | `ffill()`/`bfill()`/`interpolate(method="time")` |
| Nulo con significado propio (ej. "no aplica") | **no imputar** — crear columna indicadora `es_nulo` y dejar explícito |

Para pipelines de ML formales con imputación como paso reproducible dentro de un `Pipeline`, ver `SimpleImputer`/`KNNImputer` en [[11 - Datos Faltantes y Clases Desbalanceadas|Scikit-learn]].

## Duplicados

```python
df.duplicated()                          # True en cada fila que es un duplicado de una anterior
df.duplicated(subset=["email"])          # duplicado según solo estas columnas
df.duplicated(keep="first")              # default: marca la 2da+ ocurrencia como duplicada
df.duplicated(keep="last")               # marca todas menos la ÚLTIMA ocurrencia
df.duplicated(keep=False)                # marca TODAS las ocurrencias involucradas en un duplicado

df.drop_duplicates()                      # elimina duplicados exactos
df.drop_duplicates(subset=["email"], keep="last")   # se queda con el registro más reciente por email
```

## `combine_first()` — rellenar nulos de un DataFrame con otro

```python
df_principal.combine_first(df_respaldo)   # donde df_principal tiene NaN, usa el valor de df_respaldo
```

Útil cuando se tienen dos fuentes de la misma entidad (ej. dos exports parciales) y se quiere una versión "consolidada" priorizando la primera fuente.

## Ver también

- [[06 - Operaciones con Columnas]]
- [[08 - Agrupación y Agregación (GroupBy)]]
- [[11 - Series de Tiempo]]
- [[11 - Datos Faltantes y Clases Desbalanceadas|Scikit-learn]]
- [[07 - Coerción, Tipos de Datos y Filtrado de Filas Inválidas|Pandera]] — validar que ya no queden nulos antes de modelar
