---
tags: [pandera, testing, integraciones, fastapi, polars, great-expectations, cheat-sheet]
---

# 09 — Integración con el Ecosistema

> Consolida referencias de todo el cheat-sheet.

## Más allá de pandas — soporte multi-motor de DataFrame

Desde versiones recientes, Pandera no está limitado a pandas — soporta validar el mismo tipo de esquema sobre otros motores de DataFrame, relevante cuando un proyecto migra a datasets más grandes.

### Polars

```bash
pip install "pandera[polars]"
```

```python
import pandera.polars as pa
import polars as pl

schema = pa.DataFrameSchema({
    "office_id": pa.Column(int, pa.Check.greater_than(0)),
    "total_demand": pa.Column(float, pa.Check.greater_than_or_equal_to(0)),
})

df_polars = pl.DataFrame({"office_id": [1, 2, 3], "total_demand": [10.0, 20.0, 30.0]})
schema.validate(df_polars)
```

La API es prácticamente idéntica a la versión de pandas (`pandera.polars` en vez de `pandera`) — el conocimiento de esquemas, checks y validación perezosa cubierto en el resto de este cheat-sheet se transfiere directamente, solo cambia el motor de ejecución subyacente.

### Dask

```bash
pip install "pandera[dask]"
```

```python
import pandera.dask as pa
import dask.dataframe as dd

schema = pa.DataFrameSchema({"office_id": pa.Column(int, pa.Check.greater_than(0))})

df_dask = dd.from_pandas(df, npartitions=4)
df_validado = schema.validate(df_dask)   # la validación se distribuye entre particiones
```

Relevante para datasets que no caben en memoria de una sola máquina — ver también `Gradient Boosting/09 - GPU, Entrenamiento Distribuido y Rendimiento.md` para más contexto de Dask en pipelines de ML distribuidos.

### PySpark (pyspark.pandas)

```bash
pip install "pandera[pyspark]"
```

```python
import pandera.pyspark as pa
```

Mismo principio — validar DataFrames que viven en un cluster de Spark sin tener que extraerlos primero a pandas local, relevante en entornos Databricks/EMR donde los datos ya residen distribuidos.

## FastAPI — validar requests/responses con esquemas de Pandera

```python
from fastapi import FastAPI
from pandera.typing import DataFrame, Series
import pandera as pa

class PrediccionInput(pa.DataFrameModel):
    office_id: Series[int] = pa.Field(gt=0)
    dias_atras: Series[int] = pa.Field(ge=1, le=365)

app = FastAPI()

@app.post("/predecir")
def predecir(datos: list[dict]):
    import pandas as pd
    df = pd.DataFrame(datos)
    df_validado = PrediccionInput.validate(df)   # SchemaError se traduce en un 422 si se maneja con un exception handler
    predicciones = modelo.predict(df_validado)
    return {"predicciones": predicciones.tolist()}
```

```python
# Exception handler para traducir SchemaError a una respuesta HTTP clara:
from fastapi import Request
from fastapi.responses import JSONResponse
import pandera.errors as pa_errors

@app.exception_handler(pa_errors.SchemaError)
def manejar_error_validacion(request: Request, exc: pa_errors.SchemaError):
    return JSONResponse(status_code=422, content={"error": "Datos de entrada inválidos", "detalle": str(exc)})
```

Mientras Pydantic (usado nativamente por FastAPI) valida la estructura de un JSON individual, Pandera valida específicamente la estructura **tabular** — la combinación es natural cuando un endpoint recibe una lista de registros que conceptualmente forman un DataFrame. Ver `Machine Learning/49-APIs-con-FastAPI-para-Servir-Modelos.md`.

## pytest — fixtures reutilizables con esquemas

```python
import pytest

@pytest.fixture
def df_demanda_valido():
    return schema.example(size=10)   # ver 08 - Generación de Datos Sintéticos con Hypothesis

def test_calcular_demanda_por_hora(df_demanda_valido):
    resultado = calcular_demanda_por_hora(df_demanda_valido)
    assert "demanda_por_hora" in resultado.columns

def test_rechaza_datos_invalidos():
    df_malo = pd.DataFrame({"office_id": [-1], "total_demand": [10.0]})
    with pytest.raises(pa.errors.SchemaError):
        schema.validate(df_malo)
```

Ver `Machine Learning/13-Testing-en-Machine-Learning.md` y `Machine Learning/25-Testing-Avanzado.md` para el contexto general de testing en proyectos de ML.

## MLflow — validar datos antes de loguear un run

```python
import mlflow

def entrenar_con_validacion(df):
    df_validado = schema_entrenamiento.validate(df, lazy=True)   # falla ANTES de gastar cómputo entrenando

    with mlflow.start_run():
        mlflow.log_param("filas_entrenamiento", len(df_validado))
        modelo = entrenar(df_validado)
        ...
```

Validar con Pandera **antes** de iniciar un run de MLflow evita registrar experimentos "contaminados" por datos corruptos — el pipeline falla temprano, sin gastar tiempo de cómputo entrenando sobre datos inválidos que de todas formas producirían un modelo descartable.

## DVC Pipelines — Pandera como una etapa explícita

```yaml
# dvc.yaml
stages:
  validar_datos:
    cmd: python scripts/validar_datos.py
    deps:
      - scripts/validar_datos.py
      - data/raw/demanda_historica.csv
    outs:
      - data/validated/demanda_historica.parquet
```

```python
# scripts/validar_datos.py
df = pd.read_csv("data/raw/demanda_historica.csv")
df_validado = schema.validate(df, lazy=True)
df_validado.to_parquet("data/validated/demanda_historica.parquet")
```

Insertar la validación como su propia etapa de `dvc.yaml` (ver `DVC/04 - DVC Pipelines - dvc.yaml y Reproducibilidad.md`) hace que **falle explícitamente el pipeline reproducible** si los datos de origen no cumplen el esquema, en vez de que un `dvc repro` "exitoso" oculte que los datos de entrada estaban corruptos.

## Comparativa con Great Expectations

| | Pandera | Great Expectations |
|---|---|---|
| Filosofía | Esquemas como código Python (o clases estilo Pydantic) | "Expectation Suites" declarativas, con UI y almacenamiento propio de resultados |
| Integración con type hints | Nativa (`DataFrameModel`, `pandera.typing`) | No es el foco principal |
| Curva de aprendizaje | Baja si ya se conoce Pydantic/pandas | Más alta — concepto propio de "Data Context", "Checkpoints" |
| Generación de reportes/documentación | Básica (`to_yaml`) | Muy completa — genera documentación HTML navegable de las expectativas |
| Validación de funciones directamente (`@check_io`) | Sí, nativo | No — se enfoca en datasets, no en funciones |
| Property-based testing integrado | Sí (`hypothesis`, ver [[08 - Generación de Datos Sintéticos con Hypothesis]]) | No de forma nativa |

Pandera tiende a preferirse cuando la validación se quiere **integrada al código Python** de forma ligera (decoradores, type hints); Great Expectations cuando se necesita una capa de gobernanza de datos más formal, con documentación autogenerada y una UI para que perfiles no técnicos revisen las reglas de calidad de datos vigentes.

## Ver también

- `Machine Learning/49-APIs-con-FastAPI-para-Servir-Modelos.md`
- `DVC/04 - DVC Pipelines - dvc.yaml y Reproducibilidad.md`
- `Machine Learning/13-Testing-en-Machine-Learning.md`
- [[08 - Generación de Datos Sintéticos con Hypothesis]]
