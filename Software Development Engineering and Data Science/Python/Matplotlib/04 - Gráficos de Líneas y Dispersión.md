---
tags: [matplotlib, python, data-science, visualization, cheat-sheet]
---

# 04 — Gráficos de Líneas y Dispersión

> Continúa de [[03 - Creación de Figuras y Subplots]].

## `plot()` — gráfico de líneas

```python
fig, ax = plt.subplots()
x = np.linspace(0, 10, 100)
y = np.sin(x)

ax.plot(x, y)                                    # línea simple
ax.plot(x, y, label="seno")                        # con etiqueta para la leyenda
ax.plot(x, np.sin(x), x, np.cos(x))                  # dos series en una sola llamada
ax.plot(x, np.sin(x), label="seno")
ax.plot(x, np.cos(x), label="coseno")                # forma más legible: una llamada por serie
ax.legend()
```

## Múltiples series con formato abreviado

```python
ax.plot(x, y, "r--")           # rojo ('r'), línea discontinua ('--') — sintaxis abreviada estilo MATLAB
ax.plot(x, y, "bo")               # azul ('b'), puntos ('o') sin línea conectando
ax.plot(x, y, color="green", linestyle="-.", linewidth=2, marker="^", markersize=8)   # forma explícita, más legible
```

Ver el catálogo completo de colores, estilos de línea y marcadores en [[07 - Personalización de Líneas, Marcadores y Colores]].

## `scatter()` — gráfico de dispersión

```python
x = np.random.rand(50)
y = np.random.rand(50)

ax.scatter(x, y)
ax.scatter(x, y, s=100, c="red", alpha=0.6, edgecolors="black")   # tamaño, color, transparencia, borde
```

## `scatter()` con color y tamaño variable por punto (cuarta y quinta dimensión)

```python
tamanos = np.random.rand(50) * 300
colores = np.random.rand(50)

scatter = ax.scatter(x, y, s=tamanos, c=colores, cmap="viridis", alpha=0.7)
fig.colorbar(scatter, ax=ax, label="Valor de la variable de color")
```

Codificar información adicional con el **tamaño** y **color** de cada punto (además de su posición X/Y) permite representar hasta 4 dimensiones de datos en un solo gráfico 2D — la barra de color (`colorbar`) es indispensable para que esa dimensión extra sea interpretable (ver [[11 - Mapas de Calor, Imágenes y Colorbars]]).

## `plot()` vs `scatter()`: cuándo usar cada uno

| | `plot()` | `scatter()` |
|---|---|---|
| Uso típico | Series continuas/ordenadas (series de tiempo, funciones) | Puntos independientes, correlación entre dos variables |
| Conecta puntos con línea | Sí, por default | No — cada punto es independiente |
| Tamaño/color variable por punto | No (uniforme para toda la serie) | Sí (`s=`, `c=` aceptan arrays) |
| Rendimiento con muchos puntos | Más rápido | Más lento con decenas de miles de puntos |

## Líneas de referencia horizontales/verticales

```python
ax.axhline(y=0, color="gray", linestyle="--")        # línea horizontal completa en y=0
ax.axvline(x=5, color="red", linestyle=":")            # línea vertical completa en x=5
ax.axhspan(ymin=0.4, ymax=0.6, alpha=0.2, color="yellow")   # banda horizontal sombreada
```

Útiles para marcar umbrales, promedios, o rangos de referencia sin necesidad de graficarlos como una serie de datos propiamente.

## Rellenar área bajo la curva

```python
ax.fill_between(x, y1, y2=0, alpha=0.3)             # rellena el área entre la curva y=0 (o entre dos curvas si se pasa y2)
ax.fill_between(x, y_inferior, y_superior, alpha=0.2, label="intervalo de confianza")
```

`fill_between` con dos curvas (`y1`, `y2`) es el patrón estándar para visualizar bandas de incertidumbre/intervalos de confianza alrededor de una predicción.

## Ver también

- [[03 - Creación de Figuras y Subplots]]
- [[05 - Gráficos de Barras e Histogramas]]
- [[07 - Personalización de Líneas, Marcadores y Colores]]
- [[11 - Mapas de Calor, Imágenes y Colorbars]]
