---
tags: [matplotlib, python, data-science, visualization, heatmap, cheat-sheet]
---

# 11 — Mapas de Calor, Imágenes y Colorbars

> Continúa de [[10 - Estilos y rcParams]].

## `imshow()` — mostrar una matriz 2D como imagen/mapa de calor

```python
matriz = np.random.rand(10, 10)

im = ax.imshow(matriz, cmap="viridis")
fig.colorbar(im, ax=ax, label="Intensidad")
```

```python
im = ax.imshow(matriz, cmap="coolwarm", vmin=-1, vmax=1)     # fija explícitamente el rango de color — importante para comparar varios mapas de calor entre sí con la MISMA escala
im = ax.imshow(matriz, cmap="viridis", aspect="auto")          # 'auto' permite que las celdas no sean cuadradas, ajustándose al tamaño del Axes
im = ax.imshow(matriz, interpolation="nearest")                  # sin suavizado entre píxeles — importante al mostrar imágenes reales, no matrices de datos abstractas
```

`vmin`/`vmax` explícitos son necesarios cuando se comparan varios `imshow()` en subplots distintos: sin ellos, cada uno normaliza su propia escala de color de forma independiente, haciendo que colores iguales en dos gráficos distintos representen valores numéricos distintos.

## `pcolormesh()` — cuadrícula con coordenadas X/Y reales

```python
x = np.linspace(0, 10, 50)
y = np.linspace(0, 5, 30)
X, Y = np.meshgrid(x, y)
Z = np.sin(X) * np.cos(Y)

malla = ax.pcolormesh(X, Y, Z, cmap="RdBu", shading="auto")
fig.colorbar(malla, ax=ax)
```

A diferencia de `imshow()` (que asume una cuadrícula de píxeles uniforme indexada 0,1,2...), `pcolormesh()` acepta coordenadas X/Y reales y potencialmente irregulares — la elección correcta cuando los ejes representan magnitudes físicas reales, no solo índices de matriz.

## `contour()` y `contourf()` — líneas y regiones de nivel

```python
niveles = ax.contour(X, Y, Z, levels=10, colors="black")        # solo LÍNEAS de contorno
ax.clabel(niveles, inline=True, fontsize=8)                        # etiqueta cada línea con su valor

relleno = ax.contourf(X, Y, Z, levels=20, cmap="viridis")           # REGIONES rellenas entre niveles
fig.colorbar(relleno, ax=ax)
```

`contourf()` (relleno) es visualmente más parecido a un mapa de calor continuo; `contour()` (solo líneas) es mejor para superponer sobre otro gráfico ya existente sin tapar los datos de fondo — combinarlos (`contourf` + `contour` con `clabel`) es un patrón común en visualización científica.

## Personalizar la `colorbar`

```python
cb = fig.colorbar(im, ax=ax)
cb.set_label("Temperatura (°C)")
cb.ax.tick_params(labelsize=9)

fig.colorbar(im, ax=ax, orientation="horizontal", shrink=0.6, pad=0.1)   # horizontal, más angosta, con espacio extra
```

## Mostrar imágenes reales (no matrices de datos abstractas)

```python
from matplotlib import image as mpimg

img = mpimg.imread("foto.png")       # devuelve un ndarray (alto, ancho, canales) — ver Python/NumPy
ax.imshow(img)
ax.axis("off")                          # oculta ejes/ticks — no tienen sentido sobre una foto real
```

Una imagen leída con `mpimg.imread()` es, por debajo, exactamente un `ndarray` de NumPy — ver [[Python/NumPy/01 - Introducción y Arquitectura Interna|Python/NumPy]] para la estructura subyacente (alto × ancho × canales RGB/RGBA).

## Ver también

- [[10 - Estilos y rcParams]]
- [[06 - Gráficos Estadísticos]]
- [[12 - Gráficos 3D]]
- [[Python/NumPy/01 - Introducción y Arquitectura Interna|Python/NumPy]]
