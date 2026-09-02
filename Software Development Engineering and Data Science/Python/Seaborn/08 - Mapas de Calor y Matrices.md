---
tags: [seaborn, python, data-science, visualization, heatmap, cheat-sheet]
---

# 08 — Mapas de Calor y Matrices

> Continúa de [[07 - Gráficos de Regresión]]. La familia de funciones diseñada para trabajar con datos en formato **wide** (matriz), a diferencia del resto de Seaborn (ver [[02 - Formato de Datos - Long vs Wide]]).

## `heatmap()` — visualizar una matriz completa

```python
matriz_corr = df.select_dtypes(include="number").corr()

sns.heatmap(matriz_corr)
sns.heatmap(matriz_corr, annot=True, fmt=".2f")                # muestra el valor numérico dentro de cada celda
sns.heatmap(matriz_corr, cmap="coolwarm", vmin=-1, vmax=1)        # colormap divergente centrado en 0, típico para correlaciones
sns.heatmap(matriz_corr, mask=np.triu(np.ones_like(matriz_corr, dtype=bool)))   # oculta el triángulo superior (redundante en una matriz de correlación simétrica)
```

`annot=True` es la razón principal por la que `sns.heatmap()` es preferible sobre `ax.imshow()` de Matplotlib puro (ver [[Python/Matplotlib/11 - Mapas de Calor, Imágenes y Colorbars|Python/Matplotlib]]) para matrices de correlación: escribir el valor exacto dentro de cada celda con una sola línea, sin loops manuales de `ax.text()`.

## El patrón completo: matriz de correlación triangular

```python
matriz_corr = df.select_dtypes(include="number").corr()
mascara = np.triu(np.ones_like(matriz_corr, dtype=bool))    # True en el triángulo superior (incluyendo la diagonal)

fig, ax = plt.subplots(figsize=(10, 8))
sns.heatmap(
    matriz_corr,
    mask=mascara,
    annot=True,
    fmt=".2f",
    cmap="coolwarm",
    vmin=-1, vmax=1,
    square=True,
    ax=ax,
)
```

Ocultar el triángulo superior evita la redundancia visual de una matriz de correlación simétrica (donde la correlación de A con B es idéntica a la de B con A) — es prácticamente el patrón estándar en cualquier análisis exploratorio de datos que incluya una matriz de correlación.

## `clustermap()` — heatmap con reordenamiento jerárquico automático

```python
sns.clustermap(matriz_corr, cmap="coolwarm", vmin=-1, vmax=1, annot=True, fmt=".2f")
```

A diferencia de `heatmap()` (que respeta el orden original de filas/columnas), `clustermap()` aplica **clustering jerárquico** automáticamente y reordena filas y columnas para agrupar elementos similares juntos, dibujando además los dendrogramas del agrupamiento a los lados. Es especialmente útil para descubrir **grupos de variables correlacionadas** que no eran evidentes en el orden original de las columnas.

```python
sns.clustermap(matriz_corr, method="average", metric="euclidean")   # método de linkage y métrica de distancia del clustering
sns.clustermap(matriz_corr, row_cluster=False)                        # clusteriza solo columnas, deja filas en su orden original
```

## Cuándo usar cada uno

| | `heatmap()` | `clustermap()` |
|---|---|---|
| Orden de filas/columnas | El original del DataFrame | Reordenado por similitud (clustering jerárquico) |
| Objeto devuelto | `Axes` (axes-level, acepta `ax=`) | `ClusterGrid` propio (figure-level, NO acepta `ax=`) |
| Uso típico | Matriz de correlación con orden significativo ya conocido | Descubrir agrupaciones/patrones ocultos en la matriz |

## Ver también

- [[07 - Gráficos de Regresión]]
- [[10 - Gráficos Multivariados]]
- [[Python/Matplotlib/11 - Mapas de Calor, Imágenes y Colorbars|Python/Matplotlib]]
- [[Python/NumPy/11 - Álgebra Lineal (np.linalg)|Python/NumPy]]
