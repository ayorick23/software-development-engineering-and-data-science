---
tags: [gradient-boosting, ngboost, interpretml, h2o, comparativa, cheat-sheet]
---

# 12 — Otras Librerías Similares y Cuándo Considerarlas

> Cierre del cheat-sheet. Responde directamente a "hay más librerías como estas" — panorama del resto de la familia y el veredicto final sustituye-vs-complementa.

## `HistGradientBoostingClassifier`/`Regressor` — el boosting "nativo" de scikit-learn

Ya cubierto en detalle en `Scikit-learn/07 - Árboles y Ensambles - Sintaxis y API.md`. Vale la mención aquí por contexto directo: es la respuesta de scikit-learn a la brecha de rendimiento frente a XGBoost/LightGBM, inspirada explícitamente en el diseño de LightGBM (histogram binning, soporte nativo de categóricas y valores faltantes).

```python
from sklearn.ensemble import HistGradientBoostingRegressor

modelo = HistGradientBoostingRegressor(max_iter=300, learning_rate=0.05, categorical_features="from_dtype")
```

**Cuándo preferirla sobre XGBoost/LightGBM/CatBoost**: cuando se quiere evitar una dependencia externa adicional (el proyecto ya depende de scikit-learn de todas formas) y el desempeño de las tres librerías especializadas no justifica el costo de una dependencia más — en benchmarks recientes, `HistGradientBoosting` ha cerrado bastante la brecha de velocidad, aunque las tres especializadas siguen ofreciendo más control fino de hiperparámetros y mejor ecosistema de tuning/interpretabilidad.

## NGBoost — Boosting Probabilístico

```bash
pip install ngboost
```

```python
from ngboost import NGBRegressor

modelo = NGBRegressor(n_estimators=300, learning_rate=0.05)
modelo.fit(X_train, y_train)

# La diferencia central: predice una DISTRIBUCIÓN completa, no un único valor puntual
distribucion_pred = modelo.pred_dist(X_test)
print(distribucion_pred.mean())    # equivalente a una predicción puntual estándar
print(distribucion_pred.std())     # incertidumbre asociada a CADA predicción individual
```

Ninguna de las tres librerías principales (XGBoost/LightGBM/CatBoost) produce nativamente una estimación de **incertidumbre** por predicción — solo un valor puntual. NGBoost aplica el mismo principio de boosting secuencial, pero optimiza los parámetros de una distribución de probabilidad completa (por ejemplo, media y varianza de una normal) en vez de un solo número. Relevante cuando el negocio necesita no solo "¿cuánto será la demanda?" sino "¿qué tan seguro está el modelo de esa predicción?" — por ejemplo, para dimensionar intervalos de confianza en decisiones de inventario.

## Explainable Boosting Machines (EBM) — InterpretML

```bash
pip install interpret
```

```python
from interpret.glassbox import ExplainableBoostingRegressor

modelo = ExplainableBoostingRegressor()
modelo.fit(X_train, y_train)

from interpret import show
show(modelo.explain_global())   # visualización interactiva del efecto de CADA feature, exacto y completo
```

Un enfoque radicalmente distinto: en vez de árboles con interacciones complejas entre features (difíciles de interpretar exactamente, por eso se recurre a SHAP como aproximación), EBM entrena un modelo aditivo generalizado (GAM) donde el efecto de cada feature se puede graficar **exactamente**, sin aproximaciones. Sacrifica algo de capacidad predictiva en problemas con interacciones fuertes entre features, a cambio de interpretabilidad total y exacta — relevante en contextos regulados (crédito, salud, seguros) donde una aproximación tipo SHAP no es suficiente y se requiere explicabilidad exacta y auditable.

## H2O — GBM dentro de una plataforma de AutoML más amplia

```bash
pip install h2o
```

```python
import h2o
from h2o.estimators import H2OGradientBoostingEstimator

h2o.init()
datos_h2o = h2o.H2OFrame(df_train)

modelo = H2OGradientBoostingEstimator(ntrees=300, max_depth=6, learn_rate=0.05)
modelo.train(x=columnas_features, y="target", training_frame=datos_h2o)
```

H2O no es solo una librería de boosting — es una plataforma completa de AutoML que incluye su propia implementación de GBM, Random Forest, GLM, y capacidades de AutoML (`H2OAutoML`) que prueban automáticamente múltiples algoritmos (incluyendo XGBoost internamente) y los ensamblan. Tiene su propio motor distribuido en Java bajo el capó, independiente de pandas/NumPy — relevante en organizaciones que ya usan H2O como plataforma de ML empresarial, menos común como elección greenfield para un proyecto nuevo en Python puro.

## LightGBM's `dart` y otros modos de boosting alternativos (mención rápida)

```python
modelo = LGBMRegressor(boosting_type="dart")   # Dropout meets Additive Regression Trees
```

`dart` aplica la idea de *dropout* (popular en redes neuronales) al boosting: en cada iteración, "apaga" aleatoriamente algunos árboles ya construidos al calcular el siguiente, reduciendo la tendencia de los últimos árboles a sobre-especializarse en corregir errores muy específicos de los árboles inmediatamente anteriores. No es una librería distinta, sino un modo alternativo dentro de LightGBM/XGBoost (ambas lo soportan) — vale mencionarlo porque cambia el comportamiento del boosting de forma no trivial sin cambiar de librería.

## Veredicto final: sustituye vs. complementa

| Herramienta | Relación con scikit-learn |
|---|---|
| XGBoost / LightGBM / CatBoost | **Complementan** — se insertan como un estimador más dentro del ecosistema sklearn (`Pipeline`, `GridSearchCV`) |
| `HistGradientBoosting*` | Es scikit-learn mismo — no hay nada que complementar, es la respuesta nativa a la misma necesidad |
| NGBoost | **Complementa** — API sklearn-compatible, añade una capacidad (incertidumbre) que ninguna de las anteriores tiene |
| EBM (InterpretML) | **Complementa** — API sklearn-compatible, prioriza interpretabilidad exacta sobre las ganancias marginales de precisión de un ensamble complejo |
| H2O | **Coexiste, parcialmente independiente** — tiene su propio ecosistema, aunque puede interoperar con Python/pandas |

La conclusión general de todo este cheat-sheet: **ninguna de estas librerías reemplaza a scikit-learn como marco de trabajo** — scikit-learn sigue proveyendo la infraestructura (`Pipeline`, `ColumnTransformer`, `GridSearchCV`, métricas, `train_test_split`) sobre la cual XGBoost, LightGBM, CatBoost, NGBoost y EBM se insertan como piezas intercambiables de modelado, cada una optimizando un eje distinto (velocidad, categóricas, interpretabilidad, incertidumbre) del mismo problema central de boosting sobre árboles.

## Ver también

- [[01 - Introducción y Panorama]]
- `Scikit-learn/07 - Árboles y Ensambles - Sintaxis y API.md`
- `Machine Learning/39-Interpretabilidad-de-Modelos.md`
- `Machine Learning/07-Librerias-de-Data-Science-y-ML.md`
