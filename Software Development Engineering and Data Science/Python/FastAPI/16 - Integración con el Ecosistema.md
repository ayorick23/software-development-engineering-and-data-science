---
tags: [fastapi, apis, mlops, python, cheat-sheet]
---

# 16 — Integración con el Ecosistema

> Cierra la serie de FastAPI, continuando de [[15 - Despliegue y Producción]]. Para el caso de uso central que motiva la mayoría de estas integraciones — servir un modelo de ML — ver [[49-APIs-con-FastAPI-para-Servir-Modelos]], que asume todo lo cubierto en esta carpeta.

## MLflow — servir un modelo versionado desde el Model Registry

```python
import mlflow.pyfunc
from contextlib import asynccontextmanager

modelo = None

@asynccontextmanager
async def lifespan(app: FastAPI):
    global modelo
    modelo = mlflow.pyfunc.load_model(model_uri="models:/mi-modelo/Production")
    yield

app = FastAPI(lifespan=lifespan)

@app.post("/predecir")
def predecir(datos: DatosEntrada):
    prediccion = modelo.predict([datos.model_dump()])
    return {"prediccion": prediccion[0]}
```

El patrón `lifespan` de [[14 - Routers y Estructura de Proyectos Grandes]] es exactamente lo que evita recargar el modelo en cada request. Ver `MLflow/09 - Model Serving y Despliegue.md` para el lado de MLflow — cómo se promueve un modelo a `Production` en el registry antes de que este endpoint lo cargue.

## Pandera — validar DataFrames que entran o salen de un endpoint

```python
import pandera as pa
import pandas as pd

esquema_entrada = pa.DataFrameSchema({
    "demand_lag_1": pa.Column(float, checks=pa.Check.ge(0)),
    "es_feriado": pa.Column(int, checks=pa.Check.isin([0, 1])),
})

@app.post("/predecir-batch")
def predecir_batch(registros: list[dict]):
    df = pd.DataFrame(registros)
    esquema_entrada.validate(df)   # lanza pandera.errors.SchemaError si algo no cumple
    predicciones = modelo.predict(df)
    return {"predicciones": predicciones.tolist()}
```

Mientras Pydantic (ver [[03 - Request Body y Modelos Pydantic]]) valida el JSON fila por fila en el borde de la API, Pandera valida el **DataFrame completo** una vez ensamblado — útil en endpoints batch donde la entrada natural es tabular, no un objeto por request. Ver `Pandera/01 - Introducción y Conceptos Fundamentales.md`.

## Streamlit como frontend, FastAPI como backend

```python
# lado Streamlit (app.py)
import streamlit as st
import httpx

datos = st.text_input("Ingresa datos para predicción")
if st.button("Predecir"):
    respuesta = httpx.post("http://localhost:8000/predecir", json={"texto": datos})
    st.write(respuesta.json())
```

Patrón común en proyectos de datos: Streamlit da una interfaz rápida para usuarios no técnicos (ver `Streamlit/01 - Introducción y Modelo de Ejecución.md`), mientras FastAPI expone la lógica real como una API separada — permite que otros clientes (un frontend real, un job batch, otro servicio) consuman el mismo backend sin pasar por Streamlit.

## Docker Compose — orquestando API + base de datos + frontend juntos

```yaml
services:
  api:
    build: ./api
    ports: ["8000:8000"]
    environment:
      - DATABASE_URL=postgresql://usuario:pass@db:5432/mibd
    depends_on: [db]

  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mibd
    volumes: ["db_data:/var/lib/postgresql/data"]

  frontend:
    build: ./streamlit_app
    ports: ["8501:8501"]
    depends_on: [api]

volumes:
  db_data:
```

Extiende el `Dockerfile` de un solo servicio visto en [[15 - Despliegue y Producción]] a un stack completo — la API se conecta a la base de datos por el nombre del servicio (`db`), no por `localhost`, siguiendo la resolución de DNS interna de Docker Compose.

## DVC — servir un modelo cuyos datos/pipeline están versionados

Cuando el modelo cargado en el endpoint `/predecir` proviene de un pipeline reproducible con DVC (ver `DVC/01 - Introducción y Conceptos Fundamentales.md`), es común incluir en la respuesta metadata de trazabilidad:

```python
@app.get("/info-modelo")
def info_modelo():
    return {
        "version_modelo": "1.4.2",
        "commit_dvc": "a3f9c21",       # commit del pipeline que produjo este modelo
        "fecha_entrenamiento": "2026-08-01",
    }
```

Permite a cualquier consumidor de la API saber exactamente qué versión de datos/código generó las predicciones que está recibiendo — trazabilidad completa desde el request HTTP hasta el commit de Git que entrenó el modelo.

## Ver también

- [[49-APIs-con-FastAPI-para-Servir-Modelos]]
- [[15 - Despliegue y Producción]]
- `MLflow/09 - Model Serving y Despliegue.md`
- `Pandera/01 - Introducción y Conceptos Fundamentales.md`
- `Streamlit/01 - Introducción y Modelo de Ejecución.md`
