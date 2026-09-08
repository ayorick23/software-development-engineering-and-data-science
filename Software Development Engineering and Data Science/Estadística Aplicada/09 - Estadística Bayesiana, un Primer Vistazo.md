---
tags: [estadistica, bayesiana, cheat-sheet]
---

# 09 — Estadística Bayesiana, un Primer Vistazo

> Todo lo cubierto en los archivos anteriores (t-test, IC, p-valores) pertenece al enfoque **frecuentista**. Este archivo es una introducción breve al enfoque alternativo — no pretende ser exhaustivo, sino dar el vocabulario y la intuición mínima para reconocer cuándo el pensamiento bayesiano es la herramienta más natural.

## La diferencia central: qué es aleatorio

| | Frecuentista | Bayesiano |
|---|---|---|
| El parámetro (ej. $\mu$, $p$) | Fijo pero desconocido | Tratado como una variable aleatoria con una distribución de creencia |
| Los datos | Aleatorios (varían de muestra en muestra) | Fijos, una vez observados |
| "IC del 95%" significa | El *método* captura el valor real el 95% de las veces (ver [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 07 - Estimación e Intervalos de Confianza#Interpretación Correcta de un Intervalo de Confianza|la interpretación correcta vista en Estadística Inferencial]]) | Hay 95% de probabilidad (en el sentido de creencia) de que el parámetro esté en ese rango — una afirmación directa que el enfoque frecuentista prohíbe |

Esta es exactamente la distinción que la clase de Estadística Inferencial marca como el error más común al interpretar un IC frecuentista — en el enfoque bayesiano, esa misma afirmación ("hay 95% de probabilidad de que $\mu$ esté aquí") sí es válida, porque el parámetro se trata como aleatorio.

## El teorema de Bayes, aplicado a inferencia de parámetros

$$
P(\theta \mid \text{datos}) = \frac{P(\text{datos} \mid \theta) \cdot P(\theta)}{P(\text{datos})}
$$

- $P(\theta)$ — el **prior**: la creencia sobre el parámetro *antes* de ver los datos.
- $P(\text{datos} \mid \theta)$ — la **verosimilitud**: qué tan probables son los datos observados, para cada valor posible de $\theta$.
- $P(\theta \mid \text{datos})$ — el **posterior**: la creencia actualizada, combinando el prior con la evidencia de los datos.

## Ejemplo mínimo: estimar una tasa de conversión con un prior Beta

La distribución Beta es el prior conjugado de una Bernoulli/Binomial — combinarlos da un posterior que también es Beta, con una actualización que se calcula sin integrales complicadas:

```python
from scipy import stats

# Prior: Beta(2, 2) -- una creencia suave, centrada en 0.5, sin mucha certeza
alpha_prior, beta_prior = 2, 2

# Datos observados: 48 conversiones de 400 visitantes
exitos, fracasos = 48, 400 - 48

# Posterior: se suma directamente a los parámetros del prior (la "magia" de la conjugación)
alpha_posterior = alpha_prior + exitos
beta_posterior = beta_prior + fracasos

posterior = stats.beta(alpha_posterior, beta_posterior)

posterior.mean()                 # estimación puntual de la tasa de conversión
posterior.interval(0.95)         # intervalo de credibilidad del 95% -- interpretable como probabilidad directa
```

## Cuándo el enfoque bayesiano es especialmente útil en la práctica

- **Datos escasos**, donde incorporar conocimiento previo (de experimentos anteriores, de la literatura) mejora la estimación más que confiar solo en la muestra pequeña actual.
- **A/B testing secuencial**: a diferencia del enfoque frecuentista clásico (donde revisar resultados repetidamente antes de terminar el experimento infla falsos positivos, ver [[06 - A-B Testing de Principio a Fin#Errores clásicos de A/B testing|"peeking"]]), los métodos bayesianos están diseñados para permitir monitoreo continuo sin esa penalización.
- **Comunicar resultados a no estadísticos**: "hay 95% de probabilidad de que el tratamiento sea mejor" es una afirmación más intuitiva y directa que la interpretación frecuentista de un IC, aunque requiere aceptar el marco bayesiano para que sea válida.

## La objeción más común: la elección del prior

La crítica estándar al enfoque bayesiano es que el prior introduce subjetividad — dos analistas con priors distintos pueden llegar a conclusiones distintas con los mismos datos. La respuesta práctica: con suficientes datos, el posterior converge al mismo resultado sin importar el prior inicial (la verosimilitud domina); el prior importa más precisamente cuando los datos son escasos, que es también cuando más ayuda incorporar conocimiento previo razonable.

## Ver también

- [[04 - Intervalos de Confianza Aplicados]]
- [[06 - A-B Testing de Principio a Fin]]
- [[Python/SciPy/04 - Estadística I - Distribuciones (scipy.stats)]]
