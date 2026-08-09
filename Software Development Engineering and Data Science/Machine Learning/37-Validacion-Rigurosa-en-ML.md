---
tags: [machine-learning, validacion, data-leakage, series-de-tiempo]
---

# 37 — Validación Rigurosa: Temporal Split, Walk-Forward, Cross-Validation, Leakage

> Nota del mentor: esta es, sin exagerar, la nota más importante de toda la fase de Machine Learning. Un modelo con una validación mal hecha te miente con confianza — te muestra métricas excelentes en desarrollo y luego falla silenciosamente en producción. En forecasting de series de tiempo como el tuyo, la validación incorrecta es el error más común y más caro que existe.

## 1. Por qué K-Fold Cross-Validation estándar está PROHIBIDO en series de tiempo

```python
# INCORRECTO para series de tiempo — NUNCA hagas esto con datos de Claro RD
from sklearn.model_selection import KFold, cross_val_score
kf = KFold(n_splits=5, shuffle=True)  # shuffle=True es el error fatal aquí
```

K-Fold estándar mezcla aleatoriamente las filas y las divide en 5 grupos, entrenando con 4 y validando con 1 — pero en series de tiempo, esto significa que el modelo puede **entrenar con datos del futuro y validar con datos del pasado**, o viceversa. Un modelo que "aprende" usando información de la próxima semana para predecir la semana anterior obtiene métricas artificialmente excelentes que **nunca vas a poder replicar en producción real**, porque en producción real nunca tienes acceso al futuro cuando predices.

## 2. Temporal Split — la base mínima correcta

```python
fecha_corte = "2026-06-01"
train = datos[datos["interval_start"] < fecha_corte]
test = datos[datos["interval_start"] >= fecha_corte]
```

Simple: todo lo anterior a una fecha es entrenamiento, todo lo posterior es validación — el modelo nunca ve el futuro durante el entrenamiento. Es el mínimo indispensable, pero tiene una limitación real: **una sola partición** te da una sola medición de qué tan bien generaliza el modelo, sensible a particularidades de ese periodo específico de validación (¿fue una semana atípica? ¿coincidió con una promoción del cliente?).

## 3. Walk-Forward Validation — la validación seria para forecasting

```python
def walk_forward_validation(datos, ventana_entrenamiento_dias, ventana_validacion_dias, paso_dias):
    resultados = []
    fecha_inicio = datos["interval_start"].min()
    fecha_fin_disponible = datos["interval_start"].max()

    fecha_corte = fecha_inicio + pd.Timedelta(days=ventana_entrenamiento_dias)
    while fecha_corte + pd.Timedelta(days=ventana_validacion_dias) <= fecha_fin_disponible:
        train = datos[
            (datos["interval_start"] >= fecha_corte - pd.Timedelta(days=ventana_entrenamiento_dias))
            & (datos["interval_start"] < fecha_corte)
        ]
        val = datos[
            (datos["interval_start"] >= fecha_corte)
            & (datos["interval_start"] < fecha_corte + pd.Timedelta(days=ventana_validacion_dias))
        ]

        modelo = XGBRegressor().fit(train[features], train[target])
        mae = mean_absolute_error(val[target], modelo.predict(val[features]))
        resultados.append({"fecha_corte": fecha_corte, "mae": mae})

        fecha_corte += pd.Timedelta(days=paso_dias)  # avanza la ventana en el tiempo
    return pd.DataFrame(resultados)
```

En vez de una sola partición, walk-forward simula **repetidamente** el escenario real de producción: entrena con el histórico disponible hasta cierta fecha, valida con el periodo inmediatamente siguiente, avanza la ventana en el tiempo, y repite. Esto te da una distribución de métricas a través de distintos periodos — mucho más informativo que un solo número, y te permite ver si el modelo es consistentemente bueno o si su desempeño varía drásticamente según la época del año (algo muy relevante para un call center con estacionalidad).

Este es, en esencia, el mecanismo formal detrás de tus experimentos con ventanas de -30, -60, -90 y -120 días — walk-forward es la generalización sistemática de ese mismo tipo de prueba.

## 4. TimeSeriesSplit de scikit-learn — la versión "lista para usar"

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5, test_size=336)  # 336 intervalos de 30min = 7 días
for train_idx, val_idx in tscv.split(datos):
    train, val = datos.iloc[train_idx], datos.iloc[val_idx]
    # cada fold usa SOLO datos anteriores al fold de validación
```

`TimeSeriesSplit` implementa una variante de walk-forward donde cada fold sucesivo de entrenamiento **crece** incorporando los datos del fold de validación anterior (a diferencia del walk-forward de ventana fija de la sección anterior, que descarta los datos más antiguos). Ambas variantes son válidas — la elección depende de si crees que datos muy antiguos siguen siendo relevantes (usa `TimeSeriesSplit`, ventana creciente) o si sospechas que el patrón cambia con el tiempo y el histórico viejo puede ser ruido (usa walk-forward de ventana fija, como ya haces con `dias_atras`).

## 5. Data Leakage — la forma más sutil y peligrosa de invalidar una validación

El leakage ocurre cuando información que **no estaría disponible en el momento real de la predicción** se cuela en el entrenamiento, inflando artificialmente las métricas de validación.

### Leakage temporal (el más común en forecasting)
```python
# INCORRECTO: calcular el promedio usando TODO el dataset, incluyendo el futuro
datos["demanda_promedio_historica"] = datos["total_demand"].mean()  # usa TODO, incluye datos futuros del set

# CORRECTO: el promedio hasta cada punto debe calcularse solo con datos anteriores a ese punto
datos["demanda_promedio_historica"] = (
    datos.groupby("office_id")["total_demand"]
    .expanding().mean().shift(1).reset_index(level=0, drop=True)
)
```

El `shift(1)` es crítico: garantiza que el promedio en la fila de un intervalo dado **no incluya ese mismo intervalo**, solo los anteriores — exactamente lo que ya validaste en tu test de `build_lag_features` de la nota 13, pero aplicado ahora a features de agregación, no solo a lags simples.

### Leakage por preprocesamiento incorrecto (muy común y sutil)
```python
# INCORRECTO: escalar ANTES de partir train/test — el scaler "ve" el test set
scaler = StandardScaler()
datos_escalados = scaler.fit_transform(datos)  # fit sobre TODO el dataset
train, test = train_test_split(datos_escalados)

# CORRECTO: fit solo con train, transform aplicado a ambos
train, test = train_test_split(datos)
scaler = StandardScaler().fit(train)  # fit SOLO con train
train_escalado = scaler.transform(train)
test_escalado = scaler.transform(test)  # solo transform, nunca fit
```

Este patrón (`fit` solo en train, `transform` en ambos) aplica a **cualquier paso de preprocesamiento que aprenda estadísticas de los datos**: escalado, imputación de nulos con la media/mediana, selección de features por varianza, encoding de categorías — todos deben ajustarse exclusivamente con datos de entrenamiento.

### Leakage por variables proxy del objetivo
Un feature que está tan correlacionado con el objetivo que en la práctica "revela la respuesta" sin que sea obvio a simple vista — por ejemplo, si por error se incluyera `agentes_asignados` como feature para predecir `total_demand`, cuando `agentes_asignados` en realidad se calculó *a partir de* una versión anterior de la predicción de demanda. Este tipo de leakage es el más peligroso porque no se detecta con un `shift()` simple — requiere entender genuinamente el proceso de generación de cada columna del dataset.

## 6. Cómo saber si tienes leakage — la señal de alarma clásica

Si tu MAE de validación es sospechosamente mejor que el MAE que tu sistema ha logrado consistentemente en producción durante meses, **sospecha de leakage antes que de tener "un modelo excepcional"**. Un salto de calidad injustificado casi siempre es una señal de que el modelo está viendo información que no debería, no de que el algoritmo elegido sea genuinamente mejor.

## Ver también
- [[13-Testing-en-Machine-Learning]]
- [[36-Ensambles-en-Profundidad]]
- [[38-Metricas-de-Regresion-Cuando-Usar-y-Cuando-No]]
