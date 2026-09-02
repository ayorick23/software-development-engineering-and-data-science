---
tags: [pandas, python, data-science, best-practices, cheat-sheet]
---

# 17 — Buenas Prácticas, Errores Comunes y Comparativa

> Cierra la serie iniciada en [[Python/Pandas/01 - Introducción y Arquitectura Interna]].

## `SettingWithCopyWarning` — el warning más confuso de Pandas

```python
# Genera el warning — Pandas no puede garantizar si 'subset' es una VISTA o una COPIA de df
subset = df[df["precio"] > 100]
subset["en_oferta"] = True   # ¿modificó df? depende de detalles internos — comportamiento AMBIGUO
```

El problema es indexación **encadenada**: `df[condicion]` devuelve algo que puede ser vista o copia según el layout interno del `BlockManager` (ver [[Python/Pandas/01 - Introducción y Arquitectura Interna]]), y una segunda operación de indexación (`["en_oferta"] = True`) sobre ese resultado intermedio no tiene forma confiable de propagarse al DataFrame original.

```python
# CORRECTO — una sola operación .loc, sin ambigüedad
df.loc[df["precio"] > 100, "en_oferta"] = True

# Si de verdad se necesita trabajar con el subset por separado, copiarlo EXPLÍCITAMENTE
subset = df[df["precio"] > 100].copy()
subset["en_oferta"] = True   # ahora es inequívocamente una copia independiente, sin warning
```

**Regla práctica:** cualquier asignación siempre en una sola llamada a `.loc[fila, columna] = valor`; si se necesita un DataFrame separado del original, llamar `.copy()` explícitamente de inmediato.

## Copy-on-Write (CoW) resuelve la ambigüedad de raíz

Con `pd.set_option("mode.copy_on_write", True)` (default en pandas 3.0), toda operación que "parece" devolver una vista se comporta consistentemente como si fuera una copia perezosa — el código incorrecto de arriba deja de modificar `df` silenciosamente (en vez de comportamiento ambiguo, ahora es predecible: nunca modifica el original sin `.loc` explícito). Ver el detalle en [[15 - Rendimiento y Optimización#Copy-on-Write (CoW)|Rendimiento y Optimización]].

## Comparar con `==` cuando hay `NaN` de por medio

```python
# INCORRECTO — NaN == NaN es False, este filtro pierde silenciosamente las filas nulas
df[df["categoria"] == "Desconocido"]

# CORRECTO si se quiere incluir nulos explícitamente
df[df["categoria"].isna() | (df["categoria"] == "Desconocido")]
```

## Modificar un DataFrame mientras se itera sobre él

```python
# INCORRECTO — modificar df dentro de un loop que itera sobre df es frágil e impredecible
for idx, fila in df.iterrows():
    if fila["stock"] == 0:
        df.drop(idx, inplace=True)   # puede saltarse filas o lanzar errores según la versión

# CORRECTO — construir la máscara y filtrar de una vez
df = df[df["stock"] > 0]
```

## Encadenar `inplace=True` no es más rápido, y rompe el encadenamiento

```python
# inplace=True NO evita una copia interna en la mayoría de los métodos, y no se puede encadenar
df.dropna(inplace=True)
df.reset_index(inplace=True, drop=True)

# preferible: encadenar métodos que devuelven un nuevo objeto — más legible, mismo costo real
df = df.dropna().reset_index(drop=True)
```

`inplace=True` es en gran medida un mito de rendimiento: internamente casi todos los métodos calculan igual un resultado nuevo y solo reasignan la referencia — el beneficio real es marginal y el costo en legibilidad (no se puede encadenar, dificulta debugging) generalmente no vale la pena.

## Comparación de tipo antes de comparar valor

```python
# fecha_col es datetime64, comparar contra un string funciona por conversión implícita, pero es frágil
df[df["fecha"] > "2026-01-01"]   # funciona, pero mejor:
df[df["fecha"] > pd.Timestamp("2026-01-01")]   # explícito, sin depender de conversión automática
```

## Comparativa: Pandas vs Polars vs Dask vs PySpark

| | Pandas | Polars | Dask | PySpark |
|---|---|---|---|---|
| Motor | NumPy / (opcional) Arrow | Rust, Arrow nativo | Pandas fragmentado | JVM distribuido |
| Paralelismo | No (single-thread por defecto) | Sí, multi-hilo nativo | Sí, multi-proceso/cluster | Sí, cluster distribuido |
| Tamaño de dato ideal | Cabe en RAM de una máquina | Cabe en RAM, pero se beneficia de más núcleos | Más grande que RAM, una máquina o cluster chico | Terabytes, cluster grande |
| API | Estándar de facto | Similar pero no idéntica (expresiones "lazy") | Calcada de Pandas | Propia (basada en Spark SQL/DataFrame) |
| Madurez del ecosistema | Máxima (10+ años, todo lo integra) | Creciente rápido | Madura en el nicho "Pandas grande" | Máxima en Big Data empresarial |
| Cuándo usar | Default para todo lo que quepa en memoria | Cuando Pandas es el cuello de botella y no se justifica un cluster | Escalar código Pandas existente sin reescribir todo | Big Data ya viviendo en un cluster Spark/Databricks |

**Regla práctica:** empezar siempre en Pandas. Migrar a Polars si el cuello de botella es CPU/memoria en una sola máquina y se puede reescribir la capa de datos. Migrar a Dask si se quiere reusar código Pandas existente casi sin cambios mientras escala. Migrar a PySpark solo cuando el volumen o la infraestructura ya exige un cluster distribuido real (ver `Machine Learning/07-Librerias-de-Data-Science-y-ML.md`).

Ver la comparativa completa y mucho más detallada (arquitectura, paralelismo, benchmarks, árbol de decisión) en [[Python/Polars/17 - Buenas Prácticas, Errores Comunes y Comparativa#Comparativa exhaustiva Polars vs Pandas vs Dask vs PySpark|Python/Polars]].

## Checklist final antes de dar un pipeline por "listo"

- [ ] ¿Las asignaciones condicionales usan `.loc[cond, col] = valor` (nunca indexación encadenada)?
- [ ] ¿Los dtypes están declarados explícitamente en la carga (`dtype=`), no dependiendo de inferencia?
- [ ] ¿Las columnas categóricas de baja cardinalidad usan `category`?
- [ ] ¿Hay validación de esquema formal antes de que el DataFrame llegue al modelo? → [[Python/Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]]
- [ ] ¿Las transformaciones fila-por-fila (`apply(axis=1)`) tienen de verdad razón de ser, o pueden vectorizarse? → [[15 - Rendimiento y Optimización]]

## Ver también

- [[Python/Pandas/01 - Introducción y Arquitectura Interna]]
- [[04 - Selección e Indexación]]
- [[15 - Rendimiento y Optimización]]
- [[Python/Pandas/16 - Integración con el Ecosistema]]
- [[Python/Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]]
