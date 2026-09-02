---
tags: [streamlit, dashboards, dataframes, cheat-sheet]
---

# 05 — Mostrar Datos: DataFrames, Tablas y Métricas

> Continúa de [[04 - Layout y Contenedores]].

## `st.dataframe` — tabla interactiva

```python
st.dataframe(df)
```

```python
st.dataframe(
    df,
    use_container_width=True,   # ocupa todo el ancho disponible
    height=400,
    hide_index=True,
    column_config={
        "precio": st.column_config.NumberColumn("Precio (USD)", format="$%.2f"),
        "fecha": st.column_config.DateColumn("Fecha", format="DD/MM/YYYY"),
        "porcentaje": st.column_config.ProgressColumn("Avance", min_value=0, max_value=100),
    },
)
```

`st.dataframe` es interactivo por defecto: el usuario puede ordenar columnas, redimensionarlas, y buscar — a diferencia de `st.table` (siguiente sección), que es completamente estático.

### `column_config` — controlar cómo se ve cada columna

```python
st.dataframe(df, column_config={
    "url_producto": st.column_config.LinkColumn("Producto"),
    "imagen": st.column_config.ImageColumn("Vista previa"),
    "categoria": st.column_config.SelectboxColumn("Categoría", options=["A", "B", "C"]),
    "tendencia": st.column_config.LineChartColumn("Tendencia"),   # mini-gráfico DENTRO de la celda
    "barras": st.column_config.BarChartColumn("Distribución"),
})
```

`LineChartColumn`/`BarChartColumn` renderizan un sparkline directamente dentro de cada celda (esperando una lista de valores por fila) — útil para mostrar tendencias históricas por fila sin necesitar una gráfica separada.

## `st.data_editor` — tabla EDITABLE por el usuario

```python
df_editado = st.data_editor(df, num_rows="dynamic")   # "dynamic" permite agregar/eliminar filas
```

```python
df_editado = st.data_editor(
    df,
    column_config={
        "aprobado": st.column_config.CheckboxColumn("¿Aprobado?"),
        "cantidad": st.column_config.NumberColumn(min_value=0, step=1),
    },
    disabled=["id"],   # columnas que el usuario NO puede editar
)
```

El valor retornado es el DataFrame **con las ediciones del usuario ya aplicadas** — el patrón estándar para construir herramientas internas donde alguien necesita corregir datos manualmente (ej. ajustar manualmente una predicción antes de aprobarla, o curar una lista de excepciones).

## `st.table` — tabla estática

```python
st.table(df.head())
```

A diferencia de `st.dataframe`, `st.table` renderiza toda la tabla como HTML estático de una sola vez — sin scroll, sin ordenamiento interactivo, sin límite de altura. Apropiado solo para tablas pequeñas donde la interactividad no aporta nada (ej. un resumen de 5 filas).

## `st.metric` — indicador de KPI con delta

```python
st.metric(label="MAE", value="12.4", delta="-1.7")   # delta verde (mejora) si es negativo Y la métrica es "menor es mejor"
st.metric(label="Demanda", value="1,240", delta="12%")
st.metric(label="Errores", value="3", delta="2", delta_color="inverse")   # invierte la lógica de color
```

```python
# delta_color: "normal" (verde=positivo, rojo=negativo), "inverse" (invertido), "off" (sin color)
st.metric("Tasa de error", "2.1%", delta="0.3%", delta_color="inverse")   # un AUMENTO de errores debe verse en rojo
```

`delta_color="inverse"` es importante para métricas donde "más alto" es malo (errores, latencia, churn) — sin esto, un aumento se mostraría en verde por defecto, transmitiendo la señal visual equivocada.

### Fila de métricas dentro de columnas — el patrón de dashboard más común

```python
col1, col2, col3 = st.columns(3)
col1.metric("MAE", "12.4", "-1.7")
col2.metric("RMSE", "18.7", "-1.1")
col3.metric("R²", "0.89", "0.04")
```

## `st.json` — mostrar estructuras JSON/dict de forma navegable

```python
st.json({
    "modelo": "xgboost-v3",
    "hiperparametros": {"n_estimators": 300, "max_depth": 6},
    "metricas": {"mae": 12.4, "rmse": 18.7},
})
```

```python
st.json(data, expanded=False)   # inicia colapsado, útil para JSON grandes
```

## Descargar datos desde la app — `st.download_button`

```python
csv = df.to_csv(index=False).encode("utf-8")
st.download_button(
    label="Descargar CSV",
    data=csv,
    file_name="demanda_filtrada.csv",
    mime="text/csv",
)
```

```python
# Descargar un modelo serializado directamente desde la app:
import joblib, io

buffer = io.BytesIO()
joblib.dump(modelo, buffer)
st.download_button("Descargar modelo", data=buffer.getvalue(), file_name="modelo.joblib")
```

## `st.dataframe` vs. `st.table` vs. `st.data_editor` — tabla de decisión

| Necesitas | Elemento |
|---|---|
| Explorar datos, ordenar/buscar, tabla grande | `st.dataframe` |
| Tabla pequeña, puramente informativa, sin interacción | `st.table` |
| Que el usuario pueda MODIFICAR los datos | `st.data_editor` |
| Un número destacado con contexto de tendencia | `st.metric` |
| Estructura anidada (dict/JSON), no tabular | `st.json` |

## Ver también

- [[04 - Layout y Contenedores]]
- [[06 - Gráficos y Visualización]]
- `Scikit-learn/13 - Persistencia de Modelos.md`
