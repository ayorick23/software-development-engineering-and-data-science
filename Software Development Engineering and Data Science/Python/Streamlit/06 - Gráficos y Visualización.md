---
tags: [streamlit, dashboards, visualizacion, matplotlib, plotly, cheat-sheet]
---

# 06 — Gráficos y Visualización

> Continúa de [[05 - Mostrar Datos - DataFrames, Tablas y Métricas]]. Ver también la skill `dataviz` para principios de diseño de visualizaciones en general.

## Gráficos nativos — rápidos, con menos control

```python
st.line_chart(df, x="fecha", y="demanda")
st.bar_chart(df, x="region", y="ventas")
st.area_chart(df, x="fecha", y="demanda")
st.scatter_chart(df, x="precio", y="cantidad", color="region", size="importancia")
```

```python
# Múltiples series a la vez, si el DataFrame tiene varias columnas numéricas:
st.line_chart(df.set_index("fecha")[["demanda_rd", "demanda_pr", "demanda_mx"]])
```

Los charts nativos (basados en Vega-Lite internamente) son la opción más rápida para exploración — cubren el 80% de los casos de dashboard estándar (series de tiempo, comparación de categorías) sin necesitar importar una librería de gráficos adicional. Para necesidades más específicas (subplots, anotaciones complejas, tipos de gráfico especializados), se recurre a Matplotlib/Plotly/Altair.

## `st.pyplot` — integración con Matplotlib

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()
ax.plot(df["fecha"], df["demanda"])
ax.set_xlabel("Fecha")
ax.set_ylabel("Demanda")
st.pyplot(fig)
```

Cualquier gráfica de Matplotlib (incluyendo las generadas por seaborn, que se construye sobre Matplotlib) se muestra pasando el objeto `Figure` a `st.pyplot` — el patrón estándar para reutilizar código de visualización ya escrito en notebooks sin reescribirlo.

```python
import seaborn as sns

fig, ax = plt.subplots()
sns.heatmap(df.corr(), annot=True, ax=ax)
st.pyplot(fig)
```

## `st.plotly_chart` — integración con Plotly (interactivo)

```python
import plotly.express as px

fig = px.line(df, x="fecha", y="demanda", color="region", title="Demanda por región")
st.plotly_chart(fig, use_container_width=True)
```

A diferencia de `st.pyplot` (imagen estática renderizada del lado del servidor), `st.plotly_chart` mantiene toda la interactividad nativa de Plotly (zoom, hover con tooltips, pan) directamente en el navegador del usuario — preferible cuando la exploración interactiva del gráfico mismo es valiosa, no solo el resultado final.

### Capturar eventos de interacción del usuario sobre el gráfico

```python
evento = st.plotly_chart(fig, use_container_width=True, on_select="rerun", key="grafico_demanda")

if evento and evento["selection"]["points"]:
    puntos_seleccionados = evento["selection"]["points"]
    st.write("Seleccionaste:", puntos_seleccionados)
```

`on_select="rerun"` convierte el gráfico en un widget interactivo más: seleccionar puntos con el mouse dispara un rerun del script con la selección disponible — útil para construir dashboards donde hacer clic/seleccionar en una gráfica filtra el resto de la página (drill-down interactivo).

## `st.altair_chart` — integración con Altair (Vega-Lite)

```python
import altair as alt

chart = alt.Chart(df).mark_line().encode(
    x="fecha:T",
    y="demanda:Q",
    color="region:N",
).interactive()

st.altair_chart(chart, use_container_width=True)
```

Los charts nativos de Streamlit (`st.line_chart`, etc.) están construidos internamente sobre Altair/Vega-Lite — usar Altair directamente da acceso a la gramática de visualización completa (encodings, transformaciones, capas) cuando los charts nativos son demasiado simples para lo que se necesita.

## `st.bokeh_chart`, `st.pydeck_chart`, `st.graphviz_chart`

```python
# Mapas geoespaciales interactivos (PyDeck, basado en deck.gl):
import pydeck as pdk

st.pydeck_chart(pdk.Deck(
    layers=[pdk.Layer("ScatterplotLayer", data=df, get_position=["lon", "lat"], get_radius=1000)],
    initial_view_state=pdk.ViewState(latitude=18.4861, longitude=-69.9312, zoom=8),
))

# Grafos/diagramas de flujo:
st.graphviz_chart("""
digraph {
    "Datos crudos" -> "Preprocesamiento" -> "Modelo" -> "Predicción"
}
""")
```

`st.pydeck_chart` es la opción estándar para visualización geoespacial (mapas de calor, dispersión sobre coordenadas) dentro de Streamlit — relevante para dashboards con componente geográfico (ej. demanda por oficina/región sobre un mapa real).

## `st.map` — mapa simple, atajo sobre PyDeck

```python
st.map(df, latitude="lat", longitude="lon", size="demanda", color="#1f77b4")
```

Versión simplificada de `st.pydeck_chart` para el caso común de "puntos sobre un mapa" — suficiente cuando no se necesita el control fino de capas de PyDeck.

## `st.image` — mostrar imágenes estáticas

```python
st.image("logo.png", width=200)
st.image(imagen_pil, caption="Predicción visual del modelo")

# Desde un array de NumPy (ej. salida de un modelo de visión):
st.image(array_numpy, clamp=True)
```

## Elegir la herramienta de gráficos correcta

| Necesitas | Opción |
|---|---|
| Exploración rápida, sin código adicional | Charts nativos (`st.line_chart`, etc.) |
| Reutilizar código de Matplotlib/seaborn ya existente | `st.pyplot` |
| Interactividad rica (zoom, hover, selección) | `st.plotly_chart` |
| Control total sobre la gramática de visualización | `st.altair_chart` |
| Mapas geoespaciales | `st.pydeck_chart` / `st.map` |
| Diagramas de flujo/grafos | `st.graphviz_chart` |

## Ver también

- [[05 - Mostrar Datos - DataFrames, Tablas y Métricas]]
- [[08 - Caching - cache_data y cache_resource]] (cachear la generación de gráficas costosas)
- Skill `dataviz` (principios de diseño de visualización)
