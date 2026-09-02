---
tags: [seaborn, python, data-science, visualization, styling, cheat-sheet]
---

# 12 — Temas y Estética

> Continúa de [[11 - Paletas de Color]].

## `set_theme()` — configurar todo de una vez

```python
sns.set_theme()                                              # aplica el tema default de Seaborn a TODO (incluido Matplotlib puro)
sns.set_theme(style="whitegrid", palette="viridis", context="talk")
```

`set_theme()` (el reemplazo moderno de las funciones separadas `set_style`/`set_palette`/`set_context` de versiones antiguas) es la forma recomendada de fijar la estética de toda una sesión/notebook en una sola llamada al inicio — y afecta también a gráficos hechos con Matplotlib puro después de llamarla, no solo a los de Seaborn.

## `set_style()` — el fondo y las líneas de referencia

```python
sns.set_style("white")           # fondo blanco, sin grid — minimalista
sns.set_style("whitegrid")         # fondo blanco CON grid — el más usado para gráficos analíticos
sns.set_style("darkgrid")            # fondo gris claro con grid blanco — el default histórico de Seaborn
sns.set_style("dark")                  # fondo gris, sin grid
sns.set_style("ticks")                   # fondo blanco, sin grid, con marcas de tick visibles en los bordes
```

| Estilo | Fondo | Grid | Cuándo usarlo |
|---|---|---|---|
| `whitegrid` | Blanco | Sí | Comparar valores con precisión (barras, líneas) |
| `darkgrid` | Gris claro | Sí | Default general, buen contraste |
| `white` / `ticks` | Blanco | No | Gráficos limpios para publicación, scatter/relación |
| `dark` | Gris | No | Poco usado; estética sin grid pero con fondo oscuro |

## `set_context()` — escalar todo para el medio de destino

```python
sns.set_context("paper")       # elementos pequeños — para figuras en un documento/paper académico
sns.set_context("notebook")      # default — tamaño intermedio, para exploración en Jupyter
sns.set_context("talk")            # elementos grandes — para presentaciones proyectadas
sns.set_context("poster")            # elementos muy grandes — para pósters impresos
```

`set_context()` escala proporcionalmente el grosor de línea, tamaño de fuente y tamaño de marcador **sin** tener que ajustar cada uno individualmente vía `rcParams` — el mismo código de graficación produce una versión visualmente apropiada para cada medio con solo cambiar este argumento.

## Aplicar un estilo temporalmente

```python
with sns.axes_style("darkgrid"):
    sns.scatterplot(data=df, x="precio", y="demanda")
# fuera del 'with', vuelve al estilo anterior
```

Igual que `plt.style.context()` de Matplotlib (ver [[Python/Matplotlib/10 - Estilos y rcParams|Python/Matplotlib]]), usar el context manager evita que un estilo puntual contamine el resto de gráficos de la sesión.

## `despine()` — quitar bordes sobrantes

```python
ax = sns.scatterplot(data=df, x="precio", y="demanda")
sns.despine()                             # quita los bordes superior y derecho (default) — look más limpio
sns.despine(left=True, bottom=True)         # quita también izquierdo/inferior si se desea un look aún más minimalista
sns.despine(offset=10)                        # además, desplaza los bordes restantes 10 puntos hacia afuera
```

`despine()` es un atajo específico de Seaborn sobre `ax.spines[...].set_visible(False)` de Matplotlib (ver [[Python/Matplotlib/02 - Anatomía de una Figura#Manipular los spines (bordes)|Python/Matplotlib]]) — la estética "sin bordes superior/derecho" es tan común en visualización de datos que Seaborn le dedica una función propia de una sola llamada.

## Combinar rcParams de Matplotlib con Seaborn

```python
sns.set_theme(style="whitegrid")
plt.rcParams["figure.figsize"] = (10, 6)      # rcParams de Matplotlib se pueden seguir ajustando después de set_theme()
```

`set_theme()` internamente modifica los mismos `rcParams` de Matplotlib — cualquier ajuste manual posterior de `rcParams` sigue funcionando y se combina con el tema de Seaborn ya aplicado.

## Ver también

- [[11 - Paletas de Color]]
- [[13 - Estadística Integrada]]
- [[Python/Matplotlib/10 - Estilos y rcParams|Python/Matplotlib]]
