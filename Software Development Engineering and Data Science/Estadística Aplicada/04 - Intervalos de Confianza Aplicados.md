---
tags: [estadistica, intervalos-de-confianza, cheat-sheet]
---

# 04 — Intervalos de Confianza Aplicados

> Para la construcción clásica (Z/t) del intervalo de confianza para la media, ver [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 07 - Estimación e Intervalos de Confianza]]. Este archivo cubre casos que la construcción clásica no resuelve directamente: métricas de modelos de ML y la comparación de dos grupos en un experimento.

## Recordatorio de la anatomía (no se repite la derivación)

$$
\text{estimación puntual} \pm \text{valor crítico} \times \text{error estándar}
$$

Para la media con σ desconocida, `scipy.stats.t.interval` reproduce directamente el cálculo de la clase de Estadística Inferencial:

```python
import numpy as np
from scipy import stats

datos = np.array([512, 498, 530, 545, 505, 520, 488, 535, 512, 525])
media = datos.mean()
error_estandar = stats.sem(datos)   # "standard error of the mean" -- s/sqrt(n), ya calculado

ic_95 = stats.t.interval(0.95, df=len(datos)-1, loc=media, scale=error_estandar)
```

## IC para una diferencia de medias (A/B testing)

El caso más frecuente en la práctica de Data Science no es un IC para una sola media, sino para la **diferencia** entre dos grupos (control vs. tratamiento):

```python
control = np.array([120, 115, 130, 125, 118, 122])
tratamiento = np.array([135, 140, 128, 145, 138, 133])

diferencia = tratamiento.mean() - control.mean()

# Error estándar de la diferencia de dos medias independientes
error_estandar_diff = np.sqrt(stats.sem(control)**2 + stats.sem(tratamiento)**2)
df_aprox = len(control) + len(tratamiento) - 2

ic_95 = stats.t.interval(0.95, df=df_aprox, loc=diferencia, scale=error_estandar_diff)
print(f"Diferencia: {diferencia:.2f}, IC 95%: {ic_95}")
```

Si el intervalo completo queda por encima de cero, hay evidencia de que el tratamiento es mejor; si incluye el cero, la diferencia observada no es concluyente al nivel de confianza elegido — la misma lógica de interpretación vista en las clases de Estadística Inferencial, aplicada a una diferencia en vez de a una sola media.

## IC para una proporción (tasa de conversión, tasa de error)

```python
from statsmodels.stats.proportion import proportion_confint

exitos, n = 148, 400
ic_95 = proportion_confint(exitos, n, alpha=0.05, method='wilson')   # Wilson: más preciso que Wald con muestras pequeñas
```

`statsmodels` ofrece varios métodos (`'normal'` = Wald, `'wilson'`, `'beta'` = Clopper-Pearson exacto) — ver la comparación de métodos en [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 08 - IC para Varianza y Proporciones#Tres Métodos en R|la clase de IC para proporciones]].

## IC para métricas de modelos de ML sin fórmula cerrada (bootstrap)

Para métricas como F1, AUC, o un percentil de error, no existe una fórmula analítica simple de error estándar — la solución es el bootstrap descrito en [[03 - Muestreo, TCL y Bootstrap en la Práctica]]:

```python
# ic_95 = np.percentile(metricas_bootstrap, [2.5, 97.5])  -- ver archivo 03
```

## Un intervalo ancho también es información

Un IC muy ancho (por ejemplo, para la mejora esperada de un modelo nuevo) no es un resultado "sin valor" — es información honesta de que la muestra actual no permite una conclusión precisa, y señala que se necesitan más datos antes de decidir. Reportar solo el punto central (la diferencia observada) sin su intervalo es el error más común y más costoso al comunicar resultados de un experimento.

## Ver también

- [[03 - Muestreo, TCL y Bootstrap en la Práctica]]
- [[05 - Pruebas de Hipótesis para Data Science]]
- [[06 - A-B Testing de Principio a Fin]]
