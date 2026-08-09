---
tags: [machine-learning, metricas, evaluacion]
---

# 38 — Métricas de Regresión: Cuándo Usar Cada Una (y Cuándo NO)

> Nota del mentor: elegir la métrica equivocada para reportarle a tu jefa puede hacer que un modelo se vea mejor o peor de lo que realmente es en términos de impacto de negocio. No existe "la mejor métrica" en abstracto — existe la métrica correcta para la pregunta específica que estás respondiendo.

## 1. RMSE (Root Mean Squared Error) — penaliza errores grandes desproporcionadamente

```
RMSE = √(Σ(y_real - y_predicho)² / n)
```

```python
from sklearn.metrics import root_mean_squared_error
rmse = root_mean_squared_error(y_val, predicciones)
```

Al elevar al cuadrado antes de promediar, un error de 20 pesa **4 veces más** que un error de 10 (20² = 400 vs 10² = 100, no el doble). Esto hace que RMSE sea especialmente sensible a outliers y errores grandes puntuales.

**Úsalo cuando**: los errores grandes son desproporcionadamente costosos para el negocio — por ejemplo, si sub-dotar severamente una oficina un día específico genera un problema de servicio mucho más grave que estar ligeramente equivocado de forma consistente todos los días. Es exactamente por eso que tu sistema usa RMSE como **gate de tolerancia secundario**: no quieres promover un challenger que en promedio mejora (MAE menor) pero que ocasionalmente comete errores catastróficos en casos puntuales.

**NO lo uses cuando**: quieres una medida robusta a outliers, o cuando quieres comunicar el error "típico" a alguien no técnico — un RMSE de 18 no significa "en promedio me equivoco por 18", significa algo matemáticamente más complejo de explicar.

## 2. MAE (Mean Absolute Error) — el error "típico", robusto e interpretable

```
MAE = Σ|y_real - y_predicho| / n
```

```python
from sklearn.metrics import mean_absolute_error
mae = mean_absolute_error(y_val, predicciones)
```

Cada error pesa proporcionalmente a su magnitud, sin exagerar los grandes. Un MAE de 12 significa literalmente "en promedio, la predicción se equivoca por 12 unidades" — mucho más fácil de comunicar a tu jefa que un RMSE.

**Úsalo cuando**: quieres la métrica primaria de negocio, robusta y fácil de interpretar — exactamente por qué es tu métrica primaria de gate en el sistema de autoaprendizaje.

**NO lo uses cuando**: los errores grandes son mucho más costosos que muchos errores pequeños — MAE trata un error de 100 igual de "grave" (proporcionalmente) que dos errores de 50, cuando en la realidad de negocio un solo error de 100 podría ser mucho más disruptivo.

## 3. MAPE (Mean Absolute Percentage Error) — error relativo, cuidado con ceros

```
MAPE = (100/n) · Σ|y_real - y_predicho| / |y_real|
```

```python
from sklearn.metrics import mean_absolute_percentage_error
mape = mean_absolute_percentage_error(y_val, predicciones)
```

Expresa el error como porcentaje del valor real — parece atractivo porque es "intuitivo" (¿"me equivoco un 12% en promedio"?), pero tiene un defecto matemático serio: **se indefine o explota cuando `y_real` está cerca de cero**. En tu contexto: si una oficina tiene `total_demand = 0` en un intervalo (madrugada, oficina cerrada), el MAPE de ese punto es infinito o indefinido, distorsionando completamente el promedio.

**NO lo uses cuando**: tu variable objetivo puede tomar valores cero o cercanos a cero — exactamente el caso de demanda de llamadas en intervalos de baja actividad. Este es un error muy común que vale la pena que evites activamente en tus reportes.

## 4. WAPE (Weighted Absolute Percentage Error) — la alternativa correcta a MAPE

```
WAPE = Σ|y_real - y_predicho| / Σ|y_real|
```

```python
def wape(y_real, y_predicho):
    return np.sum(np.abs(y_real - y_predicho)) / np.sum(np.abs(y_real))
```

En vez de dividir cada error individual por su propio valor real (donde un solo cero rompe todo), WAPE suma todos los errores absolutos y los divide entre la suma de todos los valores reales — un solo intervalo con demanda cero no distorsiona el resultado global, porque su contribución al denominador se diluye entre la suma total.

**Úsalo cuando**: necesitas un error porcentual agregado (por ejemplo, para comparar el desempeño entre oficinas de tamaños muy distintos — una oficina grande y una pequeña) pero tu variable objetivo puede tener ceros o valores muy pequeños. Es, en la práctica de forecasting de demanda de la industria, la métrica porcentual estándar preferida sobre MAPE precisamente por esta robustez.

## 5. R² (Coeficiente de Determinación) — cuánta varianza explica el modelo

```
R² = 1 - (Σ(y_real - y_predicho)² / Σ(y_real - y_promedio)²)
```

```python
from sklearn.metrics import r2_score
r2 = r2_score(y_val, predicciones)
```

Compara tu modelo contra el baseline más simple posible: predecir siempre el promedio. `R² = 1` significa predicción perfecta; `R² = 0` significa que tu modelo no es mejor que predecir el promedio siempre; `R² < 0` (sí, puede ser negativo) significa que tu modelo es **peor** que simplemente predecir el promedio — una señal de alarma seria.

**Úsalo cuando**: quieres comunicar "qué tan bien el modelo captura la variabilidad general de los datos" en una sola cifra normalizada entre (aproximadamente) 0 y 1, útil para comparar el mismo modelo contra distintos datasets o presentar en un reporte ejecutivo.

**NO lo uses cuando**: necesitas entender el error en las unidades reales de negocio (agentes, llamadas) — R² es adimensional, no te dice si el error "típico" es aceptable operacionalmente. Tampoco es confiable como única métrica en series de tiempo con fuerte estacionalidad, donde incluso un modelo mediocre puede obtener R² alto solo por capturar el patrón estacional obvio.

## 6. Tabla resumen — qué reportar según la audiencia y el objetivo

| Métrica | Robusta a outliers | Interpretable en unidades reales | Robusta a ceros | Mejor para |
|---|---|---|---|---|
| RMSE | No | Parcialmente | Sí | Detectar errores grandes puntuales (gate secundario) |
| MAE | Sí | Sí | Sí | Reporte de negocio, métrica primaria |
| MAPE | No | No (%) | **No — evitar con ceros** | Comparación relativa, solo si no hay ceros |
| WAPE | Sí | No (%) | Sí | Comparación relativa entre oficinas, robusto |
| R² | No | No | Sí | Resumen ejecutivo de calidad general del modelo |

## 7. La lección más importante de esta nota

Nunca reportes una sola métrica como si fuera "la verdad absoluta" del desempeño del modelo. Tu sistema ya hace esto bien con MAE (primario) + RMSE (gate de tolerancia) — la práctica madura es siempre reportar al menos dos métricas complementarias, una robusta e interpretable (MAE) y otra que capture el riesgo de errores extremos (RMSE), y elegir con criterio cuál usar según a quién le estás hablando y qué decisión se va a tomar con ese número.

## Ver también
- [[37-Validacion-Rigurosa-en-ML]]
- [[36-Ensambles-en-Profundidad]]
- [[18-Monitoreo-y-Observabilidad-de-Modelos]]
