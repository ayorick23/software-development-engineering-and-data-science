---
tags: [fastapi, apis, mlops, deployment]
---

# 49 — APIs con FastAPI para Servir Modelos de Machine Learning

> Nota del mentor: tu pipeline actual es batch, así que no necesitas esto hoy para Claro RD — pero es la herramienta que vas a necesitar en el momento en que cualquier proyecto (tuyo o de un compañero) requiera predicciones en tiempo real, y es tan fundamental en el ecosistema Python moderno que vale la pena dominarla ahora, con calma, antes de necesitarla bajo presión.

## 1. Por qué FastAPI se volvió el estándar para servir modelos de ML

- **Basado en type hints de Python** (ver [[20-Python-Avanzado-Sistema-de-Tipos]]) — la validación de datos de entrada/salida se deriva automáticamente de tus anotaciones de tipo, sin escribir validación manual repetitiva.
- **Documentación automática (OpenAPI/Swagger)** generada desde el código, sin mantenerla a mano por separado — exactamente la misma filosofía de "la documentación vive en el código" que viste en [[33-Documentacion-Profesional]] con MkDocs.
- **Rendimiento asíncrono nativo** (`async`/`await`), competitivo con frameworks de otros lenguajes tradicionalmente más rápidos que Python puro.

## 2. Servicio mínimo para servir un modelo — anatomía completa

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
import mlflow.pyfunc

app = FastAPI(title="Claro RD - Servicio de Predicción de Demanda")

modelo = mlflow.pyfunc.load_model(model_uri="models:/claro-rd-demand-model/Production")

class SolicitudPrediccion(BaseModel):
    office_id: int
    demand_lag_1: float
    demand_lag_336: float
    es_feriado: int = Field(ge=0, le=1)

class RespuestaPrediccion(BaseModel):
    office_id: int
    demanda_predicha: float
    version_modelo: str

@app.post("/predecir", response_model=RespuestaPrediccion)
def predecir(solicitud: SolicitudPrediccion) -> RespuestaPrediccion:
    try:
        features = [[solicitud.demand_lag_1, solicitud.demand_lag_336, solicitud.es_feriado]]
        prediccion = modelo.predict(features)[0]
        return RespuestaPrediccion(
            office_id=solicitud.office_id,
            demanda_predicha=float(prediccion),
            version_modelo="1.4.2",
        )
    except Exception:
        raise HTTPException(status_code=500, detail="Error al generar la predicción")

@app.get("/health")
def health_check():
    return {"status": "ok"}
```

**Pydantic** (la librería detrás de `BaseModel`) hace el trabajo pesado: si alguien envía `office_id` como un string en vez de un entero, o `es_feriado=5` (fuera del rango 0-1 declarado con `Field(ge=0, le=1)`), FastAPI rechaza la solicitud automáticamente con un error 422 detallado, **antes** de que ese dato malformado llegue siquiera a tu lógica de predicción — validación de entrada gratis, derivada directamente de tus type hints.

## 3. El endpoint `/health` — no es opcional en producción

Cualquier orquestador (Kubernetes, Azure Container Apps, un balanceador de carga) necesita una forma de verificar si el servicio está realmente sano y listo para recibir tráfico. Un endpoint `/health` simple que responde rápido es lo mínimo; en servicios más maduros, se distingue entre:

```python
@app.get("/health/liveness")
def liveness():
    """¿El proceso está vivo? Si falla, reiniciar el contenedor."""
    return {"status": "alive"}

@app.get("/health/readiness")
def readiness():
    """¿Está listo para recibir tráfico? (ej. el modelo ya cargó en memoria)"""
    if modelo is None:
        raise HTTPException(status_code=503, detail="Modelo aún no cargado")
    return {"status": "ready"}
```

## 4. Logging y manejo de errores — retomando lo ya aprendido

```python
import logging
logger = logging.getLogger(__name__)

@app.post("/predecir", response_model=RespuestaPrediccion)
def predecir(solicitud: SolicitudPrediccion) -> RespuestaPrediccion:
    logger.info("Solicitud de predicción recibida", extra={"extra_fields": {"office_id": solicitud.office_id}})
    try:
        prediccion = modelo.predict([[solicitud.demand_lag_1, solicitud.demand_lag_336, solicitud.es_feriado]])[0]
    except Exception:
        logger.exception(f"Error prediciendo para oficina {solicitud.office_id}")
        raise HTTPException(status_code=500, detail="Error interno al generar la predicción")

    logger.info("Predicción generada", extra={"extra_fields": {"office_id": solicitud.office_id, "prediccion": float(prediccion)}})
    return RespuestaPrediccion(office_id=solicitud.office_id, demanda_predicha=float(prediccion), version_modelo="1.4.2")
```

Este servicio aplica directamente [[11-Logging-en-Python-para-ML]] y [[23-Manejo-Profesional-de-Errores]] — cada solicitud queda trazada, y cualquier fallo se loguea con el traceback completo (`logger.exception`) sin exponer detalles internos al cliente que hizo la solicitud (el mensaje HTTP es genérico, el detalle completo va solo al log).

## 5. Middleware — lógica transversal a todas las solicitudes

```python
import time
from fastapi import Request

@app.middleware("http")
async def medir_latencia(request: Request, call_next):
    inicio = time.perf_counter()
    respuesta = await call_next(request)
    duracion = time.perf_counter() - inicio
    logger.info(f"{request.method} {request.url.path} — {duracion*1000:.1f}ms")
    return respuesta
```

Un middleware envuelve **todas** las solicitudes sin tener que repetir el mismo código en cada endpoint — el mismo principio de un decorator (ver [[21-Python-Avanzado-Ejecucion-y-Metaprogramacion]]), aplicado a nivel de toda la aplicación en vez de a una sola función. Común para medir latencia, autenticación, o logging de solicitudes.

## 6. Servir el modelo eficientemente — cargar una vez, no en cada solicitud

```python
# INCORRECTO: recarga el modelo desde MLflow en CADA solicitud — latencia altísima
@app.post("/predecir")
def predecir(solicitud):
    modelo = mlflow.pyfunc.load_model(...)  # ¡esto adentro del endpoint es un error grave!
    ...

# CORRECTO: se carga UNA VEZ al iniciar el proceso, se reutiliza en cada solicitud
modelo = mlflow.pyfunc.load_model(model_uri="models:/claro-rd-demand-model/Production")  # a nivel de módulo

@app.post("/predecir")
def predecir(solicitud):
    return modelo.predict(...)  # reutiliza el objeto ya cargado en memoria
```

Cargar un modelo de ML puede tardar segundos — hacerlo en cada solicitud convierte un servicio que debería responder en milisegundos en uno que responde en segundos, un error de rendimiento sorprendentemente común en implementaciones apresuradas.

## 7. Dockerizar el servicio — conectando con lo ya aprendido

```dockerfile
FROM python:3.11-slim
COPY pyproject.toml .
RUN pip install .
COPY src/ src/
CMD ["uvicorn", "forecasting_pipeline.api:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

`--workers 4` levanta múltiples procesos worker de Uvicorn (el servidor ASGI que ejecuta FastAPI), permitiendo atender solicitudes concurrentes en paralelo — el número de workers se ajusta típicamente según los núcleos de CPU disponibles del contenedor, siguiendo directamente el patrón de Docker de [[47-Docker-en-Profundidad-para-ML]].

## Ver también
- [[17-Arquitecturas-de-Despliegue-de-Modelos]]
- [[47-Docker-en-Profundidad-para-ML]]
- [[11-Logging-en-Python-para-ML]]
- [[15-MLflow-en-Profundidad]]
- [[01 - Introducción y Arquitectura]] — cheat-sheet dedicado de FastAPI, profundiza en routing, Pydantic, DI, seguridad, testing y despliegue más allá de servir modelos
