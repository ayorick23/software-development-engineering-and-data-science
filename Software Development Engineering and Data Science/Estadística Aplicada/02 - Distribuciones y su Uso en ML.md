---
tags: [estadistica, distribuciones, machine-learning, cheat-sheet]
---

# 02 — Distribuciones y su Uso en ML

> Para la sintaxis completa de cada distribución en Python, ver [[Python/SciPy/04 - Estadística I - Distribuciones (scipy.stats)|SciPy]]. Para el fundamento teórico de cada una, ver las clases de [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 01 - Distribuciones de Probabilidad Discretas - Introducción a R|Estadística Inferencial]]. Este archivo responde una pregunta distinta: **qué distribución asumir según el problema de ML que se está resolviendo**, no solo sus fórmulas.

## Por qué la distribución que se asume importa en ML

Muchos modelos y métricas tienen, implícita o explícitamente, un supuesto sobre cómo se distribuyen los datos o el error. Elegir mal esa distribución no es un detalle académico: produce intervalos de confianza incorrectos, funciones de pérdida mal calibradas, o modelos que ignoran la verdadera forma de la incertidumbre.

## Guía rápida: variable → distribución típica → dónde aparece en ML

| Tipo de variable | Distribución típica | Dónde aparece en ML |
|---|---|---|
| Resultado binario (clic/no clic, fraude/no fraude) | Bernoulli / Binomial | Función de pérdida de regresión logística (log-loss = verosimilitud de una Bernoulli) |
| Conteo de eventos (visitas, errores, clics por hora) | Poisson | Regresión de Poisson; métricas de conteo en A/B testing |
| Tiempo hasta un evento (churn, falla de un componente) | Exponencial / Weibull | Modelos de supervivencia (survival analysis), estimación de vida útil |
| Error de un modelo de regresión bien especificado | Normal | Supuesto detrás de OLS, intervalos de predicción, muchas pruebas de significancia de coeficientes |
| Proporciones acotadas entre 0 y 1 (tasas, probabilidades predichas) | Beta | Calibración de probabilidades, estadística bayesiana (prior conjugado de Bernoulli/Binomial) |
| Variables con muchos valores extremos (ingresos, tamaños de archivo, tiempos de respuesta de red) | Log-normal / Pareto | Justifica transformar con `log()` antes de modelar |

## Bernoulli/Binomial — la base de la clasificación binaria

Cada predicción de un clasificador binario es, conceptualmente, un ensayo Bernoulli con probabilidad de éxito $\hat{p}$ (la probabilidad que predice el modelo). La función de pérdida estándar de regresión logística (*log-loss* / *binary cross-entropy*) es exactamente el negativo de la log-verosimilitud de una Bernoulli:

$$
\text{log-loss} = -\left[y \log(\hat{p}) + (1-y)\log(1-\hat{p})\right]
$$

Entender esto explica por qué log-loss penaliza tan fuerte una predicción confiada y equivocada ($\hat{p}$ cercano a 0 o 1 cuando el resultado real es el contrario) — el logaritmo tiende a $-\infty$ cerca de los extremos.

## Poisson — conteos, no proporciones

Cuando la variable objetivo es un conteo (número de clics, número de fallas, número de transacciones), un modelo de regresión lineal ordinaria puede predecir valores negativos, algo imposible para un conteo. La regresión de Poisson (o binomial negativa cuando la varianza excede a la media — *overdispersion*) modela directamente $\log(\lambda) = \beta_0 + \beta_1 x_1 + \dots$, garantizando predicciones no negativas.

```python
from scipy import stats

# Diagnóstico rápido de overdispersion: si var >> media, la Poisson pura no ajusta bien
conteos = [3, 5, 2, 8, 1, 4, 20, 3]   # ejemplo con un valor atípico alto
import numpy as np
media, varianza = np.mean(conteos), np.var(conteos)
print(f"media={media:.2f}, varianza={varianza:.2f}")   # varianza >> media -> considerar Binomial Negativa
```

## Normal — el supuesto detrás de la regresión lineal clásica

OLS (mínimos cuadrados ordinarios) no requiere que la variable objetivo en sí sea Normal, sino que los **residuos** (errores) lo sean, para que los intervalos de confianza de los coeficientes y las pruebas t sean válidos. Ver [[08 - Supuestos Estadísticos Detrás de los Modelos de ML]] para cómo verificarlo.

## Exponencial/Weibull — modelos de supervivencia y churn

El tiempo hasta que un cliente cancela un servicio (*churn*), o hasta que un componente falla, rara vez se modela con regresión estándar — se trata como un problema de **análisis de supervivencia**, donde la Exponencial (tasa de riesgo constante) o la Weibull (tasa de riesgo que cambia con el tiempo, más realista) son el punto de partida.

## Log-normal — por qué se transforma con logaritmo tan seguido

Muchas variables económicas y de ingeniería (ingresos, tamaños de archivo, tiempos de respuesta) tienen una cola derecha larga que se ajusta mucho mejor a una log-normal que a una Normal — de ahí que transformar con `np.log1p()` antes de modelar sea una práctica tan extendida: convierte una distribución muy asimétrica en una aproximadamente simétrica, donde los supuestos de modelos lineales se sostienen mejor.

```python
import numpy as np

df["ingreso_log"] = np.log1p(df["ingreso"])   # log1p = log(1+x), evita error si hay valores en 0
```

## Ver también

- [[Python/SciPy/04 - Estadística I - Distribuciones (scipy.stats)]]
- [[08 - Supuestos Estadísticos Detrás de los Modelos de ML]]
- [[Machine Learning/05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML]]
