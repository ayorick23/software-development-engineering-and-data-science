---
tags: [machine-learning, feature-engineering, series-de-tiempo]
---

# 40 — Feature Engineering Avanzado

> Nota del mentor: en 20 años he visto equipos gastar semanas migrando de Random Forest a XGBoost a redes neuronales buscando 2% más de precisión, cuando un feature bien diseñado hubiera dado 10%. Esta es, honestamente, la habilidad individual que más retorno da en proyectos de forecasting como el tuyo — más que la elección del algoritmo.

## 1. Features de series de tiempo — la base de tu dominio

### Lags — el pasado como predictor del presente
```python
for lag in [1, 2, 48, 336]:  # 30min, 1h, 1 día, 1 semana (en intervalos de 30min)
    df[f"demand_lag_{lag}"] = df.groupby("office_id")["total_demand"].shift(lag)
```

Elegir **qué** lags incluir no es arbitrario — debe reflejar los ciclos reales del negocio: el intervalo inmediatamente anterior (autocorrelación de corto plazo), el mismo intervalo del día anterior (patrón diario), y el mismo intervalo de la semana anterior (patrón semanal, crucial si el call center tiene comportamiento distinto lunes vs. sábado).

### Ventanas móviles — suavizado de tendencia reciente
```python
df["promedio_movil_7"] = df.groupby("office_id")["total_demand"].transform(
    lambda x: x.shift(1).rolling(window=7).mean()
)
df["desviacion_movil_7"] = df.groupby("office_id")["total_demand"].transform(
    lambda x: x.shift(1).rolling(window=7).std()
)
```

Nota el `.shift(1)` **antes** del `.rolling()` — sin él, la ventana móvil de la fila actual incluiría el propio valor que se intenta predecir, exactamente el leakage temporal descrito en [[37-Validacion-Rigurosa-en-ML]]. `desviacion_movil_7` es tan valiosa como el promedio: te dice si la oficina ha tenido demanda volátil o estable recientemente, información que un simple promedio no captura.

### Features cíclicos — capturar periodicidad sin discontinuidades artificiales
```python
df["hora_sin"] = np.sin(2 * np.pi * df["hora_del_dia"] / 24)
df["hora_cos"] = np.cos(2 * np.pi * df["hora_del_dia"] / 24)
```

El problema que resuelve: si codificas la hora como un número simple (0-23), el modelo ve la hora 23 y la hora 0 como "muy lejanas" numéricamente, cuando en realidad son consecutivas (23:59 → 00:00). La codificación seno/coseno preserva la naturaleza cíclica — la distancia entre 23h y 0h es correctamente pequeña en este espacio transformado.

## 2. Features de calendario — negocio, no solo tiempo

```python
df["es_fin_de_semana"] = df["interval_start"].dt.dayofweek.isin([5, 6]).astype(int)
df["es_feriado"] = df["fecha"].isin(feriados_republica_dominicana)
df["dias_hasta_proximo_feriado"] = calcular_dias_hasta_feriado(df["fecha"])
df["es_quincena"] = df["interval_start"].dt.day.isin([15, 30, 31])  # patrones de pago pueden afectar llamadas
```

Estos features requieren **conocimiento de dominio del negocio de Claro RD**, no solo manipulación técnica de fechas — es exactamente el tipo de feature que un ingeniero de ML que entiende el negocio (tú, después de meses en el proyecto) puede diseñar mejor que alguien puramente técnico sin ese contexto.

## 3. Features de interacción — cuando la relación no es simplemente aditiva

```python
df["demanda_x_es_feriado"] = df["demand_lag_336"] * df["es_feriado"]
```

A veces el efecto de un feature depende del valor de otro — un feriado afecta la demanda de forma distinta según si la oficina normalmente tiene alta o baja demanda base. Los modelos de árboles (Random Forest, XGBoost) pueden capturar interacciones automáticamente hasta cierto punto mediante cortes sucesivos, pero features de interacción explícitos ayudan al modelo a encontrar el patrón más rápido y con menos datos, especialmente en modelos lineales (donde son indispensables, ya que un modelo lineal sin ellos **no puede** capturar interacciones por diseño).

## 4. Encoding de variables categóricas — más allá de One-Hot

```python
# One-Hot: correcto para pocas categorías sin orden
df_encoded = pd.get_dummies(df, columns=["turno"])  # mañana/tarde/noche

# Target Encoding: para categorías de alta cardinalidad (ej. office_id con 200+ oficinas)
# CUIDADO: debe calcularse SOLO con datos de entrenamiento para evitar leakage
medias_por_oficina = train.groupby("office_id")["total_demand"].mean()
train["office_target_encoded"] = train["office_id"].map(medias_por_oficina)
val["office_target_encoded"] = val["office_id"].map(medias_por_oficina)  # usa las medias de TRAIN
```

**One-Hot Encoding** es correcto para pocas categorías (turno, día de la semana) — crea una columna binaria por categoría, sin asumir ningún orden entre ellas. Con `office_id` (potencialmente cientos de oficinas), one-hot generaría cientos de columnas dispersas — ahí **Target Encoding** (reemplazar cada categoría por el promedio histórico del objetivo para esa categoría) es más eficiente, pero exige el mismo cuidado de leakage que cualquier estadística calculada sobre datos: **ajustarlo solo con train, nunca con el dataset completo**.

## 5. El proceso, no solo la técnica — cómo diseñar features con criterio

1. **Empieza con hipótesis de negocio, no con técnicas al azar**: "sospecho que la demanda de los viernes se comporta distinto" es una hipótesis útil; "voy a probar 50 transformaciones matemáticas a ver cuál pega" no lo es, y es la receta perfecta para overfitting por selección de features basada en el propio set de validación.
2. **Valida cada feature nuevo con el mismo rigor de [[37-Validacion-Rigurosa-en-ML]]**: agrega el feature, corre walk-forward validation, compara contra el baseline sin ese feature. Un feature que "se ve bien" en una sola partición puede ser ruido — solo confía en mejoras consistentes across múltiples ventanas de validación.
3. **Usa SHAP ([[39-Interpretabilidad-de-Modelos]]) para verificar que el feature nuevo realmente se está usando** de forma sensata, no solo que "no empeora" la métrica — a veces un feature mejora el MAE por razones espurias no relacionadas con la hipótesis original que lo motivó.
4. **Elimina features que no aportan**: más features no siempre es mejor — features irrelevantes agregan ruido, aumentan el riesgo de overfitting, y ralentizan el entrenamiento y la inferencia sin beneficio real. Permutation importance (nota 39) es tu herramienta principal para esta poda.

## 6. Por qué esto suele importar más que el algoritmo

Un modelo lineal regularizado con excelentes features frecuentemente supera a un XGBoost sin tunear con features pobres. La razón es simple: el algoritmo solo puede aprender patrones que **existen explícitamente** en los datos que le das. Si la relación real depende de "demanda del mismo día la semana pasada" y ese feature no existe en tu dataset, ningún algoritmo — por sofisticado que sea — puede inventarlo desde cero solo con `total_demand` cruda. El feature engineering es, literalmente, el proceso de hacerle explícito al modelo el conocimiento de negocio que tú ya tienes como humano que entiende el dominio.

## Ver también
- [[37-Validacion-Rigurosa-en-ML]]
- [[39-Interpretabilidad-de-Modelos]]
- [[29-SQL-para-Machine-Learning]]
- [[36-Ensambles-en-Profundidad]]
