---
tags: [gradient-boosting, xgboost, lightgbm, catboost, gpu, dask, cheat-sheet]
---

# 09 — GPU, Entrenamiento Distribuido y Rendimiento

> Continúa de [[08 - Comparativa Técnica y Tuning Cruzado]].

## Soporte de GPU — sintaxis por librería

### XGBoost

```python
modelo = XGBRegressor(
    tree_method="hist",
    device="cuda",   # versiones recientes (2.x); versiones anteriores usaban tree_method="gpu_hist"
)
```

### LightGBM

```python
modelo = LGBMRegressor(
    device="gpu",   # requiere una build de LightGBM compilada con soporte GPU
    gpu_platform_id=0,
    gpu_device_id=0,
)
```

> LightGBM con GPU requiere frecuentemente compilar la librería desde código fuente con las flags de OpenCL/CUDA correctas — a diferencia de XGBoost/CatBoost, el soporte GPU vía `pip install lightgbm` estándar puede no incluir binarios GPU-ready, dependiendo del sistema operativo. Verificar la documentación oficial vigente antes de asumir que `pip install` es suficiente.

### CatBoost

```python
modelo = CatBoostRegressor(
    task_type="GPU",
    devices="0",   # índice de GPU específica, o "0:1" para múltiples GPUs
)
```

CatBoost tiene, en general, la reputación de la implementación GPU más madura y fácil de activar entre las tres (funciona "out of the box" en la instalación estándar vía pip en la mayoría de entornos con CUDA disponible).

## Cuándo GPU realmente acelera el entrenamiento

GPU aporta el mayor beneficio en:
- Datasets con **muchas filas** (cientos de miles a millones) — el paralelismo de GPU se aprovecha mejor con mucho volumen de datos.
- Muchas iteraciones/árboles (`n_estimators`/`iterations` altos).
- Búsquedas de hiperparámetros con muchos trials, donde el tiempo total ahorrado por trial se multiplica.

Para datasets pequeños-medianos (decenas de miles de filas), el overhead de transferir datos a la GPU puede hacer que el entrenamiento en GPU sea **más lento** que en CPU con varios cores — vale medir empíricamente en el dataset específico antes de asumir que GPU siempre es mejor.

## Entrenamiento distribuido — múltiples máquinas

### XGBoost + Dask

```python
import dask.dataframe as dd
from xgboost import dask as dxgb
from dask.distributed import Client

client = Client()   # conecta a un cluster de Dask (local o remoto)

X_dask = dd.from_pandas(X_train, npartitions=8)
y_dask = dd.from_pandas(y_train, npartitions=8)

dtrain = dxgb.DaskDMatrix(client, X_dask, y_dask)
resultado = dxgb.train(client, params, dtrain, num_boost_round=300)
```

Permite entrenar sobre datasets que no caben en la memoria de una sola máquina, distribuyendo tanto los datos como el cómputo entre varios workers de Dask — ver también `Machine Learning/50-Orquestacion-Prefect-y-Airflow.md` para el contexto de cómputo distribuido en pipelines de ML.

### LightGBM — entrenamiento distribuido nativo

```python
params = {
    "num_leaves": 31,
    "tree_learner": "data",   # "data", "feature", "voting" — estrategias de paralelización distribuida
}
```

LightGBM incluye soporte de entrenamiento distribuido nativo (vía MPI o sockets) sin depender obligatoriamente de Dask, con tres estrategias distintas de paralelización según si el cuello de botella es el volumen de filas, de columnas, o la comunicación entre nodos.

### CatBoost — entrenamiento distribuido

CatBoost soporta entrenamiento multi-GPU nativamente (`devices="0:1:2:3"`), pero el soporte de entrenamiento distribuido **multi-máquina** (no solo multi-GPU en una máquina) es más limitado que XGBoost/LightGBM en el ecosistema open-source estándar — para escalado horizontal real entre máquinas, XGBoost (vía Dask o Spark) y LightGBM (vía su soporte nativo) suelen ser las opciones más directas.

## Integración con Spark

```python
# XGBoost tiene un paquete dedicado para PySpark: xgboost.spark
from xgboost.spark import SparkXGBRegressor

modelo_spark = SparkXGBRegressor(
    features_col="features", label_col="demanda",
    num_workers=4,
)
modelo_spark_entrenado = modelo_spark.fit(spark_df_train)
```

Relevante en entornos donde los datos ya viven en un cluster de Spark (Databricks, EMR) — evita tener que extraer todo a un DataFrame de pandas local antes de entrenar. Ver `Machine Learning/07-Librerias-de-Data-Science-y-ML.md` para contexto de Spark en el ecosistema general.

## Benchmarks conceptuales — qué esperar, sin números que se desactualizan

En vez de citar números específicos de benchmarks (que cambian con cada versión de cada librería y dependen fuertemente del hardware/dataset), la guía conceptual estable es:

| Dimensión | Librería más favorecida (regla general) |
|---|---|
| Velocidad de entrenamiento CPU, datasets grandes | LightGBM |
| Uso de memoria, datasets grandes | LightGBM |
| Velocidad de inferencia (predict) | CatBoost (árboles simétricos, muy vectorizables) |
| Facilidad de activar GPU | CatBoost |
| Ecosistema y madurez de integraciones (MLOps, cloud) | XGBoost |

Cualquier decisión de producción con impacto real en costos de infraestructura debería validarse con un benchmark propio sobre el dataset y hardware reales del proyecto, no solo con esta guía general.

## Ver también

- [[08 - Comparativa Técnica y Tuning Cruzado]]
- `Machine Learning/50-Orquestacion-Prefect-y-Airflow.md`
- `Docker/Orchestration and Scalability.md`
