---
tags: [matplotlib, python, data-science, visualization, best-practices, cheat-sheet]
---

# 17 — Buenas Prácticas, Errores Comunes y Comparativa

> Cierra la serie iniciada en [[Python/Matplotlib/01 - Introducción y Arquitectura]].

## Mezclar el estilo `pyplot` y el orientado a objetos sin querer

```python
# CONFUSO — con dos figuras activas, plt.title() modifica la "figura actual", que puede no ser la esperada
fig1, ax1 = plt.subplots()
fig2, ax2 = plt.subplots()
plt.title("¿A cuál figura le puse este título?")     # ambiguo, depende de cuál se creó/activó más recientemente

# CORRECTO — siempre explícito sobre qué Axes se modifica
ax1.set_title("Título de la figura 1")
ax2.set_title("Título de la figura 2")
```

Ver la comparación completa de ambos estilos en [[Python/Matplotlib/01 - Introducción y Arquitectura]] — la recomendación general es usar SIEMPRE el estilo orientado a objetos fuera de exploración interactiva de un único gráfico.

## Olvidar `plt.close()` en loops — fuga de memoria

```python
# Genera cientos de figuras SIN cerrarlas — la memoria crece sin límite
for cliente in clientes:
    fig, ax = plt.subplots()
    ax.plot(datos_de(cliente))
    fig.savefig(f"{cliente}.png")
    # falta plt.close(fig) aquí
```

Ver la solución completa en [[16 - Rendimiento y Grandes Volúmenes de Datos]].

## `plot()` en vez de `bar()` para categorías (o viceversa)

```python
# INCORRECTO conceptualmente — plot() conecta con líneas, implicando una progresión continua entre categorías que no existe
ax.plot(["Norte", "Sur", "Centro"], [100, 200, 150])

# CORRECTO — bar() no implica ningún orden/continuidad entre categorías
ax.bar(["Norte", "Sur", "Centro"], [100, 200, 150])
```

Usar `plot()` (línea continua) para datos categóricos sugiere visualmente una transición/tendencia entre categorías que no tiene ningún sentido real — reservar `plot()` para datos genuinamente ordenados/continuos (series de tiempo, funciones matemáticas).

## Truncar el eje Y engañosamente

```python
# ENGAÑOSO — un eje Y que no empieza en 0 puede exagerar visualmente diferencias pequeñas
ax.bar(categorias, [98, 99, 97, 100])
ax.set_ylim(95, 100)     # la diferencia entre barras se ve MUCHO más grande de lo que realmente es
```

Para gráficos de **barras**, el eje Y debe empezar en 0 salvo justificación explícita y visible — porque la altura de la barra es la señal visual principal que el lector interpreta como magnitud. Para gráficos de **línea** mostrando tendencia (no magnitud absoluta), truncar el eje es más aceptable, pero siempre debe indicarse claramente.

## Colormap `jet` para datos continuos

Ya cubierto en detalle en [[07 - Personalización de Líneas, Marcadores y Colores#Colormaps — paletas continuas para datos numéricos|Personalización de Colores]] — usar `viridis` u otro colormap perceptualmente uniforme por default.

## No etiquetar los ejes

```python
# Un gráfico sin xlabel/ylabel/title obliga al lector a adivinar qué representa
ax.plot(x, y)

# Mínimo aceptable para cualquier gráfico que salga de una exploración puramente personal
ax.plot(x, y)
ax.set_xlabel("Tiempo (días)")
ax.set_ylabel("Ventas (USD)")
ax.set_title("Ventas diarias — Q1 2026")
```

## Checklist antes de considerar un gráfico "listo" para compartir

- [ ] ¿Tiene título, y etiquetas en ambos ejes con sus unidades?
- [ ] ¿El eje Y de cualquier gráfico de barras empieza en 0?
- [ ] ¿El colormap es perceptualmente uniforme (no `jet`) si los datos son continuos?
- [ ] ¿La leyenda es necesaria y no tapa datos importantes?
- [ ] ¿Se exportó en un formato apropiado (vectorial para reportes, PNG de alta resolución para web)?

## Comparativa: Matplotlib vs Seaborn vs Plotly vs Bokeh

| | Matplotlib | Seaborn | Plotly | Bokeh |
|---|---|---|---|---|
| Nivel de abstracción | Bajo (control total) | Medio (estadístico, sobre Matplotlib) | Medio-alto | Medio-alto |
| Interactividad nativa | No (estática por default) | No (hereda de Matplotlib) | Sí (zoom, hover, pan de fábrica) | Sí |
| Salida típica | Imagen estática (PNG/SVG/PDF) | Imagen estática | HTML interactivo / notebook | HTML interactivo / dashboards |
| Mejor para | Publicaciones, control fino, cualquier gráfico base | Exploración estadística rápida con buena estética | Dashboards interactivos, exploración web | Dashboards interactivos, streaming de datos |
| Curva de aprendizaje | Media-alta | Baja | Baja-media | Media |

**Regla práctica:** Matplotlib (a veces vía Seaborn) para reportes, publicaciones y cualquier imagen estática; Plotly para dashboards interactivos rápidos de construir; Bokeh cuando se necesita más control sobre interactividad compleja o datos en streaming — ninguna reemplaza completamente a las demás, y es común usar Matplotlib para el análisis exploratorio y Plotly/Bokeh solo para la capa final de presentación.

## Ver también

- [[Python/Matplotlib/01 - Introducción y Arquitectura]]
- [[07 - Personalización de Líneas, Marcadores y Colores]]
- [[16 - Rendimiento y Grandes Volúmenes de Datos]]
- [[Python/Pandas/17 - Buenas Prácticas, Errores Comunes y Comparativa|Python/Pandas]]
