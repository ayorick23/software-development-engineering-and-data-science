---
tags: [pandas, python, data-science, strings, categorical, cheat-sheet]
---

# 12 — Texto y Datos Categóricos

> Continúa de [[11 - Series de Tiempo]].

## El accessor `.str` — vectoriza operaciones de string

```python
df["nombre"].str.upper()
df["nombre"].str.lower()
df["nombre"].str.strip()                          # quita espacios al inicio/final
df["nombre"].str.len()                             # longitud de cada string
df["email"].str.split("@").str[0]                  # split + acceso al primer elemento resultante
df["nombre"].str.replace(" ", "_", regex=False)    # reemplazo literal (más rápido si no se necesita regex)
df["telefono"].str.replace(r"\D", "", regex=True)  # regex: elimina todo lo que no sea dígito
```

Todo método de string nativo de Python (`.upper()`, `.strip()`, `.split()`...) tiene su equivalente vectorizado en `.str.*` — la diferencia es que `.str.*` aplica el método a **cada elemento de la Series de una sola llamada**, y maneja nulos automáticamente (devuelve `NaN` en vez de lanzar `AttributeError`).

## Extracción con expresiones regulares

```python
df["producto"].str.contains(r"^SKU-\d+$", regex=True, na=False)
df["email"].str.extract(r"(?P<usuario>[\w.]+)@(?P<dominio>[\w.]+)")   # captura grupos -> nuevas columnas
df["texto"].str.findall(r"\d+")                                        # todas las coincidencias, como lista por celda
```

`na=False` en `.str.contains()` es importante: sin especificarlo, una fila con valor nulo produce `NaN` en el resultado booleano en vez de `False`, lo que puede romper un filtro posterior (`df[mascara]` falla con NaN en la máscara).

## `split()` con `expand=True` — de una columna a varias

```python
df[["nombre", "apellido"]] = df["nombre_completo"].str.split(" ", n=1, expand=True)
```

## Dtype `category` — cuándo y por qué

```python
df["region"] = df["region"].astype("category")
```

Convertir a `category` cuando una columna `object` (string) tiene **pocos valores únicos repetidos muchas veces** (ej. región, estado, tipo de producto). Internamente Pandas almacena solo los códigos enteros y un mapa de categorías, no el string repetido en cada fila — reduce memoria drásticamente en columnas de baja cardinalidad. Ver el análisis de memoria completo en [[15 - Rendimiento y Optimización]].

```python
df["region"].cat.categories                  # valores únicos posibles
df["region"].cat.codes                        # representación entera interna
df["region"] = df["region"].cat.add_categories(["Nueva Región"])
df["region"] = df["region"].cat.remove_unused_categories()
```

### Categorías ordenadas

```python
tallas = pd.CategoricalDtype(categories=["S", "M", "L", "XL"], ordered=True)
df["talla"] = df["talla"].astype(tallas)

df[df["talla"] > "M"]              # comparación válida gracias al orden explícito — con object sería alfabético/incorrecto
df.sort_values("talla")             # ordena según el orden lógico definido, no alfabético
```

Sin `ordered=True` y el orden explícito de categorías, comparar u ordenar una columna tipo "S/M/L/XL" con `object` normal daría un resultado alfabético (`L < M < S < XL`), incorrecto para la semántica real de tallas.

## `cut()` y `qcut()` — discretizar variables numéricas en categorías

```python
df["rango_edad"] = pd.cut(df["edad"], bins=[0, 18, 35, 60, 100], labels=["Menor", "Joven", "Adulto", "Mayor"])
df["cuartil_ingreso"] = pd.qcut(df["ingreso"], q=4, labels=["Q1", "Q2", "Q3", "Q4"])
```

`cut()` divide en intervalos de **ancho fijo definido manualmente** (útil cuando los cortes tienen significado de negocio, como rangos de edad); `qcut()` divide en cuantiles, garantizando **igual número de observaciones** por categoría — el resultado de ambos es directamente un dtype `category`, listo para `groupby` o modelado.

## Ver también

- [[11 - Series de Tiempo]]
- [[13 - MultiIndex y Datos Jerárquicos]]
- [[15 - Rendimiento y Optimización]]
- [[02 - Preprocessing y Escalado|Scikit-learn]] — encoding de categóricas para modelos
