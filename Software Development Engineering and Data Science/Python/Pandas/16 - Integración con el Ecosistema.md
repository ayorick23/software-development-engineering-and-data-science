---
tags: [pandas, python, data-science, integrations, cheat-sheet]
---

# 16 — Integración con el Ecosistema

> Continúa de [[15 - Rendimiento y Optimización]]. Pandas raramente se usa aislado — este es el mapa de cómo conecta con el resto del stack ya cubierto en el baúl.

## NumPy — la base

```python
arr = df["precio"].to_numpy()        # Series -> ndarray (forma preferida sobre el atributo .values)
matriz = df[["precio", "stock"]].to_numpy()   # DataFrame -> matriz 2D
df_desde_numpy = pd.DataFrame(np.random.randn(5, 3), columns=["a", "b", "c"])
```

`.to_numpy()` es preferible sobre `.values` porque su comportamiento es explícito y consistente incluso con Extension Arrays (`category`, `Int64`), mientras que `.values` puede devolver tipos distintos según el dtype interno. Ver [[Python/NumPy/01 - Introducción y Arquitectura Interna|Python/NumPy]] y la arquitectura interna en [[Python/Pandas/01 - Introducción y Arquitectura Interna]].

## Matplotlib y Seaborn — graficación directa

```python
df["ventas"].plot(kind="line")                       # wrapper directo sobre Matplotlib
df.plot(x="fecha", y="ventas", kind="bar")
df.groupby("region")["monto"].sum().plot(kind="pie", autopct="%.1f%%")

import seaborn as sns
sns.barplot(data=df, x="region", y="monto")            # Seaborn consume DataFrames de Pandas nativamente
```

El método `.plot()` de Pandas es un atajo sobre Matplotlib — para gráficos con más control o estilo, usar Matplotlib/Seaborn directamente sobre las columnas del DataFrame. Ver [[Python/Matplotlib/15 - Integración con NumPy, Pandas y Seaborn|Python/Matplotlib]] para el detalle completo.

## Scikit-learn — DataFrames dentro de un `Pipeline`

```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

columnas_numericas = df.select_dtypes(include="number").columns.tolist()
columnas_categoricas = df.select_dtypes(include=["object", "category"]).columns.tolist()

preprocesador = ColumnTransformer([
    ("num", StandardScaler(), columnas_numericas),
    ("cat", OneHotEncoder(handle_unknown="ignore"), columnas_categoricas),
])
preprocesador.set_output(transform="pandas")   # fuerza que la salida siga siendo DataFrame, no ndarray
```

`set_output(transform="pandas")` (scikit-learn 1.2+) es la pieza clave para mantener nombres de columna legibles a través de todo un pipeline de preprocesamiento — sin esto, `ColumnTransformer` devuelve un `ndarray` sin nombres. Ver [[03 - Pipelines y ColumnTransformer|Scikit-learn]].

## SQLAlchemy — puente bidireccional con bases de datos

```python
from sqlalchemy import create_engine
engine = create_engine("postgresql://user:pass@host/db")

df = pd.read_sql("SELECT * FROM ventas", engine)
df.to_sql("ventas_procesadas", engine, if_exists="replace", index=False)
```

Ver [[02 - Creación y Carga de Datos]] y [[14 - Exportación de Datos]] para el detalle de parámetros; y [[11 - Bases de Datos con SQLAlchemy y SQLModel|FastAPI]] si el destino final es servir estos datos vía API.

## Pandera — validación de esquema del DataFrame

```python
import pandera.pandas as pa

schema = pa.DataFrameSchema({
    "precio": pa.Column(float, checks=pa.Check.gt(0)),
    "stock": pa.Column(int, checks=pa.Check.ge(0)),
})
df_validado = schema.validate(df)   # lanza SchemaError si algo no cumple
```

Pandera formaliza como código las mismas comprobaciones que se harían manualmente con `.isna()`, `.dtypes`, rangos (ver [[03 - Exploración e Inspección de Datos]]) — la diferencia es que queda como una pieza reproducible y testeable del pipeline. Ver [[Python/Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]].

## Dask y PySpark — escalar más allá de una máquina

```python
import dask.dataframe as dd
ddf = dd.read_csv("archivo_enorme_*.csv")   # API deliberadamente calcada de Pandas
resultado = ddf.groupby("region")["monto"].sum().compute()   # .compute() materializa el resultado
```

Dask replica intencionalmente la API de Pandas (`groupby`, `merge`, `.loc`) para minimizar la curva de aprendizaje al escalar — la diferencia central es que las operaciones son **perezosas** hasta llamar `.compute()`. Para volúmenes verdaderamente masivos en clúster, PySpark es la alternativa madura de nivel empresarial (ver `Machine Learning/07-Librerias-de-Data-Science-y-ML.md`).

## Streamlit y FastAPI — llevar el DataFrame a producción

```python
# Streamlit — mostrar interactivamente
import streamlit as st
st.dataframe(df, use_container_width=True)

# FastAPI — servir como JSON
@app.get("/ventas")
def obtener_ventas():
    return df.to_dict(orient="records")
```

Ver [[05 - Mostrar Datos - DataFrames, Tablas y Métricas|Streamlit]] y [[04 - Response Models y Serialización|FastAPI]] para el patrón completo, incluyendo tipado con Pydantic de las filas devueltas.

## Ver también

- [[Python/Pandas/01 - Introducción y Arquitectura Interna]]
- [[15 - Rendimiento y Optimización]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
- [[03 - Pipelines y ColumnTransformer|Scikit-learn]]
- [[Python/Pandera/01 - Introducción y Conceptos Fundamentales|Pandera]]
- [[05 - Mostrar Datos - DataFrames, Tablas y Métricas|Streamlit]]
- [[04 - Response Models y Serialización|FastAPI]]
