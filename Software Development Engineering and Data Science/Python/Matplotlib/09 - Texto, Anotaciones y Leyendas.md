---
tags: [matplotlib, python, data-science, visualization, cheat-sheet]
---

# 09 — Texto, Anotaciones y Leyendas

> Continúa de [[08 - Ejes, Escalas y Ticks]].

## Texto libre en coordenadas de datos

```python
ax.text(x=5, y=10, s="Punto importante", fontsize=12, color="red", ha="center", va="bottom")
```

`ha`/`va` (*horizontal/vertical alignment*) controlan qué punto exacto del texto queda anclado a la coordenada `(x, y)` dada — `ha="center"` centra el texto horizontalmente sobre ese punto en vez de que el punto sea la esquina izquierda del texto.

## `annotate()` — texto CON una flecha señalando un punto

```python
ax.annotate(
    "Máximo local",
    xy=(punto_x, punto_y),                    # el punto que se señala
    xytext=(punto_x + 2, punto_y + 5),          # dónde va el TEXTO (puede estar lejos del punto)
    arrowprops=dict(facecolor="black", arrowstyle="->"),
)
```

`annotate()` es la herramienta correcta cuando el texto debe estar **desplazado** del punto que describe (para no tapar los datos) pero necesita una flecha visual conectándolos — `text()` no tiene esa capacidad.

## Leyendas: posicionamiento y personalización

```python
ax.legend()                                  # posición automática ("best"), evita superponerse con los datos si es posible
ax.legend(loc="upper right")                   # posición explícita
ax.legend(loc="center left", bbox_to_anchor=(1, 0.5))   # FUERA del área de gráfico, a la derecha
ax.legend(ncol=2)                                # organizar en múltiples columnas
ax.legend(title="Categoría", fontsize=9, frameon=False)   # con título propio, sin marco
```

`bbox_to_anchor` combinado con `loc` es la forma de colocar la leyenda **fuera** del área de datos — muy usado cuando hay muchas series y la leyenda taparía parte del gráfico.

## Leyenda manual (cuando las series no tienen `label=` directo)

```python
from matplotlib.lines import Line2D

elementos_leyenda = [
    Line2D([0], [0], color="red", lw=2, label="Categoría A"),
    Line2D([0], [0], color="blue", lw=2, label="Categoría B"),
]
ax.legend(handles=elementos_leyenda)
```

Útil cuando la leyenda debe describir algo que no corresponde 1:1 con una llamada a `plot()`/`scatter()` — por ejemplo, un color que representa un grupo pintado a través de múltiples llamadas separadas.

## Títulos con formato matemático (LaTeX)

```python
ax.set_title(r"Distribución Normal: $f(x) = \frac{1}{\sigma\sqrt{2\pi}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}$")
ax.set_xlabel(r"$x$")
```

Matplotlib interpreta cualquier texto entre `$...$` como notación matemática LaTeX-like nativa (sin necesitar una instalación completa de LaTeX) — el prefijo `r"..."` (raw string) evita que Python interprete las barras invertidas (`\sigma`, `\frac`) como escapes.

## Recuadros de texto con fondo (`bbox`)

```python
ax.text(0.5, 0.9, "Nota importante", transform=ax.transAxes,
         bbox=dict(boxstyle="round", facecolor="wheat", alpha=0.5))
```

`transform=ax.transAxes` cambia el sistema de coordenadas de "unidades de datos" a "fracción del área del Axes" (0 a 1 en ambos ejes) — útil para posicionar texto de forma consistente sin importar el rango real de los datos graficados.

## Ver también

- [[08 - Ejes, Escalas y Ticks]]
- [[02 - Anatomía de una Figura]]
- [[10 - Estilos y rcParams]]
