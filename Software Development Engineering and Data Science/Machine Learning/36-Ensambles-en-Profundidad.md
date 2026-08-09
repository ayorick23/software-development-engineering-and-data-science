---
tags: [machine-learning, ensambles, random-forest, gradient-boosting, xgboost]
---

# 36 — Ensambles en Profundidad: Cómo Funcionan Internamente Random Forest, Gradient Boosting y XGBoost

> Nota del mentor: has usado `XGBRegressor()` y `GradientBoostingRegressor()` en tu proyecto de Claro RD por meses. Esta nota es donde dejas de tratarlos como cajas negras y entiendes exactamente qué pasa dentro cuando llamas `.fit()` — incluyendo el mecanismo exacto detrás del bug de `warm_start` que descubriste.

## 1. Bagging vs. Boosting — las dos familias fundamentales, filosofías opuestas

- **Bagging (Bootstrap Aggregating)**: entrena muchos modelos **en paralelo e independientes**, cada uno sobre una muestra aleatoria distinta de los datos (con reemplazo), y promedia sus predicciones. Reduce **varianza**. Random Forest es la implementación más famosa.
- **Boosting**: entrena modelos **en secuencia**, donde cada modelo nuevo se enfoca en corregir los errores del conjunto anterior. Reduce principalmente **sesgo** (bias), aunque bien regularizado también controla varianza. Gradient Boosting y XGBoost pertenecen a esta familia.

## 2. Random Forest — cómo funciona internamente, paso a paso

```
Para cada uno de N árboles (ej. N=300):
  1. Toma una muestra bootstrap de los datos (muestreo aleatorio CON reemplazo,
     mismo tamaño que el set original — algunas filas se repiten, otras quedan fuera)
  2. En cada nodo del árbol, considera solo un subconjunto aleatorio de features
     (no todos) — típicamente sqrt(n_features) para clasificación
  3. Crece el árbol casi sin restricción (max_depth alto)
Predicción final = promedio de las predicciones de los N árboles (regresión)
                    o voto mayoritario (clasificación)
```

```python
from sklearn.ensemble import RandomForestRegressor

modelo = RandomForestRegressor(
    n_estimators=300,      # cuántos árboles — más árboles = más estable, rendimientos decrecientes
    max_features="sqrt",    # features considerados por corte — la clave de la diversidad entre árboles
    max_depth=None,          # árboles profundos está bien — el overfitting se cancela al promediar
    n_jobs=-1,                # los árboles son independientes → paralelizable trivialmente
)
```

**¿Por qué funciona promediar árboles individualmente sobreajustados?** Cada árbol individual, entrenado sobre una muestra distinta con un subconjunto distinto de features, memoriza un ruido *distinto*. Al promediar, el ruido específico de cada árbol se cancela parcialmente (no está correlacionado entre árboles), mientras que la señal real (el patrón verdadero en los datos) se refuerza porque todos los árboles la capturan de forma similar. Esta es la razón matemática exacta por la que Random Forest tolera árboles individuales profundos sin overfitting severo del conjunto — algo que sería un desastre en un solo árbol.

## 3. Gradient Boosting — corrección secuencial de residuos

```
Modelo inicial: predicción_0 = promedio de y (la predicción más simple posible)
Para cada iteración m = 1, 2, ..., M:
  1. Calcula el residuo: r_m = y_real - predicción_(m-1)
  2. Entrena un árbol NUEVO y poco profundo para predecir ese residuo (no y directamente)
  3. predicción_m = predicción_(m-1) + learning_rate × predicción_del_árbol_nuevo
```

```python
from sklearn.ensemble import GradientBoostingRegressor

modelo = GradientBoostingRegressor(
    n_estimators=300,
    max_depth=3,            # árboles POCO profundos — cada uno corrige un error específico
    learning_rate=0.05,      # cuánto confía en cada corrección — el hiperparámetro más importante
)
```

**El rol crítico de `learning_rate`**: cada árbol nuevo no reemplaza al ensamble anterior, lo **ajusta ligeramente**. Un `learning_rate` alto (ej. 1.0) hace correcciones agresivas — converge rápido pero es propenso a overfitting. Un `learning_rate` bajo (ej. 0.01-0.05) hace correcciones suaves — necesita más árboles (`n_estimators` más alto) pero generaliza mejor. Existe un trade-off directo entre `learning_rate` y `n_estimators`: bajar uno normalmente implica subir el otro para compensar.

### El bug de `warm_start` que descubriste, explicado desde este mecanismo interno

`warm_start=True` le dice al modelo "no empieces desde cero, continúa agregando árboles a los que ya existen". El bug real: **instanciar un objeto nuevo con `warm_start=True` no tiene ningún efecto**, porque no hay árboles previos que continuar — el parámetro solo cobra sentido cuando llamas `.fit()` una **segunda vez sobre el mismo objeto ya entrenado**, incrementando `n_estimators` y dejando que continúe la secuencia de corrección de residuos desde donde se quedó. Por eso la solución correcta fue `copy.deepcopy()` del champion ya entrenado y volver a llamar `.fit()` sobre esa copia — solo así el mecanismo secuencial de boosting realmente continúa desde los residuos ya parcialmente corregidos, en vez de reiniciar desde cero.

## 4. XGBoost — Gradient Boosting con optimizaciones matemáticas y de ingeniería

XGBoost implementa el mismo principio de boosting secuencial, pero con mejoras significativas:

- **Aproximación de segundo orden (Newton boosting)**: usa tanto el gradiente como el Hessiano (segunda derivada) de la función de pérdida para decidir cada corrección, no solo el gradiente como el Gradient Boosting clásico — converge más rápido y con más precisión hacia el mínimo del error.
- **Regularización explícita integrada** (`reg_alpha` = L1, `reg_lambda` = L2) directamente en la función objetivo que se minimiza en cada árbol — esto es exactamente el mismo concepto de Ridge/Lasso de [[34-Modelos-Lineales-en-Profundidad]], aplicado a los pesos de las hojas del árbol.
- **Manejo nativo de valores faltantes**: aprende automáticamente la dirección óptima (izquierda o derecha) para valores `NaN` en cada corte, sin necesidad de imputación previa — relevante si tu histórico de `qf.hrCallData` tiene huecos.
- **Poda mediante `gamma`**: un corte solo se acepta si la reducción de pérdida supera el umbral `gamma` — poda "hacia atrás" (el árbol crece completo y luego se recortan las ramas que no cumplen el umbral), a diferencia del crecimiento "hacia adelante" más simple de scikit-learn.

```python
from xgboost import XGBRegressor

modelo = XGBRegressor(
    n_estimators=300,
    max_depth=6,
    learning_rate=0.05,
    reg_alpha=0.1,           # L1 — puede llevar pesos de hojas a cero
    reg_lambda=1.0,           # L2 — encoge pesos de hojas
    gamma=0.1,                 # umbral mínimo de reducción de pérdida para aceptar un corte
    subsample=0.8,             # bagging parcial dentro del boosting — cada árbol ve el 80% de las filas
    colsample_bytree=0.8,      # cada árbol ve solo el 80% de los features — reduce correlación entre árboles
)
```

`subsample` y `colsample_bytree` son la forma en que XGBoost **toma prestado el concepto de aleatoriedad de Random Forest** dentro de un esquema de boosting — cada árbol individual ve una muestra distinta de filas y columnas, reduciendo la correlación entre árboles consecutivos y mejorando la generalización, sin perder el mecanismo de corrección secuencial de residuos.

## 5. Comparación práctica — cuándo cada uno tiene sentido

| | Random Forest | Gradient Boosting / XGBoost |
|---|---|---|
| Paralelización de entrenamiento | Total (árboles independientes) | Limitada (secuencial por diseño) |
| Riesgo de overfitting | Bajo, tolera árboles profundos | Alto si no se regula con cuidado |
| Sensibilidad a hiperparámetros | Baja — funciona razonablemente con defaults | Alta — requiere tuning cuidadoso |
| Rendimiento típico (datos tabulares) | Muy bueno | Generalmente superior, con buen tuning |
| Interpretabilidad directa | Moderada | Moderada (mejor con SHAP, ver nota 39) |

En la práctica de forecasting como el tuyo, XGBoost suele ganar en precisión final con suficiente tuning, pero Random Forest es un baseline más robusto y rápido de poner en marcha con menos riesgo de configurarlo mal.

## Ver también
- [[35-Arboles-de-Decision-en-Profundidad]]
- [[34-Modelos-Lineales-en-Profundidad]]
- [[37-Validacion-Rigurosa-en-ML]]
- [[15-MLflow-en-Profundidad]]
