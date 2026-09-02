---
tags: [seaborn, python, data-science, visualization, cheat-sheet]
---

# 03 — Gráficos de Relación

> Continúa de [[02 - Formato de Datos - Long vs Wide]].

## `scatterplot()` — dispersión con semántica estadística

```python
sns.scatterplot(data=df, x="precio", y="demanda")
sns.scatterplot(data=df, x="precio", y="demanda", hue="region")               # color por categoría
sns.scatterplot(data=df, x="precio", y="demanda", hue="region", size="stock")   # + tamaño por otra variable numérica
sns.scatterplot(data=df, x="precio", y="demanda", hue="region", style="canal")   # + forma de marcador por otra categoría
```

`hue`, `size` y `style` son las tres "semánticas" centrales de Seaborn — cada una mapea una columna del DataFrame a un aspecto visual distinto, y **se pueden combinar las tres a la vez** para codificar hasta 5 dimensiones en un solo scatter (x, y, color, tamaño, forma).

## `lineplot()` — líneas con agregación e intervalo de confianza automáticos

```python
sns.lineplot(data=df, x="fecha", y="ventas")
sns.lineplot(data=df, x="fecha", y="ventas", hue="region")
```

Si hay **múltiples observaciones** por cada valor de `x` (por ejemplo, varias tiendas de la misma región en la misma fecha), `lineplot()` automáticamente agrega la línea central como la **media** y dibuja una banda sombreada con el **intervalo de confianza (95% por default)** — sin necesitar `groupby()` manual:

```python
sns.lineplot(data=df, x="fecha", y="ventas", hue="region", errorbar=("ci", 95))   # explícito
sns.lineplot(data=df, x="fecha", y="ventas", errorbar="sd")                         # banda de desviación estándar en vez de IC
sns.lineplot(data=df, x="fecha", y="ventas", errorbar=None)                          # sin banda, solo la línea agregada
```

Ver el mecanismo completo de estimadores/intervalos automáticos en [[13 - Estadística Integrada]].

## `relplot()` — la versión figure-level, con facetado

```python
sns.relplot(data=df, x="precio", y="demanda", hue="region", kind="scatter")
sns.relplot(data=df, x="fecha", y="ventas", hue="region", kind="line", col="canal")   # un panel por canal, además del color por región
```

`relplot(kind="scatter")` es equivalente a `scatterplot()`; `relplot(kind="line")` es equivalente a `lineplot()` — la diferencia es que `relplot()` administra su propia `Figure` y soporta `col=`/`row=` para facetado nativo (ver [[01 - Introducción y Arquitectura#Dos niveles de funciones Figure-level vs Axes-level|Figure-level vs Axes-level]] y [[09 - FacetGrid - Small Multiples]]).

## Personalizar paletas y tamaños en gráficos de relación

```python
sns.scatterplot(data=df, x="precio", y="demanda", hue="region", palette="viridis")
sns.scatterplot(data=df, x="precio", y="demanda", size="stock", sizes=(20, 200))   # rango explícito de tamaño de punto en píxeles
```

`sizes=(min, max)` controla el rango real en píxeles al que se mapea la variable de `size` — sin especificarlo, Seaborn elige un rango razonable automáticamente, pero puede no ser suficientemente contrastante para el caso de uso.

## Combinar con el `Axes` de Matplotlib para personalización fina

```python
fig, ax = plt.subplots(figsize=(8, 5))
sns.scatterplot(data=df, x="precio", y="demanda", hue="region", ax=ax)
ax.set_title("Relación Precio-Demanda por Región")     # cualquier método de Matplotlib sigue funcionando
ax.set_xlabel("Precio (USD)")
```

Ver el patrón completo de combinar Seaborn con personalización de Matplotlib en [[15 - Integración con Matplotlib y Pandas]].

## Ver también

- [[02 - Formato de Datos - Long vs Wide]]
- [[04 - Gráficos de Distribución]]
- [[09 - FacetGrid - Small Multiples]]
- [[13 - Estadística Integrada]]
