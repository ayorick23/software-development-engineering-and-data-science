---
tags: [polars, python, data-science, integrations, cheat-sheet]
---

# 16 — Integración con el Ecosistema

> Continúa de [[15 - Exportación de Datos]].

## Pandas — interoperabilidad bidireccional

```python
df_polars = pl.from_pandas(df_pandas)
df_pandas_de_vuelta = df_polars.to_pandas()
```

Esta conversión **copia** los datos entre las dos representaciones de memoria (Arrow en Polars, NumPy en Pandas clásico) — si Pandas usa el backend PyArrow (`dtype_backend="pyarrow"`, ver [[Python/Pandas/15 - Rendimiento y Optimización#Backend PyArrow (dtype_backend)|Python/Pandas]]), la conversión puede evitar la copia porque ambos ya comparten el mismo formato de memoria subyacente.

## NumPy

```python
arr = df.to_numpy()                        # DataFrame completo -> ndarray 2D
serie_np = df["precio"].to_numpy()           # una columna -> ndarray 1D

df_desde_numpy = pl.DataFrame(np.random.randn(5, 3), schema=["a", "b", "c"])
```

Ver la estructura de `ndarray` subyacente en [[Python/NumPy/01 - Introducción y Arquitectura Interna|Python/NumPy]].

## Apache Arrow — el formato nativo compartido

```python
tabla_arrow = df.to_arrow()          # sin copia (zero-copy) en la mayoría de los casos
df_desde_arrow = pl.from_arrow(tabla_arrow)
```

Como Polars usa Arrow como su formato de memoria interno, la conversión hacia/desde otras herramientas basadas en Arrow (DuckDB, PyArrow, algunas versiones de Pandas) puede evitar por completo la copia de datos — una ventaja estructural que Pandas clásico (basado en NumPy) no tiene de forma nativa.

## DuckDB — consultas SQL directamente sobre DataFrames de Polars

```python
import duckdb

resultado = duckdb.sql("SELECT region, SUM(monto) FROM df GROUP BY region").pl()   # 'df' es el DataFrame de Polars en el scope
```

Tanto Polars como DuckDB comparten el formato Arrow — DuckDB puede consultar un DataFrame de Polars directamente con SQL sin necesitar importarlo o convertirlo primero, y `.pl()` al final de la consulta devuelve el resultado como un DataFrame de Polars. Ver [[Machine Learning/04-Ingenieria-de-Datos#DuckDB|Machine Learning/04 - Ingeniería de Datos]].

## Scikit-learn

```python
from sklearn.linear_model import LinearRegression

X = df.select(["precio", "stock"]).to_numpy()     # scikit-learn espera arrays de NumPy, no DataFrames de Polars directamente
y = df["demanda"].to_numpy()

modelo = LinearRegression()
modelo.fit(X, y)
```

A diferencia de Pandas (que scikit-learn acepta cada vez más nativamente, ver [[Scikit-learn/01 - Introducción, Filosofía y la API Consistente|Scikit-learn]]), la integración de Polars con scikit-learn todavía requiere convertir explícitamente a NumPy antes de `.fit()` en la mayoría de los casos.

## Streaming hacia Streamlit / FastAPI

```python
# Streamlit puede mostrar un DataFrame de Polars directamente
import streamlit as st
st.dataframe(df)

# FastAPI — convertir a estructura serializable
@app.get("/ventas")
def obtener_ventas():
    return df.to_dicts()     # lista de diccionarios, análogo a to_dict(orient="records") de Pandas
```

Ver [[Streamlit/05 - Mostrar Datos - DataFrames, Tablas y Métricas|Streamlit]] y [[FastAPI/04 - Response Models y Serialización|FastAPI]] para el patrón completo del lado de Pandas, directamente trasladable.

## Ver también

- [[15 - Exportación de Datos]]
- [[01 - Introducción y Arquitectura]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
- [[Python/Pandas/16 - Integración con el Ecosistema|Python/Pandas]]
