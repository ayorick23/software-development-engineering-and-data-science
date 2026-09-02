---
tags: [matplotlib, python, data-science, visualization, cheat-sheet]
---

# 05 — Gráficos de Barras e Histogramas

> Continúa de [[04 - Gráficos de Líneas y Dispersión]].

## `bar()` — barras verticales

```python
categorias = ["A", "B", "C", "D"]
valores = [23, 45, 12, 38]

ax.bar(categorias, valores)
ax.bar(categorias, valores, color="steelblue", edgecolor="black", width=0.6)
```

## `barh()` — barras horizontales

```python
ax.barh(categorias, valores)     # útil cuando las etiquetas de categoría son largas (no se solapan como en bar() vertical)
```

## Barras agrupadas (múltiples series por categoría)

```python
x = np.arange(len(categorias))
ancho = 0.35

ax.bar(x - ancho/2, valores_2025, ancho, label="2025")
ax.bar(x + ancho/2, valores_2026, ancho, label="2026")
ax.set_xticks(x)
ax.set_xticklabels(categorias)
ax.legend()
```

El patrón de desplazar cada serie por una fracción del ancho de barra (`x - ancho/2`, `x + ancho/2`) es la forma estándar de crear barras agrupadas lado a lado — Matplotlib no tiene un argumento nativo `hue=` como Seaborn para esto.

## Barras apiladas

```python
ax.bar(categorias, valores_2025, label="2025")
ax.bar(categorias, valores_2026, bottom=valores_2025, label="2026")   # 'bottom' apila sobre la serie anterior
ax.legend()
```

## `hist()` — histogramas

```python
datos = np.random.normal(0, 1, 1000)

ax.hist(datos, bins=30)                            # 30 intervalos (bins)
ax.hist(datos, bins=30, density=True, alpha=0.6)      # normalizado a densidad de probabilidad (área total = 1)
ax.hist([datos_a, datos_b], bins=20, label=["A", "B"])   # dos histogramas superpuestos, con transparencia si se agrega alpha
```

**El parámetro `bins` importa más de lo que parece:** muy pocos bins ocultan la forma real de la distribución; demasiados introducen ruido visual — `bins="auto"` deja que NumPy/Matplotlib elijan un número razonable según los datos.

## Histograma 2D (densidad conjunta de dos variables)

```python
x = np.random.normal(0, 1, 5000)
y = np.random.normal(0, 1, 5000)

ax.hist2d(x, y, bins=30, cmap="Blues")
fig.colorbar(ax.collections[0], ax=ax)
```

Ver también el detalle de `imshow`/`pcolormesh` para otras formas de representar datos en cuadrícula 2D en [[11 - Mapas de Calor, Imágenes y Colorbars]].

## `pie()` — gráfico circular

```python
ax.pie(valores, labels=categorias, autopct="%1.1f%%", startangle=90)
```

`autopct="%1.1f%%"` muestra el porcentaje de cada porción con un decimal. Los gráficos de pastel son controvertidos para comparar más de 3-4 categorías (el ojo humano compara ángulos peor que longitudes) — un `bar()` horizontal suele comunicar la misma información con más claridad; ver la discusión en [[17 - Buenas Prácticas, Errores Comunes y Comparativa]].

## Ver también

- [[04 - Gráficos de Líneas y Dispersión]]
- [[06 - Gráficos Estadísticos]]
- [[11 - Mapas de Calor, Imágenes y Colorbars]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
