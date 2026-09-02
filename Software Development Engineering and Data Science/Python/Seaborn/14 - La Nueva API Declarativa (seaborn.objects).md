---
tags: [seaborn, python, data-science, visualization, cheat-sheet]
---

# 14 — La Nueva API Declarativa (seaborn.objects)

> Continúa de [[13 - Estadística Integrada]].

## Qué es `seaborn.objects`

Desde Seaborn 0.12, existe una **segunda API completa**, más declarativa y modular, expuesta bajo `seaborn.objects` (convencionalmente importada como `so`). No reemplaza a la API clásica (`sns.scatterplot`, `sns.barplot`...) — ambas coexisten y probablemente seguirán haciéndolo por mucho tiempo, pero `so` resuelve limitaciones de composición que la API clásica no puede resolver limpiamente.

```python
import seaborn.objects as so

(
    so.Plot(df, x="precio", y="demanda", color="region")
    .add(so.Dot())
)
```

## La estructura: capas explícitas (`Plot` + `.add()`)

```python
(
    so.Plot(df, x="fecha", y="ventas", color="region")
    .add(so.Line())
)
```

Inspirada en la "Grammar of Graphics" (la misma filosofía detrás de ggplot2 de R y de Plotly Express) — un gráfico se construye declarando datos + mapeos estéticos (`Plot(...)`) y luego **agregando capas** de marcas visuales (`.add(so.Dot())`, `.add(so.Line())`) de forma explícita y componible, en vez de una función monolítica por tipo de gráfico como en la API clásica.

## Combinar múltiples capas en un solo gráfico — el problema que resuelve

```python
# Con la API clásica, combinar puntos + línea de tendencia agregada requiere DOS llamadas separadas
sns.scatterplot(data=df, x="precio", y="demanda", alpha=0.3)
sns.lineplot(data=df, x="precio", y="demanda", estimator="mean")

# Con seaborn.objects, se componen como capas de UN SOLO objeto Plot
(
    so.Plot(df, x="precio", y="demanda")
    .add(so.Dot(alpha=0.3))
    .add(so.Line(), so.Agg())      # una transformación estadística (Agg) aplicada específicamente a esta capa
)
```

La ventaja real de `so` aparece aquí: cada capa puede tener su **propia** transformación estadística y su propia marca visual, compuestas explícitamente — la API clásica no tiene una forma limpia de decir "puntos crudos en una capa, línea agregada en otra, en el mismo gráfico" sin dos llamadas independientes que coincidan en los mismos ejes por casualidad.

## Transformaciones estadísticas explícitas

```python
so.Plot(df, x="region", y="ventas").add(so.Bar(), so.Agg("mean"))       # equivalente a sns.barplot
so.Plot(df, x="region", y="ventas").add(so.Dots(), so.Jitter())          # equivalente a sns.stripplot
so.Plot(df, x="precio").add(so.Bars(), so.Hist())                          # equivalente a sns.histplot
```

Cada transformación (`so.Agg`, `so.Jitter`, `so.Hist`, `so.KDE`) es un objeto independiente que se combina con una marca (`so.Bar`, `so.Dot`, `so.Line`) — la misma idea de composición de capas aplicada ahora a la parte estadística, no solo visual.

## Facetado en la nueva API

```python
so.Plot(df, x="precio", y="demanda", color="canal").facet(col="region").add(so.Dot())
```

## Cuándo usar `seaborn.objects` vs la API clásica

| | API clásica (`sns.scatterplot`...) | `seaborn.objects` (`so.Plot`) |
|---|---|---|
| Madurez | Estable desde hace años, toda la documentación/ejemplos externos la usan | Más nueva (0.12+), API aún evolucionando entre versiones |
| Composición de capas | Limitada, requiere múltiples llamadas coordinadas manualmente | Nativa, explícita |
| Curva de aprendizaje | Baja si ya se conoce Matplotlib | Requiere aprender un modelo mental nuevo (Grammar of Graphics) |
| Uso recomendado hoy | Default para la gran mayoría de casos | Cuando se necesita combinar varias capas/transformaciones en un solo gráfico complejo |

**Regla práctica actual:** seguir usando la API clásica (el resto de este cheat-sheet) como default — es más madura, mejor documentada, y suficiente para la inmensa mayoría de gráficos exploratorios. Reservar `seaborn.objects` para el caso específico de necesitar componer múltiples capas con distintas transformaciones estadísticas en un solo gráfico, algo que la API clásica no resuelve limpiamente.

## Ver también

- [[13 - Estadística Integrada]]
- [[01 - Introducción y Arquitectura]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
