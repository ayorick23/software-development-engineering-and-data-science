---
tags: [matplotlib, python, data-science, visualization, statistics, cheat-sheet]
---

# 06 — Gráficos Estadísticos

> Continúa de [[05 - Gráficos de Barras e Histogramas]].

## `boxplot()` — diagrama de caja

```python
datos = [np.random.normal(0, std, 200) for std in [1, 2, 3]]

ax.boxplot(datos, tick_labels=["Grupo A", "Grupo B", "Grupo C"])
ax.boxplot(datos, vert=False)                # orientación horizontal
ax.boxplot(datos, showfliers=False)            # oculta los valores atípicos (outliers) individuales
```

Ver la lectura estadística completa de un boxplot (Q1, Q3, RIC, bigotes, outliers) en [[Clase 05 - Anomalías y Ruido|Ciclo 04 — Anomalías y Ruido]], que ya cubre la teoría de detección de atípicos — este archivo cubre solo la sintaxis de Matplotlib.

## `violinplot()` — distribución completa, no solo cuartiles

```python
ax.violinplot(datos, showmedians=True)
```

Un violin plot muestra la **densidad estimada completa** de la distribución (vía KDE) en vez de solo los 5 valores resumen de un boxplot — revela multimodalidad (dos "jorobas") que un boxplot esconde por completo. La versión de Seaborn (`sns.violinplot`) tiene mucha más personalización (agrupar por categoría con `hue=`, por ejemplo) — ver [[15 - Integración con NumPy, Pandas y Seaborn]].

## `errorbar()` — puntos/líneas con barras de error

```python
x = np.arange(5)
y = [10, 15, 13, 18, 16]
errores = [1, 2, 1.5, 1, 2.5]

ax.errorbar(x, y, yerr=errores, fmt="o-", capsize=5)          # barra de error vertical
ax.errorbar(x, y, xerr=0.2, yerr=errores, fmt="o", capsize=5)   # error en ambos ejes
```

`capsize` dibuja las "tapas" horizontales al final de cada barra de error — sin él, las barras son solo líneas verticales sin terminación visual clara.

## `pie()` con detalle de porcentaje — repaso rápido

Ver [[05 - Gráficos de Barras e Histogramas#pie() — gráfico circular|Gráficos de Barras e Histogramas]] para la sintaxis base.

## Matriz de correlación como mapa de calor

```python
import numpy as np
matriz_corr = np.corrcoef(datos_multivariados.T)

im = ax.imshow(matriz_corr, cmap="coolwarm", vmin=-1, vmax=1)
ax.set_xticks(range(len(nombres_variables)))
ax.set_xticklabels(nombres_variables, rotation=45, ha="right")
ax.set_yticks(range(len(nombres_variables)))
ax.set_yticklabels(nombres_variables)
fig.colorbar(im, ax=ax, label="Correlación")
```

Ver el catálogo completo de `imshow()` en [[11 - Mapas de Calor, Imágenes y Colorbars]] — la matriz de correlación es uno de sus usos más frecuentes en análisis exploratorio.

## Curvas de densidad (KDE) manual con SciPy

```python
from scipy.stats import gaussian_kde

datos = np.random.normal(0, 1, 500)
kde = gaussian_kde(datos)
x_rango = np.linspace(datos.min(), datos.max(), 200)

ax.plot(x_rango, kde(x_rango))
ax.hist(datos, bins=30, density=True, alpha=0.3)     # superpuesto para comparar histograma vs KDE
```

Matplotlib no tiene una función `kdeplot()` nativa (a diferencia de Seaborn) — se construye combinando `scipy.stats.gaussian_kde` con `ax.plot()`.

## Ver también

- [[05 - Gráficos de Barras e Histogramas]]
- [[11 - Mapas de Calor, Imágenes y Colorbars]]
- [[15 - Integración con NumPy, Pandas y Seaborn]]
- [[Clase 05 - Anomalías y Ruido|Ciclo 04 — Anomalías y Ruido]]
