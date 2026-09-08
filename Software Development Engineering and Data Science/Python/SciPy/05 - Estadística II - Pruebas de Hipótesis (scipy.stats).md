---
tags: [scipy, estadistica, pruebas-de-hipotesis, cheat-sheet]
---

# 05 — Estadística II: Pruebas de Hipótesis (`scipy.stats`)

> Para el fundamento teórico de intervalos de confianza y estimación, ver [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 07 - Estimación e Intervalos de Confianza]] y [[Clase 08 - IC para Varianza y Proporciones]]. Para el uso de estas pruebas en un contexto de Data Science real (A/B testing, validación de supuestos de modelos), ver [[Estadística Aplicada/05 - Pruebas de Hipótesis para Data Science]].

## Prueba t de una muestra

```python
from scipy import stats
import numpy as np

consumo = np.array([512, 498, 530, 545, 505, 520, 488, 535, 512, 525, 499, 540, 518, 507, 530])

t_stat, p_valor = stats.ttest_1samp(consumo, popmean=512)
# H0: la media poblacional es 512
```

`ttest_1samp` no da directamente el intervalo de confianza (a diferencia de `t.test()` en R) — para eso se construye a mano con `stats.t.ppf`, o se usa `scipy.stats.bootstrap` (ver más abajo).

## Prueba t de dos muestras independientes

```python
grupo_a = np.array([23, 25, 21, 28, 24, 26])
grupo_b = np.array([30, 32, 29, 31, 33, 28])

# Por defecto asume varianzas iguales; equal_var=False usa la corrección de Welch (más robusta, recomendada casi siempre)
t_stat, p_valor = stats.ttest_ind(grupo_a, grupo_b, equal_var=False)
```

## Prueba t pareada

Cuando las dos muestras son mediciones repetidas sobre los mismos sujetos (antes/después):

```python
antes = np.array([80, 85, 78, 90, 88])
despues = np.array([82, 87, 80, 91, 90])

t_stat, p_valor = stats.ttest_rel(antes, despues)
```

## Chi-cuadrado

```python
# Bondad de ajuste — equivalente directo a chisq.test(x, p) en R
observado = [8, 12, 9, 11, 13, 7]
esperado  = [10, 10, 10, 10, 10, 10]
chi2_stat, p_valor = stats.chisquare(f_obs=observado, f_exp=esperado)

# Independencia en tabla de contingencia
tabla = np.array([[10, 20], [15, 25]])
chi2_stat, p_valor, df, esperado = stats.chi2_contingency(tabla)
```

## ANOVA de un factor

Comparar las medias de tres o más grupos simultáneamente (ver [[Machine Learning/05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML|fundamentos de validación]]):

```python
grupo1 = np.array([23, 25, 21, 28])
grupo2 = np.array([30, 32, 29, 31])
grupo3 = np.array([18, 20, 19, 22])

f_stat, p_valor = stats.f_oneway(grupo1, grupo2, grupo3)
```

## Correlación

```python
x = np.array([1, 2, 3, 4, 5])
y = np.array([2.1, 3.9, 6.2, 7.8, 10.1])

r, p_valor = stats.pearsonr(x, y)     # correlación lineal
rho, p_valor = stats.spearmanr(x, y)  # correlación de rangos (no asume linealidad ni normalidad)
```

## Prueba de normalidad

Requisito frecuente antes de aplicar pruebas paramétricas (t, ANOVA) o el IC para varianza (ver [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 08 - IC para Varianza y Proporciones#El Supuesto de Normalidad|el supuesto de normalidad]]):

```python
stat, p_valor = stats.shapiro(consumo)
# H0: los datos provienen de una Normal. p < 0.05 rechaza H0.
```

## Bootstrap: intervalos de confianza sin supuestos de distribución

Cuando no se puede asumir una distribución teórica (o la muestra es pequeña y sesgada), el bootstrap remuestrea los datos observados con reemplazo miles de veces para construir la distribución empírica de un estadístico:

```python
from scipy.stats import bootstrap

datos = (consumo,)   # bootstrap espera una tupla de muestras
resultado = bootstrap(datos, np.mean, n_resamples=10000, confidence_level=0.95)
resultado.confidence_interval   # ConfidenceInterval(low=..., high=...)
```

## Interpretación del p-valor: el mismo principio en todas las pruebas

Sin importar cuál prueba se use, la lectura es siempre la misma (ver [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 05 - Síntesis Unidad 1#"No Rechazar" no es "Demostrar"|"no rechazar" no es "demostrar"]]): un p-valor pequeño (< 0.05 por convención) es evidencia en contra de $H_0$; un p-valor grande solo indica que los datos son compatibles con $H_0$, no que se haya demostrado.

## Ver también

- [[04 - Estadística I - Distribuciones (scipy.stats)]]
- [[Estadística Aplicada/05 - Pruebas de Hipótesis para Data Science]]
- [[Estadística Aplicada/06 - A-B Testing de Principio a Fin]]
