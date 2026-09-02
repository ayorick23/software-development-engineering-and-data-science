---
tags: [matplotlib, python, data-science, visualization, cheat-sheet]
---

# 08 — Ejes, Escalas y Ticks

> Continúa de [[07 - Personalización de Líneas, Marcadores y Colores]].

## Límites de los ejes

```python
ax.set_xlim(0, 100)
ax.set_ylim(-10, 10)
ax.set_xlim(left=0)              # solo fija el mínimo, deja el máximo automático
ax.autoscale(enable=True, axis="y")   # vuelve a dejar que Matplotlib calcule los límites automáticamente
```

## Escalas: lineal, logarítmica y simétrica-log

```python
ax.set_yscale("linear")     # default
ax.set_yscale("log")          # escala logarítmica — imprescindible cuando los datos abarcan varios órdenes de magnitud
ax.set_yscale("symlog")         # logarítmica que también maneja valores negativos y el cero (log puro no puede)
```

**Cuándo usar escala log:** cuando los datos abarcan varios órdenes de magnitud (ej. población de países, ingresos, frecuencias) — en escala lineal, los valores pequeños se aplastan visualmente contra el eje y se vuelven ilegibles.

## `Locators` — dónde aparecen los ticks

```python
from matplotlib.ticker import MultipleLocator, MaxNLocator, LogLocator

ax.xaxis.set_major_locator(MultipleLocator(5))       # un tick cada 5 unidades, exactamente
ax.xaxis.set_major_locator(MaxNLocator(nbins=10))       # máximo ~10 ticks, posiciones "bonitas" elegidas automáticamente
ax.yaxis.set_minor_locator(MultipleLocator(1))            # ticks menores (sin etiqueta) cada 1 unidad
```

## `Formatters` — cómo se muestran las etiquetas de los ticks

```python
from matplotlib.ticker import FuncFormatter, PercentFormatter

ax.yaxis.set_major_formatter(FuncFormatter(lambda x, pos: f"${x:,.0f}"))    # formato de moneda
ax.yaxis.set_major_formatter(PercentFormatter(xmax=1.0))                      # convierte 0.45 -> "45%"

import matplotlib.dates as mdates
ax.xaxis.set_major_formatter(mdates.DateFormatter("%b %Y"))                    # fechas -> "Mar 2026"
```

`FuncFormatter` acepta cualquier función `(valor, posición) -> string` — es la forma más flexible de controlar exactamente cómo se ve cada etiqueta numérica, sin tener que reformatear los datos originales.

## Ejes gemelos (`twinx`/`twiny`) — dos escalas en el mismo gráfico

```python
fig, ax1 = plt.subplots()

ax1.plot(x, temperatura, color="tab:red")
ax1.set_ylabel("Temperatura (°C)", color="tab:red")
ax1.tick_params(axis="y", labelcolor="tab:red")

ax2 = ax1.twinx()                              # comparte el eje X, tiene su PROPIO eje Y independiente
ax2.plot(x, precipitacion, color="tab:blue")
ax2.set_ylabel("Precipitación (mm)", color="tab:blue")
ax2.tick_params(axis="y", labelcolor="tab:blue")
```

`twinx()` es la herramienta estándar para comparar dos series con **escalas de magnitud muy distintas** en el mismo gráfico (ej. temperatura vs precipitación) — cada serie tiene su propio eje Y, coloreado a juego para que quede claro cuál corresponde a cuál.

## Fechas en el eje X

```python
import matplotlib.dates as mdates

ax.plot(fechas, valores)
ax.xaxis.set_major_locator(mdates.MonthLocator())               # un tick por mes
ax.xaxis.set_major_formatter(mdates.DateFormatter("%b %Y"))       # formato "Mar 2026"
fig.autofmt_xdate(rotation=45)                                      # rota las etiquetas automáticamente para que no se solapen
```

## Relación de aspecto

```python
ax.set_aspect("equal")            # 1 unidad en X == 1 unidad en Y visualmente — esencial para no distorsionar geometría (ej. círculos)
ax.set_aspect("auto")               # default — Matplotlib ajusta libremente para llenar el espacio disponible
```

## Ver también

- [[07 - Personalización de Líneas, Marcadores y Colores]]
- [[02 - Anatomía de una Figura]]
- [[09 - Texto, Anotaciones y Leyendas]]
