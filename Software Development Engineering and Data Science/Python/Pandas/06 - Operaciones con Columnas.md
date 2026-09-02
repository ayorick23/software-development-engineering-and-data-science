---
tags: [pandas, python, data-science, transformation, cheat-sheet]
---

# 06 — Operaciones con Columnas

> Continúa de [[05 - Filtros y Condiciones Avanzadas]].

## Operaciones vectorizadas — siempre la primera opción

```python
df["total"] = df["precio"] * df["cantidad"]           # aritmética elemento a elemento, sin loops
df["precio_con_iva"] = df["precio"] * 1.16
df["nombre_mayus"] = df["nombre"].str.upper()
```

Cualquier operación que pueda expresarse como aritmética/método vectorizado directo sobre la Series **debe** escribirse así, no con `.apply()` — ver la comparación de rendimiento en [[15 - Rendimiento y Optimización]].

## `map()` — transformar una `Series`, valor por valor

```python
df["region_cod"] = df["region"].map({"Norte": 1, "Sur": 2, "Centro": 3})   # con diccionario (lookup)
df["precio_str"] = df["precio"].map("${:.2f}".format)                       # con función
df["region_cod"] = df["region"].map(lambda r: mapping.get(r, -1))          # con lambda + default
```

`map()` **solo existe en `Series`**, nunca en `DataFrame` directamente — es la herramienta específica para transformar/traducir una sola columna valor por valor.

## `apply()` — aplicar una función a lo largo de un eje

```python
df["precio"].apply(lambda x: x * 1.16)              # sobre una Series, elemento por elemento (igual que map, pero más lento)

df.apply(lambda fila: fila["precio"] * fila["cantidad"], axis=1)   # sobre un DataFrame, fila por fila (axis=1)
df.apply(lambda col: col.max() - col.min(), axis=0)                 # columna por columna (axis=0, default)
```

`axis=1` (fila por fila) es la forma **más lenta** de transformar un DataFrame porque Pandas reconstruye un objeto `Series` por cada fila internamente. Casi siempre existe una alternativa vectorizada (aritmética directa, `np.where`, `.str.*`) — reservar `apply(axis=1)` solo para lógica verdaderamente no vectorizable (ej. llamar una función externa compleja por fila).

## `DataFrame.map()` (antes `applymap`) — elemento por elemento en TODO el DataFrame

```python
df[["precio", "stock"]].map(lambda x: x * 2)   # aplica a cada celda individual del subset
```

Desde pandas 2.1, `applymap()` está deprecado en favor de `DataFrame.map()` (mismo comportamiento, nombre consistente con `Series.map`).

## `assign()` — crear columnas nuevas sin mutar el original, encadenable

```python
df_resultado = (
    df
    .assign(total=lambda d: d["precio"] * d["cantidad"])
    .assign(total_con_iva=lambda d: d["total"] * 1.16)
    .query("total_con_iva > 100")
)
```

`assign()` devuelve un **nuevo** DataFrame (no modifica `df` in-place) y acepta lambdas que reciben el DataFrame intermedio de la cadena — el patrón estándar para pipelines de transformación legibles y encadenables sin variables intermedias.

## `pipe()` — encadenar funciones custom dentro de una cadena de métodos

```python
def quitar_outliers(df, columna, limite):
    return df[df[columna].abs() < limite]

def normalizar(df, columna):
    return df.assign(**{f"{columna}_norm": (df[columna] - df[columna].mean()) / df[columna].std()})

resultado = (
    df
    .pipe(quitar_outliers, columna="monto", limite=1000)
    .pipe(normalizar, columna="monto")
)
```

`pipe()` permite meter funciones propias dentro de una cadena fluida de métodos, en vez de anidarlas (`normalizar(quitar_outliers(df, ...), ...)`) — mejora la legibilidad de pipelines largos.

## `eval()` — evaluación vectorizada por texto (rendimiento)

```python
df.eval("total = precio * cantidad", inplace=True)
df["total_iva"] = df.eval("precio * cantidad * 1.16")
```

Igual que `query()` para filtros, `eval()` usa el motor `numexpr` para evitar crear arreglos intermedios en cada paso de una expresión aritmética — relevante en DataFrames de millones de filas. Ver [[15 - Rendimiento y Optimización]].

## Ver también

- [[05 - Filtros y Condiciones Avanzadas]]
- [[07 - Datos Nulos y Duplicados]]
- [[12 - Texto y Datos Categóricos]]
- [[15 - Rendimiento y Optimización]]
