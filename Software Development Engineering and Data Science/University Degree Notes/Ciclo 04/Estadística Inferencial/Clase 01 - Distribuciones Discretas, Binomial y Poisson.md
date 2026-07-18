---
Fecha de creación: 2026-07-08T18:39:00
Materia:
  - Estadística Inferencial
Fecha de clase: 2026-07-08
---
# Distribuciones Discretas: Binomial y Poisson

Las **distribuciones de probabilidad discretas** modelan fenómenos donde los resultados posibles son contables o finitos (como los números enteros). Cada valor tiene asignada una probabilidad y la suma de todas ellas es exactamente 1. Son herramientas fundamentales para predecir eventos y tomar decisiones en escenarios controlados.

## Variable Aleatoria Discreta

Una **variable aleatoria (V.A.)** es una función que asigna un valor numérico a cada resultado posible de un experimento aleatorio. Se denomina discreta cuando solo puede tomar valores enteros contables (finitos o infinitos numerables), como el número de defectos en un lote, el número de llamadas que llegan a un servidor, o el número de clics en un anuncio.

Toda distribución discreta debe cumplir dos condiciones fundamentales: (1) cada probabilidad debe ser no negativa, es decir, $P(X = x) ≥ 0$ para todo valor posible $x$; y (2) la suma de todas las probabilidades debe ser exactamente igual a $1$.

### Valor Esperado y Varianza

Para una variable aleatoria discreta $X$ que toma valores $x_1, x_2,..., x_n$ con probabilidades $p_1, p_2,..., p_n$, el valor esperado (media) representa el promedio ponderado de los resultados posibles:

$$
E[X] = \sum x_i \cdot P(X = x_i)
$$

Suma de cada valor posible multiplicado por su probabilidad. Representa el valor promedio a largo plazo del experimento.

$$
Var(X) = E[X^2] - (E[X])^2
$$

Indica qué tan alejados tienden a estar los valores del promedio. La desviación estándar es $σ = \sqrt{Var(X)}$.

---

## Distribución Binomial

La **distribución Binomial** modela el número de éxitos en una secuencia de n experimentos independientes, donde cada uno tiene exactamente dos resultados posibles (éxito o fracaso) y la probabilidad de éxito p es constante en cada intento. A este experimento individual se le llama **ensayo de Bernoulli**.

### Supuestos del modelo Binomial

- El experimento consiste en n ensayos idénticos e independientes.
- Cada ensayo tiene solo dos resultados posibles: éxito (con probabilidad $p$) o fracaso (con probabilidad $1 − p$).
- La probabilidad $p$ es constante en todos los ensayos.
- La variable de interés $X$ es el número total de éxitos en los $n$ ensayos.

### Función de Probabilidad

Si $X ~ B(n, p)$, la probabilidad de obtener exactamente x éxitos en n ensayos es:

$$
P(X=x) = C(n,x) \cdot p^x \cdot (1-p)^{n-x}
$$

Donde $C(n,x) = n! / (x!(n-x)!)$ es el coeficiente binomial. $x$ puede tomar los valores $0, 1, 2, …, n$.

Sus parámetros son: Media: $E[X] = n\cdot p$ y Varianza: $Var(X) = n\cdot p\cdot(1−p)$.

**Ejemplo:**

Un sistema de reconocimiento de imágenes clasifica correctamente el 85% de las imágenes que procesa. Si se procesan 10 imágenes de forma independiente, ¿cuál es la probabilidad de que exactamente 8 sean clasificadas correctamente?

**Solución:**

- Identificamos: $n = 10$, $p = 0.85$, $x = 8$
- $C(10,8) = 10! / (8! \cdot 2!) = 45$
- $P(X = 8) = 45 \cdot (0.85)^8 \cdot (0.15)^2 = 45 \cdot 0.2725 \cdot 0.0225 \approx 0.2759$

**Verificación en R:**

```r
# Distribución Binomial: n=10, p=0.85
dbinom(x = 8, size = 10, prob = 0.85)
# [1] 0.2758956

# Probabilidad acumulada: P(X <= 8)
pbinom(q = 8, size = 10, prob = 0.85)
# [1] 0.8202
```

---

## Distribución de Poisson

La **distribución de Poisson** modela el número de veces que ocurre un evento en un intervalo fijo de tiempo, espacio o cualquier otra unidad de medida, cuando los eventos ocurren de forma **independiente** y a una tasa **promedio constante $\lambda$ (lambda)**. Es especialmente útil cuando $n$ es grande y $p$ es pequeña.

### Supuestos del Modelo de Poisson

- Los eventos ocurren de forma independiente entre sí.
- La tasa promedio de ocurrencia $\lambda$ es constante en el intervalo de interés.
- En un intervalo muy pequeño, la probabilidad de que ocurran dos o más eventos es despreciable.
- La variable $X$ puede tomar cualquier valor entero no negativo: $0, 1, 2, 3, …$

### Función de Probabilidad

$$
P(X=x) = (\frac{e^{-\lambda} \cdot \lambda ^x}{x!})
$$

Donde $\lambda > 0$ es la tasa media de ocurrencia del evento en el intervalo, $e \approx 2.71828$, y $x = 0, 1, 2, …$

Sus parámetros son: Media: $E[X] = \lambda$ y Varianza: $Var(X) = \lambda$. Nótese que en Poisson, la media y la varianza son iguales, lo que es una propiedad útil para identificar si un dataset sigue este modelo.

**Ejemplo:**

Un servidor web recibe en promedio 3 solicitudes por segundo. ¿Cuál es la probabilidad de recibir exactamente 5 solicitudes en un segundo? ¿Cuál es la probabilidad de recibir a lo sumo 2 solicitudes?

**Solución:**

- Identificamos: $\lambda = 3$ (solicitudes por segundo), $e \approx 2.71828$
- $P(X = 5) = (e^{-3} · 3^{5}) / 5! = (0.0498 · 243) / 120 \approx 0.1008$
- $P(X ≤ 2) = P(0) + P(1) + P(2) = 0.0498 + 0.1494 + 0.2240 \approx 0.4232$

**Verificación en R:**

```r
# Distribución de Poisson: lambda = 3
dpois(x = 5, lambda = 3) # P(X = 5)
# [1] 0.1008188

ppois(q = 2, lambda = 3) # P(X <= 2)
# [1] 0.4231901
```

---

## Comparación: ¿Cuándo usar cada modelo?



---
## Introducción a R

**R** es un lenguaje de programación y software libre diseñado específicamente para el análisis estadístico, la manipulación de datos y la creación de gráficos. Es una de las herramientas más utilizadas por científicos de datos, estadísticos e investigadores para procesar grandes volúmenes de información y visualizar resultados complejos

### RStudio

**RStudio** es un entorno de desarrollo integrado (IDE) de código abierto diseñado específicamente para el lenguaje de programación **R**, utilizado ampliamente para el análisis de datos, computación estadística y visualización.

### Funciones Clave para Distribuciones Discretas

R usa un sistema de prefijos consistente para todas las distribuciones: ``d`` (densidad/probabilidad puntual), ``p`` (probabilidad acumulada), ``q`` (cuantil) y ``r`` (números aleatorios).

| Prefijo         | Binomial                | Poisson            |
| --------------- | ----------------------- | ------------------ |
| `d (P.puntual)` | `dbinom(x, size, prob)` | `dpois(x, lambda)` |
| `p (acumulada)` | `pbinom(q, size, prob)` | `ppois(x, lambda)` |
| `q (cuantil)`   | `qbinom(p, size, prob)` | `qpois(x, lambda)` |
| `r (aleatorio)` | `rbinom(n, size, prob)` | `rpois(x, lambda)` |
