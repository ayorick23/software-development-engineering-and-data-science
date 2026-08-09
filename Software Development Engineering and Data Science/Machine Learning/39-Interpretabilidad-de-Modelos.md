---
tags: [machine-learning, interpretabilidad, shap, feature-importance]
---

# 39 — Interpretabilidad de Modelos: Feature Importance, Permutation Importance, SHAP

> Nota del mentor: cuando tu jefa pregunta "¿por qué el modelo predijo que se necesitan 15 agentes y no 10?", "porque XGBoost lo dijo" no es una respuesta profesional aceptable. La interpretabilidad no es un lujo académico — es lo que te permite defender, depurar y confiar en un modelo que toma decisiones con impacto real de negocio.

## 1. Feature Importance nativo (Gini/Split-based) — rápido, pero con trampas

```python
importancias = pd.Series(modelo.feature_importances_, index=X_train.columns)
importancias.sort_values(ascending=False).head(10)
```

En árboles y ensambles, esto mide cuánto contribuyó cada feature a reducir la impureza (Gini) o el error (MSE) a lo largo de todos los cortes del modelo, sumado y normalizado. Es prácticamente gratis de calcular (viene incluido tras el entrenamiento), pero tiene **dos sesgos serios** que debes conocer:

- **Sesgo hacia features de alta cardinalidad**: una columna con muchos valores únicos (como un ID de oficina numérico continuo) tiende a mostrar importancia inflada solo porque el árbol tiene más oportunidades de encontrar cortes útiles en ella, no necesariamente porque sea más predictiva en un sentido causal real.
- **No refleja el efecto en datos nuevos**: mide qué tanto se usó el feature **durante el entrenamiento**, no qué tan útil es realmente para generalizar a datos que el modelo no ha visto.

## 2. Permutation Importance — mide el impacto real en el desempeño, no solo el uso interno

```python
from sklearn.inspection import permutation_importance

resultado = permutation_importance(
    modelo, X_val, y_val, n_repeats=10, random_state=42, scoring="neg_mean_absolute_error"
)
importancias_permutacion = pd.Series(resultado.importances_mean, index=X_val.columns)
```

El mecanismo: para cada feature, **mezcla aleatoriamente sus valores** (rompiendo su relación real con el objetivo) y mide cuánto empeora el desempeño del modelo ya entrenado. Si mezclar `demand_lag_1` empeora dramáticamente el MAE, ese feature es genuinamente importante para las predicciones reales. Si mezclarlo casi no cambia nada, el modelo no depende realmente de él, sin importar qué tan alto salga en el feature importance nativo.

**Ventaja clave sobre el nativo**: se calcula sobre el **set de validación**, midiendo impacto real en generalización, no solo uso durante el entrenamiento — y funciona con cualquier tipo de modelo (lineal, árboles, redes neuronales), no solo modelos basados en árboles.

**Costo**: computacionalmente más caro (requiere re-evaluar el modelo múltiples veces por cada feature), y puede subestimar la importancia de features fuertemente correlacionados entre sí (si `demand_lag_1` y `demand_lag_2` están muy correlacionados, mezclar solo uno de los dos no rompe tanto la señal porque el otro la sigue aportando).

## 3. SHAP (SHapley Additive exPlanations) — el estándar de la industria hoy

SHAP viene de la teoría de juegos cooperativos (valores de Shapley): calcula, para **cada predicción individual**, cuánto contribuyó cada feature a alejar esa predicción específica del promedio general — con garantías matemáticas de que las contribuciones se distribuyen de forma justa y consistente entre features.

```python
import shap

explainer = shap.TreeExplainer(modelo)  # optimizado para modelos de árboles (XGBoost, RF, GBM)
shap_values = explainer.shap_values(X_val)

# Explicación global: qué features importan más en general
shap.summary_plot(shap_values, X_val)

# Explicación de UNA predicción específica — la pregunta real de tu jefa
shap.force_plot(explainer.expected_value, shap_values[0], X_val.iloc[0])
```

### Por qué SHAP resuelve exactamente el problema de "¿por qué esta predicción específica?"

A diferencia de feature importance (global, "en general qué importa") y permutation importance (también global), SHAP te da una explicación **local**: para la predicción de la oficina 145 en el intervalo de las 2pm de hoy, SHAP te dice exactamente "la demanda base esperada era 10, pero `demand_lag_1` alto la subió +3, y `dia_semana=viernes` la subió +2, resultando en la predicción final de 15". Esta es exactamente la explicación defendible que necesitas cuando tu jefa cuestiona una predicción puntual.

### Propiedad matemática clave: aditividad

```
predicción = valor_base + Σ(contribuciones_SHAP_de_cada_feature)
```

Las contribuciones SHAP de todos los features de una predicción **suman exactamente** la diferencia entre la predicción final y el valor base (el promedio de todas las predicciones) — a diferencia de otros métodos de interpretabilidad más heurísticos, SHAP tiene esta garantía matemática de consistencia, derivada directamente de los axiomas de los valores de Shapley.

## 4. Comparación práctica — cuál usar y cuándo

| Método | Costo computacional | Nivel | Mejor para |
|---|---|---|---|
| Feature Importance nativo | Gratis (post-entrenamiento) | Global | Primer vistazo rápido, exploración inicial |
| Permutation Importance | Medio-alto | Global | Confirmar importancia real en validación, cualquier modelo |
| SHAP | Alto (aunque `TreeExplainer` está muy optimizado) | Global y local | Explicar predicciones individuales, auditoría, reportes al negocio |

**Flujo práctico recomendado**: usa feature importance nativo para exploración rápida durante el desarrollo del modelo. Confirma con permutation importance antes de tomar decisiones de feature selection. Usa SHAP cuando necesites explicar predicciones específicas a stakeholders no técnicos, o cuando estés auditando por qué el modelo se comporta de forma inesperada en un caso puntual.

## 5. Interpretabilidad como parte del monitoreo continuo

Esto no es solo una herramienta de desarrollo — conecta directo con [[18-Monitoreo-y-Observabilidad-de-Modelos]]: si el **feature importance de SHAP cambia significativamente entre versiones del modelo** (el challenger empieza a depender mucho más de un feature que el champion casi no usaba), es una señal de alerta que merece investigación antes de promover el modelo, incluso si el MAE/RMSE del gate pasan — un cambio drástico en qué features impulsan las predicciones puede indicar que el modelo está capturando una relación espuria nueva en los datos recientes, no necesariamente una mejora genuina.

## Ver también
- [[36-Ensambles-en-Profundidad]]
- [[38-Metricas-de-Regresion-Cuando-Usar-y-Cuando-No]]
- [[18-Monitoreo-y-Observabilidad-de-Modelos]]
- [[40-Feature-Engineering-Avanzado]]
