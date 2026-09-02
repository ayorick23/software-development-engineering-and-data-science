---
tags: [matplotlib, python, data-science, visualization, colors, cheat-sheet]
---

# 07 — Personalización de Líneas, Marcadores y Colores

> Continúa de [[06 - Gráficos Estadísticos]].

## Especificar colores

```python
ax.plot(x, y, color="red")            # nombre de color
ax.plot(x, y, color="#1f77b4")          # hexadecimal
ax.plot(x, y, color=(0.2, 0.4, 0.6))      # tupla RGB (valores 0-1)
ax.plot(x, y, color=(0.2, 0.4, 0.6, 0.5))   # RGBA, con transparencia (alpha) incluida
ax.plot(x, y, alpha=0.5)                     # transparencia por separado
```

## Estilos de línea

```python
ax.plot(x, y, linestyle="-")      # sólida (default)
ax.plot(x, y, linestyle="--")      # discontinua
ax.plot(x, y, linestyle="-.")       # guion-punto
ax.plot(x, y, linestyle=":")         # punteada
ax.plot(x, y, linewidth=2.5)          # grosor de línea
```

## Marcadores

```python
ax.plot(x, y, marker="o")           # círculo
ax.plot(x, y, marker="s")            # cuadrado
ax.plot(x, y, marker="^")             # triángulo
ax.plot(x, y, marker="*")              # estrella
ax.plot(x, y, marker="o", markersize=10, markerfacecolor="white", markeredgecolor="blue", markeredgewidth=2)
```

| Código | Marcador | Código | Marcador |
|---|---|---|---|
| `o` | Círculo | `s` | Cuadrado |
| `^`, `v` | Triángulo arriba/abajo | `D` | Diamante |
| `*` | Estrella | `x`, `+` | X, cruz |
| `.` | Punto pequeño | `None` | Sin marcador |

## Colormaps — paletas continuas para datos numéricos

```python
ax.scatter(x, y, c=valores, cmap="viridis")     # viridis: perceptualmente uniforme, recomendado por default
ax.scatter(x, y, c=valores, cmap="coolwarm")      # divergente — bueno para datos con un punto medio significativo (ej. correlaciones -1 a 1)
ax.scatter(x, y, c=valores, cmap="Reds")            # secuencial — bueno para magnitudes de un solo signo (0 a máximo)
```

| Tipo de colormap | Cuándo usarlo | Ejemplos |
|---|---|---|
| Secuencial | Datos que van de bajo a alto, un solo sentido | `viridis`, `Blues`, `Reds` |
| Divergente | Datos con un punto medio significativo (ej. 0, o un promedio) | `coolwarm`, `RdBu`, `seismic` |
| Cualitativo | Categorías SIN orden inherente | `tab10`, `Set2`, `Pastel1` |

**Advertencia importante:** `jet` (el colormap "arcoíris" clásico) **no es perceptualmente uniforme** — introduce bandas de contraste falso que no reflejan la magnitud real de los datos. `viridis` (default desde Matplotlib 2.0) fue diseñado específicamente para ser uniforme y legible incluso en escala de grises o para personas con daltonismo — usarlo como default salvo razón específica para otro.

## Ciclos de color automáticos para múltiples series

```python
plt.rcParams["axes.prop_cycle"] = plt.cycler(color=["#1f77b4", "#ff7f0e", "#2ca02c"])

fig, ax = plt.subplots()
for serie in lista_de_series:
    ax.plot(serie)     # cada llamada usa el SIGUIENTE color del ciclo automáticamente, sin especificarlo
```

Ver cómo cambiar el ciclo de colores globalmente (o por estilo completo) en [[10 - Estilos y rcParams]].

## Colores accesibles para daltonismo

```python
colores_seguros = ["#0173B2", "#DE8F05", "#029E73", "#D55E00", "#CC78BC"]   # paleta "colorblind" de Seaborn, funciona igual en Matplotlib
```

Evitar depender únicamente de rojo/verde para distinguir categorías (la combinación más común de daltonismo) — combinar color con marcador o linestyle distinto asegura que el gráfico siga siendo legible incluso sin distinguir el color.

## Ver también

- [[06 - Gráficos Estadísticos]]
- [[08 - Ejes, Escalas y Ticks]]
- [[10 - Estilos y rcParams]]
- [[11 - Mapas de Calor, Imágenes y Colorbars]]
