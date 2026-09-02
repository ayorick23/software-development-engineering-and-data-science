---
tags: [seaborn, python, data-science, visualization, pairplot, cheat-sheet]
---

# 10 — Gráficos Multivariados

> Continúa de [[09 - FacetGrid - Small Multiples]].

## `pairplot()` — todas las relaciones por pares de un vistazo

```python
sns.pairplot(df)                                          # scatter de cada par de columnas numéricas + histograma en la diagonal
sns.pairplot(df, hue="region")                              # coloreado por categoría
sns.pairplot(df, vars=["precio", "demanda", "stock"])         # limitar a columnas específicas (evita una matriz gigante con muchas columnas)
sns.pairplot(df, diag_kind="kde")                               # KDE en vez de histograma en la diagonal
sns.pairplot(df, kind="reg")                                      # línea de regresión en cada scatter fuera de la diagonal
```

`pairplot()` es la herramienta estándar de **primer vistazo exploratorio** sobre un dataset nuevo: en una sola llamada muestra la distribución de cada variable (diagonal) y la relación entre cada par de variables (fuera de la diagonal) — el punto de partida antes de decidir qué relaciones investigar más a fondo con [[03 - Gráficos de Relación|scatterplot/lineplot]] o [[07 - Gráficos de Regresión|regplot]] individuales.

**Advertencia de rendimiento:** con `n` columnas numéricas, `pairplot()` dibuja `n²` subplots — con datasets de más de ~8-10 columnas numéricas se vuelve lento y visualmente saturado; usar `vars=` para limitar a las columnas de interés real.

## `PairGrid` — la versión de bajo nivel, personalizable por zona

```python
g = sns.PairGrid(df, hue="region")
g.map_diag(sns.histplot)                     # función distinta para la diagonal
g.map_upper(sns.scatterplot)                   # función distinta para el triángulo superior
g.map_lower(sns.kdeplot)                         # función distinta para el triángulo inferior
g.add_legend()
```

`PairGrid` es a `pairplot()` lo que `FacetGrid` es a `relplot()` — el mecanismo de bajo nivel que permite usar una función **distinta** en cada zona de la matriz (por ejemplo, KDE abajo y scatter arriba), algo que `pairplot()` con sus argumentos simples no puede lograr directamente.

## `jointplot()` — relación entre dos variables + sus distribuciones marginales

```python
sns.jointplot(data=df, x="precio", y="demanda")                       # scatter central + histogramas marginales en los bordes
sns.jointplot(data=df, x="precio", y="demanda", kind="hex")              # hexbin en el centro — mejor con muchos puntos superpuestos
sns.jointplot(data=df, x="precio", y="demanda", kind="kde")                # densidad conjunta 2D + marginales KDE
sns.jointplot(data=df, x="precio", y="demanda", kind="reg")                  # scatter + línea de regresión + marginales
sns.jointplot(data=df, x="precio", y="demanda", hue="region")                  # coloreado por categoría
```

`jointplot()` responde específicamente a la pregunta "¿cómo se relacionan estas DOS variables, y cómo se distribuye cada una individualmente?" en un solo gráfico compacto — el gráfico central muestra la relación conjunta, los histogramas/KDE en los márgenes superior/derecho muestran cada distribución marginal por separado.

## `JointGrid` — la versión de bajo nivel de `jointplot`

```python
g = sns.JointGrid(data=df, x="precio", y="demanda")
g.plot_joint(sns.scatterplot, hue=df["region"])
g.plot_marginals(sns.histplot)
```

## Cuándo usar cada uno

| Pregunta | Herramienta |
|---|---|
| "¿Cómo se relaciona CADA PAR de muchas variables entre sí?" | `pairplot()` / `PairGrid` |
| "¿Cómo se relacionan EXACTAMENTE DOS variables, con sus distribuciones marginales?" | `jointplot()` / `JointGrid` |
| Necesito una función DISTINTA en cada zona de la matriz | `PairGrid`/`JointGrid` (bajo nivel) en vez de la versión simple |

## Ver también

- [[09 - FacetGrid - Small Multiples]]
- [[03 - Gráficos de Relación]]
- [[04 - Gráficos de Distribución]]
- [[08 - Mapas de Calor y Matrices]]
