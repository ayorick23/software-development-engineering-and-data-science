---
tags: [seaborn, python, data-science, visualization, tidy-data, cheat-sheet]
---

# 02 — Formato de Datos: Long vs Wide

> Continúa de [[01 - Introducción y Arquitectura]]. Entender esta distinción es el prerequisito real para usar Seaborn bien — más que memorizar funciones.

## Por qué Seaborn es "opinionado" sobre el formato de datos

Seaborn está diseñado para trabajar mejor con datos en formato **long** (o "tidy"): cada fila es una observación individual, cada columna es una variable. Este formato es exactamente el que produce [[Python/Pandas/10 - Reshaping y Pivoting|`melt()` de Pandas]].

```python
# WIDE — una columna por producto (intuitivo para leer, difícil de graficar con Seaborn)
#   fecha       producto_A  producto_B  producto_C
#   2026-01-01  100         200         150

# LONG / TIDY — una fila por observación (lo que Seaborn espera)
#   fecha       producto    valor
#   2026-01-01  producto_A  100
#   2026-01-01  producto_B  200
#   2026-01-01  producto_C  150
```

## El mismo gráfico, mucho más simple en formato long

```python
# Con datos WIDE, graficar 3 líneas requiere 3 llamadas manuales
fig, ax = plt.subplots()
ax.plot(df_wide["fecha"], df_wide["producto_A"], label="A")
ax.plot(df_wide["fecha"], df_wide["producto_B"], label="B")
ax.plot(df_wide["fecha"], df_wide["producto_C"], label="C")
ax.legend()

# Con datos LONG, Seaborn lo resuelve en una sola llamada usando 'hue'
df_long = df_wide.melt(id_vars="fecha", var_name="producto", value_name="valor")
sns.lineplot(data=df_long, x="fecha", y="valor", hue="producto")
```

Este es el beneficio concreto del formato long: el argumento `hue=` (y `size=`, `style=`, `col=`, `row=` — ver [[03 - Gráficos de Relación]] y [[09 - FacetGrid - Small Multiples]]) reemplaza por completo la necesidad de loops manuales sobre cada categoría.

## Convertir de wide a long antes de graficar

```python
df_long = df_wide.melt(
    id_vars=["fecha"],
    value_vars=["producto_A", "producto_B", "producto_C"],
    var_name="producto",
    value_name="valor",
)
sns.lineplot(data=df_long, x="fecha", y="valor", hue="producto")
```

Ver el detalle completo de `melt()`/`pivot()` en [[Python/Pandas/10 - Reshaping y Pivoting|Python/Pandas]] — es, en la práctica, el paso de preparación de datos más común antes de cualquier gráfico de Seaborn con múltiples series.

## Seaborn SÍ acepta formato wide directamente (con limitaciones)

```python
sns.lineplot(data=df_wide.set_index("fecha"))     # funciona, pero SIN control fino sobre hue/size/style por columna
sns.heatmap(df_wide.set_index("fecha"))              # heatmap SÍ está pensado para wide (ver 08)
```

Algunas funciones (`heatmap`, `clustermap`) están diseñadas específicamente para datos wide (una matriz), mientras que la mayoría de las funciones estadísticas (`scatterplot`, `barplot`, `boxplot`...) dan mucho más control cuando reciben datos long con columnas explícitas para `x`, `y`, `hue`.

## Verificar que los datos ya están "tidy"

Un dataset está en formato tidy cuando se puede responder que sí a las tres preguntas: ¿cada columna es una sola variable?, ¿cada fila es una sola observación?, ¿cada celda es un solo valor? Si una columna mezcla varias variables (como `producto_A`, `producto_B` siendo en realidad la misma variable "producto" con distintos valores), el dataset está en wide y probablemente conviene `melt()` antes de graficar.

## Ver también

- [[01 - Introducción y Arquitectura]]
- [[03 - Gráficos de Relación]]
- [[09 - FacetGrid - Small Multiples]]
- [[Python/Pandas/10 - Reshaping y Pivoting|Python/Pandas]]
