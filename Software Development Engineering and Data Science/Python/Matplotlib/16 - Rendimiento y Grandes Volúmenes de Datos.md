---
tags: [matplotlib, python, data-science, visualization, performance, cheat-sheet]
---

# 16 — Rendimiento y Grandes Volúmenes de Datos

> Continúa de [[15 - Integración con NumPy, Pandas y Seaborn]].

## El backend importa para el rendimiento

```python
import matplotlib
matplotlib.use("Agg")     # backend no interactivo — más rápido para generar muchas imágenes en batch/servidor
```

Los backends interactivos (`Qt5Agg`, `TkAgg`) tienen overhead adicional por mantener una ventana renderizada en tiempo real — para generar cientos/miles de gráficos en un script batch (sin necesidad de verlos interactivamente), `Agg` es más rápido y no requiere entorno gráfico disponible.

## Reducir el número de puntos graficados

```python
# Con 1 millón de puntos, plot() se vuelve notablemente lento y el archivo exportado pesa mucho
ax.plot(x[::100], y[::100])          # graficar solo 1 de cada 100 puntos — downsampling simple

# Alternativa más informada: promediar en ventanas en vez de descartar puntos al azar
y_downsampled = y.reshape(-1, 100).mean(axis=1)
```

Para series de tiempo muy largas, graficar cada punto individual rara vez aporta información visual adicional (los píxeles de la pantalla ya no pueden distinguir esa densidad) — reducir el número de puntos (por muestreo o por promedio en ventanas) acelera tanto el renderizado como el guardado del archivo.

## `rasterized=True` — vectorial para el resto, raster solo para los datos pesados

```python
ax.scatter(x_millones_de_puntos, y_millones_de_puntos, rasterized=True)
fig.savefig("grafico.pdf")     # el scatter se rasteriza (píxeles), pero texto/ejes siguen siendo vectoriales nítidos
```

Al exportar a un formato vectorial (`.pdf`, `.svg`) un gráfico con muchísimos puntos/líneas, el archivo resultante puede volverse enorme y lento de abrir porque cada punto se guarda como objeto vectorial individual — `rasterized=True` convierte solo esa capa específica en píxeles, mientras el resto de la figura (texto, ejes, leyenda) permanece nítidamente vectorial.

## Blitting fuera de animaciones — actualizar sin redibujar todo

```python
fig, ax = plt.subplots()
linea, = ax.plot(x, y)
fig.canvas.draw()
fondo = fig.canvas.copy_from_bbox(ax.bbox)     # guarda el fondo estático una sola vez

for nuevo_y in secuencia_de_actualizaciones:
    fig.canvas.restore_region(fondo)             # restaura el fondo sin redibujar
    linea.set_ydata(nuevo_y)
    ax.draw_artist(linea)                          # redibuja SOLO la línea, no toda la figura
    fig.canvas.blit(ax.bbox)
```

Este patrón manual de blitting (la misma idea detrás de `FuncAnimation(blit=True)`, ver [[13 - Animaciones]]) es relevante para dashboards en vivo que actualizan un gráfico muchas veces por segundo — redibujar la figura completa en cada actualización es el cuello de botella típico de gráficos "en tiempo real" lentos.

## Evitar recrear figuras en loops

```python
# LENTO — crea una figura NUEVA en cada iteración (overhead de allocación repetido)
for cliente in clientes:
    fig, ax = plt.subplots()
    ax.plot(datos_de(cliente))
    fig.savefig(f"{cliente}.png")
    plt.close(fig)

# MÁS RÁPIDO — reutiliza la misma figura, solo limpia y redibuja
fig, ax = plt.subplots()
for cliente in clientes:
    ax.clear()                    # limpia el contenido, mantiene la figura/Axes ya creados
    ax.plot(datos_de(cliente))
    fig.savefig(f"{cliente}.png")
```

`ax.clear()` reutilizando la misma `Figure`/`Axes` evita el costo de allocación de crear objetos nuevos en cada iteración — relevante al generar un reporte con un gráfico por cada fila de un DataFrame grande.

## Cuándo Matplotlib ya no es la herramienta correcta

| Síntoma | Alternativa |
|---|---|
| Millones de puntos, necesidad de interactividad fluida (zoom/pan en tiempo real) | **Datashader** (agrega antes de renderizar) o Bokeh/Plotly con WebGL |
| Dashboards web interactivos para usuarios finales | Plotly, Bokeh, o Matplotlib embebido en [[Streamlit/06 - Gráficos y Visualización|Streamlit]] |
| Actualización en tiempo real a alta frecuencia (>10 fps) de datos en streaming | Bibliotecas especializadas en tiempo real, o el patrón de blitting manual de arriba como mínimo |

## Ver también

- [[15 - Integración con NumPy, Pandas y Seaborn]]
- [[13 - Animaciones]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
