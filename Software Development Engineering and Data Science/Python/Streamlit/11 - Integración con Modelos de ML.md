---
tags: [streamlit, dashboards, machine-learning, mlflow, cheat-sheet]
---

# 11 — Integración con Modelos de ML

> Se apoya en [[08 - Caching - cache_data y cache_resource]] y [[09 - Forms, Callbacks y Control de Reruns]].

Uno de los usos más comunes de Streamlit en un flujo de trabajo de Data Science: construir rápidamente una interfaz interactiva para demostrar, probar o servir un modelo entrenado — sin necesitar un frontend separado.

## Patrón base — cargar el modelo una vez, predecir en cada interacción

```python
import streamlit as st
import joblib
import pandas as pd

@st.cache_resource   # el modelo se carga UNA sola vez, no en cada rerun — ver 08
def cargar_modelo():
    return joblib.load("modelo_produccion.joblib")

modelo = cargar_modelo()

st.title("Predicción de Demanda")

dias_atras = st.slider("Días atrás", 7, 365, 90)
region = st.selectbox("Región", ["RD", "PR", "MX"])

if st.button("Predecir"):
    entrada = pd.DataFrame({"dias_atras": [dias_atras], "region": [region]})
    prediccion = modelo.predict(entrada)
    st.metric("Demanda predicha", f"{prediccion[0]:.1f}")
```

## Cargar un modelo desde el Model Registry de MLflow

```python
import mlflow

@st.cache_resource
def cargar_modelo_mlflow():
    mlflow.set_tracking_uri("http://mlflow-server:5000")
    return mlflow.pyfunc.load_model("models:/claro-rd-demand-model@champion")

modelo = cargar_modelo_mlflow()
```

Ver `MLflow/07 - Model Registry.md` — este patrón conecta directamente el dashboard con el modelo actualmente marcado como `champion`, sin hardcodear una ruta de archivo local que quedaría desactualizada cada vez que se promueve una nueva versión.

## Predicción por lotes — subir un CSV, descargar predicciones

```python
archivo = st.file_uploader("Sube un CSV con los casos a predecir", type="csv")

if archivo is not None:
    df_entrada = pd.read_csv(archivo)
    st.write("Vista previa:")
    st.dataframe(df_entrada.head())

    if st.button("Generar predicciones"):
        predicciones = modelo.predict(df_entrada)
        df_resultado = df_entrada.copy()
        df_resultado["prediccion"] = predicciones

        st.dataframe(df_resultado)

        csv_resultado = df_resultado.to_csv(index=False).encode("utf-8")
        st.download_button("Descargar predicciones", csv_resultado, "predicciones.csv", "text/csv")
```

## Validar datos de entrada con Pandera antes de predecir

```python
import pandera as pa

schema_entrada = pa.DataFrameSchema({
    "dias_atras": pa.Column(int, pa.Check.in_range(1, 365)),
    "region": pa.Column(str, pa.Check.isin(["RD", "PR", "MX"])),
})

if archivo is not None:
    df_entrada = pd.read_csv(archivo)
    try:
        df_validado = schema_entrada.validate(df_entrada, lazy=True)
    except pa.errors.SchemaErrors as e:
        st.error("El archivo tiene datos inválidos:")
        st.dataframe(e.failure_cases)
        st.stop()   # ver 09 — detiene el resto del script si la validación falla

    predicciones = modelo.predict(df_validado)
```

Combinar Pandera (ver `Pandera/01 - Introducción y Conceptos Fundamentales.md`) con `st.error`/`st.stop` convierte errores de validación en mensajes claros y accionables para el usuario de la app, en vez de una excepción críptica de Python sin contexto.

## Explicabilidad — mostrar SHAP dentro de la app

```python
import shap
import matplotlib.pyplot as plt

@st.cache_resource
def crear_explainer(_modelo):   # el guión bajo evita que Streamlit intente hashear el modelo — ver 08
    return shap.TreeExplainer(_modelo)

explainer = crear_explainer(modelo)

if st.button("Explicar predicción"):
    shap_values = explainer.shap_values(df_entrada)
    fig, ax = plt.subplots()
    shap.summary_plot(shap_values, df_entrada, show=False)
    st.pyplot(fig)
```

Ver `Gradient Boosting/10 - Interpretabilidad e Importancia de Features.md` para el detalle de SHAP — mostrarlo interactivamente en Streamlit es una forma efectiva de comunicar "por qué el modelo predijo esto" a usuarios de negocio no técnicos.

## Comparar predicciones de dos modelos (champion vs. challenger)

```python
modelo_champion = cargar_modelo_mlflow_alias("champion")
modelo_challenger = cargar_modelo_mlflow_alias("challenger")

pred_champion = modelo_champion.predict(df_entrada)
pred_challenger = modelo_challenger.predict(df_entrada)

col1, col2 = st.columns(2)
col1.metric("Predicción — Champion", f"{pred_champion[0]:.1f}")
col2.metric("Predicción — Challenger", f"{pred_challenger[0]:.1f}", delta=f"{pred_challenger[0] - pred_champion[0]:.1f}")
```

Un dashboard de este tipo es útil como herramienta de revisión humana antes de promover formalmente un challenger a champion (ver `MLflow/07 - Model Registry.md`) — permite a un analista de negocio comparar visualmente el comportamiento de ambos modelos sobre casos concretos, no solo sobre métricas agregadas.

## Dashboard de monitoreo de experimentos — leer directamente desde MLflow

```python
import mlflow

@st.cache_data(ttl=300)   # refresca cada 5 minutos
def obtener_experimentos():
    mlflow.set_tracking_uri("http://mlflow-server:5000")
    return mlflow.search_runs(experiment_names=["claro-rd-demand-forecast"])

df_runs = obtener_experimentos()
st.dataframe(df_runs[["run_id", "metrics.mae", "metrics.rmse", "params.n_estimators"]])
st.line_chart(df_runs.set_index("start_time")["metrics.mae"])
```

Ver `MLflow/04 - Tracking - Búsqueda, Comparación y Organización.md` — construir un dashboard propio sobre `mlflow.search_runs` es común cuando se necesita una vista personalizada (con lógica de negocio específica) más allá de lo que ofrece la UI genérica de MLflow.

## Ver también

- [[08 - Caching - cache_data y cache_resource]]
- `MLflow/07 - Model Registry.md`
- `Pandera/01 - Introducción y Conceptos Fundamentales.md`
- `Gradient Boosting/10 - Interpretabilidad e Importancia de Features.md`
