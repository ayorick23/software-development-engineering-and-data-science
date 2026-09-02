---
tags: [matplotlib, python, data-science, visualization, subplots, cheat-sheet]
---

# 03 — Creación de Figuras y Subplots

> Continúa de [[02 - Anatomía de una Figura]].

## `plt.subplots()` — la forma estándar de empezar

```python
fig, ax = plt.subplots()                        # una sola figura con un solo Axes
fig, ax = plt.subplots(figsize=(10, 6))            # tamaño en pulgadas (ancho, alto)
fig, ax = plt.subplots(dpi=100)                      # resolución — relevante al exportar, ver 14
```

## Múltiples subplots en una cuadrícula

```python
fig, axes = plt.subplots(2, 3)                    # cuadrícula 2 filas x 3 columnas -> axes es un array 2D de Axes
axes[0, 0].plot([1, 2, 3])                          # acceder a un subplot específico por posición [fila, columna]
axes[1, 2].bar(["a", "b"], [3, 5])

fig, axes = plt.subplots(1, 3)                      # con una sola fila, axes es un array 1D
axes[0].plot(...)

fig, axes = plt.subplots(2, 2, sharex=True, sharey=True)   # ejes compartidos entre subplots — útil para comparar escalas
```

## Iterar sobre subplots dinámicamente

```python
fig, axes = plt.subplots(2, 2, figsize=(10, 8))
datos = [serie_a, serie_b, serie_c, serie_d]

for ax, serie, titulo in zip(axes.flat, datos, ["A", "B", "C", "D"]):
    ax.plot(serie)
    ax.set_title(titulo)

fig.tight_layout()     # ajusta automáticamente el espaciado para que los títulos/labels no se superpongan
```

`axes.flat` aplana el array 2D de `Axes` a un iterador 1D — el patrón estándar para recorrer todos los subplots con un solo loop sin anidar dos `for`.

## `add_subplot()` — control más granular

```python
fig = plt.figure(figsize=(10, 6))
ax1 = fig.add_subplot(2, 2, 1)     # (filas, columnas, posición) — posición 1-indexed, izquierda a derecha, arriba a abajo
ax2 = fig.add_subplot(2, 2, 2)
ax3 = fig.add_subplot(2, 1, 2)       # una cuadrícula DISTINTA en la misma figura — subplot que ocupa toda la fila inferior
```

`add_subplot()` permite combinar cuadrículas de tamaños distintos dentro de una misma figura (por ejemplo, dos gráficos pequeños arriba y uno grande abajo) — algo que `plt.subplots()` con una sola cuadrícula uniforme no puede hacer directamente.

## `GridSpec` — layouts irregulares y con distinto tamaño relativo

```python
from matplotlib.gridspec import GridSpec

fig = plt.figure(figsize=(10, 8))
gs = GridSpec(3, 3, figure=fig, height_ratios=[1, 2, 1], width_ratios=[2, 1, 1])

ax1 = fig.add_subplot(gs[0, :])         # toda la primera fila
ax2 = fig.add_subplot(gs[1, 0])          # fila 1, columna 0
ax3 = fig.add_subplot(gs[1:, 1:])          # bloque que ocupa filas 1-2, columnas 1-2
```

`GridSpec` es la herramienta para layouts complejos con paneles de tamaños desiguales — `height_ratios`/`width_ratios` controlan la proporción relativa de cada fila/columna.

## `subplot_mosaic()` — layouts nombrados con ASCII art

```python
fig, axes = plt.subplot_mosaic("""
    AAB
    CCB
    CCD
""")

axes["A"].plot([1, 2, 3])
axes["B"].bar(["x", "y"], [3, 5])
axes["D"].scatter([1, 2], [3, 4])
```

`subplot_mosaic()` (Matplotlib 3.3+) describe el layout con un string tipo ASCII art donde cada letra es un panel — cada letra repetida se fusiona en un solo `Axes` que ocupa esas celdas. Es, en la práctica, más legible que `GridSpec` para layouts asimétricos y devuelve los `Axes` en un diccionario indexado por nombre en vez de por posición.

## Espaciado entre subplots

```python
fig.tight_layout()                                     # ajuste automático — la opción más simple, cubre la mayoría de casos
fig.subplots_adjust(hspace=0.4, wspace=0.3)              # control manual del espacio vertical/horizontal entre subplots
fig, axes = plt.subplots(2, 2, layout="constrained")      # 'constrained layout' — alternativa moderna, más robusta que tight_layout con colorbars/leyendas externas
```

## Ver también

- [[Python/Matplotlib/01 - Introducción y Arquitectura]]
- [[02 - Anatomía de una Figura]]
- [[14 - Guardado y Exportación]]
