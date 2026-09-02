---
tags: [seaborn, python, data-science, visualization, distribution, cheat-sheet]
---

# 04 — Gráficos de Distribución

> Continúa de [[03 - Gráficos de Relación]].

## `histplot()` — histogramas con más control que Matplotlib

```python
sns.histplot(data=df, x="edad")
sns.histplot(data=df, x="edad", bins=30)
sns.histplot(data=df, x="edad", hue="genero")                          # histogramas superpuestos por categoría
sns.histplot(data=df, x="edad", hue="genero", multiple="stack")           # apilados en vez de superpuestos
sns.histplot(data=df, x="edad", hue="genero", multiple="dodge")             # lado a lado
sns.histplot(data=df, x="edad", kde=True)                                     # histograma + curva de densidad superpuesta
sns.histplot(data=df, x="edad", stat="density")                                 # normalizado a densidad (área = 1) en vez de conteo
```

`multiple="stack"`/`"dodge"` resuelven un problema real de comparar distribuciones por grupo: histogramas superpuestos con transparencia (`multiple="layer"`, el default) se vuelven ilegibles con 3+ categorías — apilar o poner lado a lado suele comunicar mejor.

## `kdeplot()` — estimación de densidad por kernel

```python
sns.kdeplot(data=df, x="edad")
sns.kdeplot(data=df, x="edad", hue="genero")
sns.kdeplot(data=df, x="edad", hue="genero", fill=True, alpha=0.3)     # área rellena, más legible que solo la línea
sns.kdeplot(data=df, x="edad", bw_adjust=0.5)                            # curva más "ajustada" a los datos (menos suavizado)
sns.kdeplot(data=df, x="edad", bw_adjust=2)                                # más suavizado, oculta detalles finos
```

`bw_adjust` (bandwidth) controla el trade-off suavizado-vs-detalle de la curva de densidad — un valor muy bajo sobreajusta a ruido específico de la muestra; uno muy alto oculta multimodalidad real. Es conceptualmente equivalente al parámetro `bins` en un histograma.

### KDE bivariado — densidad conjunta de dos variables

```python
sns.kdeplot(data=df, x="precio", y="demanda")                    # curvas de nivel de densidad 2D
sns.kdeplot(data=df, x="precio", y="demanda", fill=True, cmap="viridis")
```

## `ecdfplot()` — función de distribución acumulada empírica

```python
sns.ecdfplot(data=df, x="edad")
sns.ecdfplot(data=df, x="edad", hue="genero")
```

A diferencia de un histograma (que depende de la elección de `bins`), una ECDF no tiene ningún parámetro de suavizado que ajustar — cada punto de datos contribuye exactamente un escalón, mostrando sin ambigüedad qué fracción de las observaciones cae por debajo de cualquier valor dado. Es preferible al histograma cuando se necesita leer percentiles exactos directamente del gráfico.

## `rugplot()` — marcas individuales de cada observación

```python
sns.histplot(data=df, x="edad")
sns.rugplot(data=df, x="edad")     # superpone pequeñas marcas verticales, una por observación real, en el eje X
```

`rugplot()` casi nunca se usa solo — se combina con `histplot`/`kdeplot` para mostrar la ubicación exacta de cada observación individual además de la forma agregada de la distribución, útil para detectar huecos o concentraciones que el suavizado del KDE podría ocultar.

## `displot()` — la versión figure-level

```python
sns.displot(data=df, x="edad", kind="hist", col="genero")     # un panel de histograma por género
sns.displot(data=df, x="edad", kind="kde", hue="genero")
sns.displot(data=df, x="edad", kind="ecdf")
```

## Ver también

- [[03 - Gráficos de Relación]]
- [[05 - Gráficos Categóricos I - Dispersión y Conteo]]
- [[09 - FacetGrid - Small Multiples]]
- [[Clase 05 - Anomalías y Ruido|Ciclo 04 — Anomalías y Ruido]]
