---
tags: [scikit-learn, machine-learning, persistencia, joblib, onnx, cheat-sheet]
---

# 13 — Persistencia de Modelos

> Continúa de [[03 - Pipelines y ColumnTransformer]]. Para el registro y versionado formal de modelos en producción, ver `MLflow/06 - Model Format y Flavors.md` y `MLflow/07 - Model Registry.md`.

## `joblib` — el estándar para modelos de scikit-learn

```python
import joblib

joblib.dump(pipeline_completo, "modelo_v3.joblib")

modelo_cargado = joblib.load("modelo_v3.joblib")
predicciones = modelo_cargado.predict(X_nuevo)
```

`joblib` es preferible a `pickle` estándar para objetos de scikit-learn porque maneja de forma más eficiente los arrays de NumPy grandes que suelen vivir dentro de los modelos entrenados (por ejemplo, los árboles de un `RandomForest` con cientos de estimadores).

### Compresión

```python
joblib.dump(pipeline_completo, "modelo_v3.joblib.gz", compress=3)   # nivel 0 (sin compresión) a 9 (máxima)
```

Compresión reduce el tamaño en disco a costa de tiempo de carga/guardado — relevante cuando el modelo se distribuye por red o se versiona en un repositorio con límites de tamaño de archivo.

## `pickle` — alternativa nativa de Python

```python
import pickle

with open("modelo_v3.pkl", "wb") as f:
    pickle.dump(pipeline_completo, f)

with open("modelo_v3.pkl", "rb") as f:
    modelo_cargado = pickle.load(f)
```

Funciona igual de bien para la mayoría de los casos, pero `joblib` suele ser más eficiente específicamente para objetos con muchos arrays de NumPy internos — en la práctica, la comunidad de scikit-learn usa `joblib` como convención por defecto.

## El riesgo central de `pickle`/`joblib`: compatibilidad de versiones

> **Advertencia crítica de producción**: un modelo serializado con `pickle`/`joblib` **no es portable entre versiones distintas de scikit-learn** (ni siquiera siempre entre versiones distintas de Python) — cargar un modelo entrenado con scikit-learn 1.2 en un entorno con scikit-learn 1.5 puede fallar silenciosamente, producir resultados distintos, o lanzar una excepción directa. Es indispensable fijar la versión exacta de `scikit-learn` (y `numpy`, `scipy`) en el entorno donde se despliega el modelo, idéntica a la del entorno donde se entrenó.

```python
import sklearn
print(sklearn.__version__)   # registrar SIEMPRE esta versión junto al modelo serializado
```

```txt
# requirements.txt del entorno de despliegue — versiones EXACTAS, no rangos
scikit-learn==1.5.1
numpy==1.26.4
scipy==1.13.0
```

Este es exactamente el problema que `MLflow Models` resuelve de forma sistemática — el archivo `MLmodel` guarda automáticamente el entorno completo (`conda.yaml`/`requirements.txt`) junto al modelo, en vez de depender de que el humano recuerde fijar versiones manualmente. Ver `MLflow/06 - Model Format y Flavors.md`.

## Guardar un Pipeline completo, no solo el modelo final

```python
# CORRECTO: serializa preprocesamiento + modelo como una sola unidad
pipeline_completo = Pipeline([
    ("preprocesamiento", column_transformer),
    ("modelo", RandomForestRegressor()),
])
pipeline_completo.fit(X_train, y_train)
joblib.dump(pipeline_completo, "pipeline_completo.joblib")

# En producción, basta con:
pipeline_cargado = joblib.load("pipeline_completo.joblib")
predicciones = pipeline_cargado.predict(datos_crudos_nuevos)   # sin necesidad de re-implementar el preprocesamiento
```

Serializar solo el modelo final (sin el preprocesamiento) obliga a reimplementar manualmente la lógica de escalado/encoding en el sistema que consume el modelo — una fuente frecuente de bugs sutiles cuando esa lógica se desincroniza del pipeline de entrenamiento original. Guardar el `Pipeline` completo elimina ese riesgo por diseño.

## Exportar a ONNX — para inferencia fuera del ecosistema Python

```bash
pip install skl2onnx onnxruntime
```

```python
from skl2onnx import to_onnx
import numpy as np

modelo_onnx = to_onnx(pipeline_completo, X_train.to_numpy().astype(np.float32))

with open("modelo.onnx", "wb") as f:
    f.write(modelo_onnx.SerializeToString())
```

```python
# Inferencia con onnxruntime, sin necesidad de scikit-learn instalado:
import onnxruntime as rt

sesion = rt.InferenceSession("modelo.onnx")
resultado = sesion.run(None, {"X": X_test.to_numpy().astype(np.float32)})
```

ONNX (Open Neural Network Exchange) resuelve dos problemas que `joblib`/`pickle` no resuelven: portabilidad entre lenguajes (el modelo se puede servir desde C++, Java, C#, no solo Python) y desacople de la versión exacta de scikit-learn en producción — el runtime ONNX no depende de tener instalada la misma versión de la librería que se usó para entrenar. El costo es que no todos los estimadores/transformadores de scikit-learn tienen soporte completo de conversión a ONNX; conviene validar la conversión (comparar predicciones antes/después) antes de confiar en ella para producción.

## `skops` — alternativa más segura que pickle para compartir modelos

```bash
pip install skops
```

```python
import skops.io as sio

sio.dump(pipeline_completo, "modelo.skops")

# Cargar requiere aprobar explícitamente qué tipos de objeto se permiten deserializar:
modelo_cargado = sio.load("modelo.skops", trusted=sio.get_untrusted_types(file="modelo.skops"))
```

> **Riesgo de seguridad de `pickle`/`joblib`**: deserializar un archivo `.pkl`/`.joblib` de origen no confiable puede ejecutar código arbitrario — el formato de pickle permite incrustar instrucciones que se ejecutan al cargar el archivo. `skops` fue diseñado específicamente para evitar este riesgo (requiere aprobación explícita de qué tipos se deserializan), y es la opción recomendada cuando el modelo se comparte públicamente (ej. subido a un repositorio de modelos como Hugging Face Hub) en vez de moverse solo dentro de infraestructura propia y confiable.

## Verificar reproducibilidad tras cargar

```python
# Buena práctica antes de desplegar: verificar que el modelo cargado da EXACTAMENTE los mismos resultados
predicciones_originales = pipeline_completo.predict(X_test)
joblib.dump(pipeline_completo, "temp.joblib")
predicciones_cargadas = joblib.load("temp.joblib").predict(X_test)

import numpy as np
assert np.allclose(predicciones_originales, predicciones_cargadas)
```

## Ver también

- [[03 - Pipelines y ColumnTransformer]]
- `MLflow/06 - Model Format y Flavors.md`
- `MLflow/07 - Model Registry.md`
- [[16 - Buenas Prácticas, Errores Comunes y Rendimiento]]
