---
tags: [seaborn, python, data-science, visualization, categorical, cheat-sheet]
---

# 05 — Gráficos Categóricos I: Dispersión y Conteo

> Continúa de [[04 - Gráficos de Distribución]].

## `stripplot()` — cada observación como un punto individual

```python
sns.stripplot(data=df, x="region", y="ventas")
sns.stripplot(data=df, x="region", y="ventas", hue="canal")
sns.stripplot(data=df, x="region", y="ventas", jitter=True)     # dispersa horizontalmente los puntos para evitar superposición total
```

`jitter=True` (default) agrega un desplazamiento horizontal aleatorio pequeño a cada punto — sin él, todos los puntos con el mismo valor de `y` quedarían apilados exactamente en la misma línea vertical, ocultando cuántas observaciones realmente hay.

## `swarmplot()` — como stripplot, pero sin superposición

```python
sns.swarmplot(data=df, x="region", y="ventas")
sns.swarmplot(data=df, x="region", y="ventas", hue="canal")
```

`swarmplot()` calcula posiciones para que **ningún punto se superponga** con otro (a diferencia del jitter aleatorio de `stripplot`), formando una silueta que también comunica la densidad de la distribución — el trade-off es que se vuelve lento y visualmente saturado con datasets de más de unos pocos miles de puntos por categoría.

| | `stripplot()` | `swarmplot()` |
|---|---|---|
| Posición horizontal | Aleatoria (jitter) | Calculada para no superponerse |
| Rendimiento con muchos puntos | Rápido | Lento (>unos miles de puntos) |
| Comunica densidad | Aproximadamente (por superposición visual) | Con precisión (silueta tipo violín) |

## Combinar con un gráfico de resumen estadístico

```python
sns.boxplot(data=df, x="region", y="ventas", showfliers=False, color="lightgray")
sns.stripplot(data=df, x="region", y="ventas", color="black", alpha=0.5, size=3)
```

Superponer un `stripplot`/`swarmplot` sobre un `boxplot`/`violinplot` (ver [[06 - Gráficos Categóricos II - Estimadores]]) es un patrón muy común en análisis exploratorio: el resumen estadístico da la forma general, los puntos individuales muestran el tamaño real de la muestra y posibles outliers específicos que un boxplot solo señala como puntos anónimos.

## `countplot()` — conteo de frecuencias por categoría

```python
sns.countplot(data=df, x="region")                          # cuenta cuántas filas hay por cada valor de 'region'
sns.countplot(data=df, x="region", hue="canal")                # conteo cruzado por dos categorías
sns.countplot(data=df, y="region")                                # orientación horizontal — mejor con etiquetas largas
sns.countplot(data=df, x="region", order=df["region"].value_counts().index)   # ordenado de mayor a menor frecuencia
```

`countplot()` es el equivalente categórico de un histograma: mientras `histplot()` cuenta observaciones dentro de rangos numéricos (bins), `countplot()` cuenta observaciones por cada valor discreto de una variable categórica — no requiere una columna `y` numérica porque la altura de la barra **es** el conteo.

## `catplot()` — la versión figure-level para todo lo categórico

```python
sns.catplot(data=df, x="region", y="ventas", kind="strip", col="canal")
sns.catplot(data=df, x="region", y="ventas", kind="swarm", hue="canal")
sns.catplot(data=df, x="region", kind="count", col="canal")
```

`catplot(kind=...)` unifica el acceso figure-level a **todos** los gráficos categóricos de Seaborn (los de este archivo y los de [[06 - Gráficos Categóricos II - Estimadores]]) — el argumento `kind` selecciona cuál función axes-level se usa internamente (`"strip"`, `"swarm"`, `"box"`, `"violin"`, `"bar"`, `"point"`, `"count"`, `"boxen"`).

## Ver también

- [[04 - Gráficos de Distribución]]
- [[06 - Gráficos Categóricos II - Estimadores]]
- [[09 - FacetGrid - Small Multiples]]
