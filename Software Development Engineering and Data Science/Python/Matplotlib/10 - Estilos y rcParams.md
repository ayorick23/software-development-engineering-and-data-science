---
tags: [matplotlib, python, data-science, visualization, styling, cheat-sheet]
---

# 10 — Estilos y rcParams

> Continúa de [[09 - Texto, Anotaciones y Leyendas]].

## Stylesheets — cambiar todo el look de golpe

```python
plt.style.available            # lista de estilos incluidos: 'ggplot', 'seaborn-v0_8', 'fivethirtyeight', 'dark_background'...
plt.style.use("ggplot")          # aplica GLOBALMENTE a todos los gráficos siguientes en el script/sesión
```

## Aplicar un estilo solo temporalmente (recomendado)

```python
with plt.style.context("dark_background"):
    fig, ax = plt.subplots()
    ax.plot(x, y)
    # fuera del 'with', el estilo vuelve al anterior — no contamina el resto del notebook/script
```

`plt.style.context()` como context manager es preferible a `plt.style.use()` global en notebooks — evita que un estilo aplicado para un gráfico específico afecte accidentalmente a todos los gráficos posteriores de la sesión.

## `rcParams` — configuración fina, parámetro por parámetro

```python
plt.rcParams["figure.figsize"] = (10, 6)          # tamaño default para TODAS las figuras nuevas
plt.rcParams["font.size"] = 12
plt.rcParams["axes.grid"] = True
plt.rcParams["axes.spines.top"] = False             # oculta el borde superior en TODOS los Axes por default
plt.rcParams["lines.linewidth"] = 2
plt.rcParams["savefig.dpi"] = 300
```

`rcParams` es un diccionario global que define los valores **default** de prácticamente cualquier propiedad visual — fijarlo una vez al inicio de un notebook/script evita repetir los mismos argumentos (`linewidth=2`, `figsize=(10,6)`...) en cada llamada a `plot()`/`subplots()`.

## Restaurar la configuración default

```python
plt.rcdefaults()             # revierte TODOS los rcParams a los valores originales de Matplotlib
```

## Definir un estilo propio reutilizable

```python
plt.rcParams.update({
    "figure.figsize": (10, 6),
    "axes.spines.top": False,
    "axes.spines.right": False,
    "axes.grid": True,
    "grid.alpha": 0.3,
    "font.family": "sans-serif",
})
```

Guardar este bloque como una función `aplicar_estilo_corporativo()` que se llama al inicio de cada notebook/script del proyecto es una forma simple de mantener consistencia visual entre todos los gráficos de un reporte sin depender de un archivo `.mplstyle` separado.

## Archivo de estilo custom (`.mplstyle`)

```python
# archivo: mi_estilo.mplstyle
# figure.figsize: 10, 6
# axes.grid: True
# font.size: 12

plt.style.use("./mi_estilo.mplstyle")
```

Un archivo `.mplstyle` es la forma de compartir un estilo consistente **entre proyectos y personas** (versionable en git) en vez de copiar el mismo bloque de `rcParams.update()` en cada notebook.

## Grid (cuadrícula de fondo)

```python
ax.grid(True)
ax.grid(True, which="major", linestyle="--", alpha=0.5)
ax.grid(True, axis="y")            # solo líneas horizontales, no verticales — reduce el ruido visual en gráficos de barras
```

## Ver también

- [[09 - Texto, Anotaciones y Leyendas]]
- [[07 - Personalización de Líneas, Marcadores y Colores]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
