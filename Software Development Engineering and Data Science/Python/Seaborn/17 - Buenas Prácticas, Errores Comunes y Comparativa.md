---
tags: [seaborn, python, data-science, visualization, best-practices, cheat-sheet]
---

# 17 — Buenas Prácticas, Errores Comunes y Comparativa

> Cierra la serie iniciada en [[01 - Introducción y Arquitectura]].

## Confundir función figure-level con axes-level

```python
# ERROR — relplot() es figure-level, NO acepta 'ax='
fig, ax = plt.subplots()
sns.relplot(data=df, x="precio", y="demanda", ax=ax)     # TypeError

# CORRECTO — usar la versión axes-level equivalente
sns.scatterplot(data=df, x="precio", y="demanda", ax=ax)
```

Ver la tabla completa de equivalencias figure-level ↔ axes-level en [[01 - Introducción y Arquitectura#Dos niveles de funciones Figure-level vs Axes-level|Introducción y Arquitectura]] — este es, por lejos, el error más común para quien recién empieza con Seaborn.

## Pasar datos en formato wide sin darse cuenta

```python
# Con datos wide, 'hue' no tiene ningún efecto útil — no hay una columna categórica que mapear
sns.lineplot(data=df_wide)     # grafica cada columna como una línea, pero sin control fino de hue/size/style

# CORRECTO — convertir a long primero
df_long = df_wide.melt(id_vars="fecha", var_name="serie", value_name="valor")
sns.lineplot(data=df_long, x="fecha", y="valor", hue="serie")
```

Ver el detalle completo en [[02 - Formato de Datos - Long vs Wide]] — la mayoría de "Seaborn no hace lo que yo esperaba" se resuelve revisando si los datos están en el formato correcto.

## Ignorar el costo del bootstrap en datasets grandes

Ya cubierto en detalle en [[16 - Rendimiento y Datasets Grandes]] — recordatorio rápido: con datasets grandes, considerar `errorbar=None` o pre-agregar con Pandas antes de graficar si el tiempo de renderizado se vuelve un problema.

## Usar `swarmplot`/`stripplot` cuando `boxenplot`/`violinplot` comunicarían mejor

Con datasets grandes, mostrar cada punto individual (`swarmplot`) satura visualmente el gráfico y no añade información sobre la forma real de la distribución — un `violinplot` o `boxenplot` (ver [[06 - Gráficos Categóricos II - Estimadores]]) comunica la forma de la distribución completa de forma más legible cuando el volumen de datos es alto.

## No fijar `hue_order`/`order` — el orden por default puede no ser el deseado

```python
# Sin especificar orden, Seaborn usa el orden de aparición o alfabético — puede no ser el lógico
sns.boxplot(data=df, x="talla", y="precio")     # ¿"S, M, L, XL" o alfabético "L, M, S, XL"?

# CORRECTO — orden explícito cuando existe un orden lógico real
sns.boxplot(data=df, x="talla", y="precio", order=["S", "M", "L", "XL"])
```

El mismo problema conceptual que el dtype `category` ordenado en Pandas (ver [[Python/Pandas/12 - Texto y Datos Categóricos#Categorías ordenadas|Python/Pandas]]) — si la variable categórica tiene un orden lógico natural, especificarlo explícitamente evita que Seaborn elija un orden arbitrario.

## Checklist antes de compartir un gráfico de Seaborn

- [ ] ¿Se usó la función figure-level (`catplot`/`relplot`/`displot`) en vez de axes-level cuando se necesitaba facetado?
- [ ] ¿Los datos están en formato long antes de usar `hue`/`col`/`row`?
- [ ] ¿El orden de las categorías (`order=`/`hue_order=`) refleja un orden lógico real, no uno arbitrario?
- [ ] ¿El colormap/paleta es apropiado al tipo de dato (secuencial/divergente/cualitativo)? Ver [[11 - Paletas de Color]].
- [ ] Con datasets grandes, ¿se consideró el costo del bootstrap y la saturación visual de `swarmplot`/scatter denso?

## Comparativa: Seaborn vs Plotnine vs Plotly Express

| | Seaborn | Plotnine | Plotly Express |
|---|---|---|---|
| Filosofía | Funciones especializadas por tipo de gráfico | Grammar of Graphics completa (clon de ggplot2) | API de alto nivel, salida interactiva |
| Interactividad | No (estática, vía Matplotlib) | No (estática) | Sí, nativa (zoom, hover) |
| Curva de aprendizaje | Baja | Media (requiere entender la gramática de capas) | Baja |
| Mejor para | Exploración estadística rápida, reportes estáticos | Quien viene de R/ggplot2, control compositivo fino | Dashboards y exploración interactiva en notebook/web |

**Regla práctica:** Seaborn sigue siendo el default razonable para análisis exploratorio estático en Python — Plotnine es la opción natural para quien piensa en términos de "grammar of graphics" (viniendo de R); Plotly Express cuando la interactividad (hover, zoom) aporta valor real, típicamente en un dashboard o notebook compartido. Ver también la comparativa más amplia (incluyendo Bokeh) en [[Python/Matplotlib/17 - Buenas Prácticas, Errores Comunes y Comparativa|Python/Matplotlib]].

## Ver también

- [[01 - Introducción y Arquitectura]]
- [[02 - Formato de Datos - Long vs Wide]]
- [[16 - Rendimiento y Datasets Grandes]]
- [[Python/Matplotlib/17 - Buenas Prácticas, Errores Comunes y Comparativa|Python/Matplotlib]]
- [[Python/Pandas/17 - Buenas Prácticas, Errores Comunes y Comparativa|Python/Pandas]]
