---
tags: [seaborn, python, data-science, visualization, facetgrid, cheat-sheet]
---

# 09 — FacetGrid: Small Multiples

> Continúa de [[08 - Mapas de Calor y Matrices]]. El motor interno detrás de todas las funciones figure-level.

## Qué es un `FacetGrid`

Un `FacetGrid` divide los datos en subconjuntos según una o más variables categóricas, y dibuja el **mismo tipo de gráfico** una vez por cada subconjunto — el patrón de visualización conocido como "small multiples" o "trellis plot".

```python
g = sns.FacetGrid(df, col="region")
g.map(sns.scatterplot, "precio", "demanda")
```

Este es el mecanismo de bajo nivel; en la práctica, casi siempre se usa la versión de alto nivel ya vista en archivos anteriores (`relplot`, `catplot`, `lmplot`, `displot`), que internamente crean y administran un `FacetGrid` automáticamente:

```python
# Equivalente de alto nivel al FacetGrid manual de arriba
sns.relplot(data=df, x="precio", y="demanda", col="region", kind="scatter")
```

## Facetado por columnas, filas, o ambas

```python
sns.relplot(data=df, x="precio", y="demanda", col="region")                   # un panel por región, en columnas
sns.relplot(data=df, x="precio", y="demanda", row="canal")                      # un panel por canal, en filas
sns.relplot(data=df, x="precio", y="demanda", col="region", row="canal")          # cuadrícula completa: región × canal
sns.relplot(data=df, x="precio", y="demanda", col="region", hue="canal")            # facetado + color combinados
```

## Controlar el layout de la cuadrícula

```python
sns.relplot(data=df, x="precio", y="demanda", col="region", col_wrap=3)     # máximo 3 columnas, el resto pasa a nueva fila
sns.relplot(data=df, x="precio", y="demanda", col="region", height=4, aspect=1.2)   # tamaño de CADA panel individual
sns.relplot(data=df, x="precio", y="demanda", col="region", col_order=["Norte", "Sur", "Centro"])   # orden explícito de paneles
```

`col_wrap` es esencial cuando la variable de facetado tiene muchos niveles (más de 4-5) — sin él, `col=` sola produciría una única fila extremadamente ancha, difícil de ver completa en pantalla.

## Personalizar cada panel del FacetGrid manualmente

```python
g = sns.relplot(data=df, x="fecha", y="ventas", col="region", kind="line")
g.set_titles("Región: {col_name}")                # título de cada panel, con placeholder de la variable de facetado
g.set_axis_labels("Fecha", "Ventas (USD)")           # etiquetas de eje compartidas
g.set(ylim=(0, None))                                  # aplica el mismo límite de eje Y a TODOS los paneles
g.fig.suptitle("Ventas por región a lo largo del tiempo", y=1.02)   # título general de toda la Figure

for ax in g.axes.flat:                               # acceso directo a cada Axes individual si se necesita más control
    ax.axhline(y=1000, color="red", linestyle="--")
```

El objeto devuelto por una función figure-level (`g`) da acceso a `g.axes` (array de todos los `Axes`, igual que `plt.subplots()`) y `g.fig` (la `Figure` completa) — cualquier personalización de Matplotlib sigue siendo posible después del facetado automático.

## Cuándo usar FacetGrid vs `hue=` solo

| | `hue=` (superpuesto) | Facetado (`col=`/`row=`) |
|---|---|---|
| Comparación | Todas las categorías en el MISMO panel | Cada categoría en SU PROPIO panel |
| Mejor con | Pocas categorías (2-4), donde la superposición es legible | Muchas categorías, o cuando la superposición se vuelve confusa |
| Ventaja | Comparación directa en el mismo eje | Cada patrón individual se ve con claridad, sin solaparse |

## Ver también

- [[08 - Mapas de Calor y Matrices]]
- [[10 - Gráficos Multivariados]]
- [[01 - Introducción y Arquitectura]]
