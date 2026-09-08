---
tags: [scipy, estadistica, distribuciones, cheat-sheet]
---

# 04 — Estadística I: Distribuciones (`scipy.stats`)

> Para el fundamento teórico de cada distribución (fórmulas, cuándo aplica cada una), ver [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 01 - Distribuciones de Probabilidad Discretas - Introducción a R|las clases de Estadística Inferencial]] — este archivo es la equivalencia práctica en Python de lo que ahí se hace en R. Para el uso orientado a decisiones de modelado, ver [[Estadística Aplicada/02 - Distribuciones y su Uso en ML]].

## La API unificada de `scipy.stats`

Todas las distribuciones de `scipy.stats` —discretas y continuas— comparten la misma interfaz de métodos, sin importar cuál se use. Aprender el patrón una vez sirve para las ~100 distribuciones que la librería implementa:

```python
from scipy import stats

dist = stats.norm(loc=170, scale=8)   # Normal(mu=170, sigma=8) — "loc" y "scale", no "mean"/"std"

dist.pdf(180)     # densidad en x=180 (continua) / pmf en discretas
dist.cdf(180)     # P(X <= 180)
dist.sf(180)      # P(X > 180) — "survival function", más preciso que 1 - cdf en las colas
dist.ppf(0.95)    # cuantil: valor x tal que P(X <= x) = 0.95 (inverso de cdf)
dist.rvs(size=1000)   # generar valores aleatorios
dist.mean(); dist.std(); dist.var()
```

| Método | Equivalente en R (Estadística Inferencial) |
|---|---|
| `.pdf(x)` / `.pmf(x)` | `dnorm`, `dbinom`, ... |
| `.cdf(x)` | `pnorm`, `pbinom`, ... |
| `.ppf(p)` | `qnorm`, `qbinom`, ... |
| `.rvs(size=n)` | `rnorm`, `rbinom`, ... |

## Distribuciones continuas más usadas

```python
stats.norm(loc=0, scale=1)              # Normal
stats.uniform(loc=0, scale=10)          # Uniforme en [0, 10] — OJO: scale es el ancho, no el límite superior
stats.expon(scale=1/5)                  # Exponencial, scale = 1/lambda
stats.gamma(a=2, scale=1/3)             # Gamma, a = shape (k), scale = 1/rate
stats.chi2(df=5)                        # Chi-cuadrado
stats.t(df=14)                          # t de Student
stats.f(dfn=8, dfd=10)                  # F de Snedecor
```

La trampa más frecuente: `scipy.stats.uniform(loc, scale)` no toma "mínimo y máximo" como en R (`dunif(min, max)`) sino **origen y ancho** — `uniform(loc=10, scale=5)` es $U(10, 15)$, no $U(10, 5)$.

## Distribuciones discretas

```python
stats.binom(n=10, p=0.85)
stats.poisson(mu=3)                     # OJO: el parámetro se llama "mu", no "lambda" (palabra reservada implícita en varios contextos)
stats.geom(p=0.2)                       # aquí SÍ cuenta ensayos (1, 2, 3, ...), a diferencia de R
stats.hypergeom(M=20, n=6, N=5)         # M=N total, n=K éxitos, N=tamaño de muestra — nombres distintos a R otra vez
```

## Ajustar una distribución a datos observados

Dado un conjunto de datos, estimar los parámetros de la distribución que mejor los describe (equivalente al `λ̂` estimado en la [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 05 - Síntesis Unidad 1#Prueba de Bondad de Ajuste (χ²)|prueba de bondad de ajuste]]):

```python
import numpy as np

datos = np.random.normal(50, 5, 200)   # datos de ejemplo
mu_hat, sigma_hat = stats.norm.fit(datos)   # estimación por máxima verosimilitud
```

## Estadística descriptiva rápida

```python
stats.describe(datos)   # nobs, minmax, mean, variance, skewness, kurtosis en un solo objeto
stats.zscore(datos)     # estandarización vectorizada
stats.iqr(datos)        # rango intercuartílico
```

## Ver también

- [[05 - Estadística II - Pruebas de Hipótesis (scipy.stats)]]
- [[Estadística Aplicada/02 - Distribuciones y su Uso en ML]]
- [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 01 - Distribuciones de Probabilidad Discretas - Introducción a R]]
