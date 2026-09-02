---
tags: [seaborn, python, data-science, visualization, colors, cheat-sheet]
---

# 11 — Paletas de Color

> Continúa de [[10 - Gráficos Multivariados]].

## Los tres tipos de paleta, y cuándo usar cada uno

Ver la misma clasificación conceptual en [[Python/Matplotlib/07 - Personalización de Líneas, Marcadores y Colores#Colormaps — paletas continuas para datos numéricos|Python/Matplotlib]] — Seaborn añade funciones dedicadas para generar y previsualizar cada tipo.

```python
sns.color_palette("viridis")            # secuencial — para magnitudes de un solo sentido (bajo a alto)
sns.color_palette("coolwarm")             # divergente — para datos con un punto medio significativo (ej. 0)
sns.color_palette("tab10")                  # cualitativa — para categorías SIN orden inherente
```

## Previsualizar una paleta

```python
sns.color_palette("viridis", n_colors=8)              # devuelve una lista de 8 tuplas RGB
sns.palplot(sns.color_palette("viridis", 8))            # dibuja una franja de colores para inspección visual rápida
```

## Aplicar una paleta a un gráfico

```python
sns.scatterplot(data=df, x="precio", y="demanda", hue="region", palette="Set2")
sns.barplot(data=df, x="region", y="ventas", palette="viridis")
sns.heatmap(matriz_corr, cmap="coolwarm")     # en heatmap/kdeplot 2D el argumento se llama 'cmap', no 'palette'
```

**Nota de nomenclatura:** funciones categóricas/de relación usan el argumento `palette=`; funciones que colorean una superficie continua (`heatmap`, `kdeplot` bivariado) usan `cmap=` — ambos aceptan los mismos nombres de colormap de Matplotlib/Seaborn, solo cambia el nombre del parámetro según el tipo de función.

## Paletas cualitativas por defecto

```python
sns.color_palette()                    # la paleta default de Seaborn (10 colores, distinguibles y estéticamente cuidados)
sns.set_palette("Set2")                  # cambia la paleta default para TODOS los gráficos siguientes en la sesión
```

## `cubehelix_palette()` — secuencial que también funciona en escala de grises

```python
sns.color_palette(sns.cubehelix_palette(as_cmap=True))
sns.kdeplot(data=df, x="precio", y="demanda", cmap=sns.cubehelix_palette(as_cmap=True), fill=True)
```

Diseñada para mantener un orden de luminosidad monótono incluso al imprimir en blanco y negro o para lectores con daltonismo — una alternativa a `viridis` con más control sobre el tono/saturación inicial.

## Paletas divergentes centradas manualmente

```python
sns.diverging_palette(220, 20, as_cmap=True)     # tonos personalizados (matiz inicial, matiz final) centrados en un punto neutro
sns.heatmap(matriz_corr, cmap=sns.diverging_palette(220, 20, as_cmap=True), center=0)
```

`center=0` en `heatmap()` asegura que el color neutro de la paleta divergente caiga exactamente en el valor 0 — sin esto, si los datos no están perfectamente balanceados alrededor de 0, el punto "neutro" visual no coincidiría con el punto numéricamente neutro.

## Paletas accesibles (daltonismo)

```python
sns.color_palette("colorblind")     # paleta cualitativa diseñada para ser distinguible con las formas más comunes de daltonismo
```

Ver la discusión completa de accesibilidad de color en [[Python/Matplotlib/07 - Personalización de Líneas, Marcadores y Colores#Colores accesibles para daltonismo|Python/Matplotlib]] — la paleta `"colorblind"` de Seaborn es la misma lista de colores usada como ejemplo ahí.

## Ver también

- [[10 - Gráficos Multivariados]]
- [[12 - Temas y Estética]]
- [[Python/Matplotlib/07 - Personalización de Líneas, Marcadores y Colores|Python/Matplotlib]]
