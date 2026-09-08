---
Fecha de creación: 2026-07-22T18:28:00
Materia:
  - Estadística Inferencial
Fecha de clase: 2026-07-22
---
[[Clase 02 - Distribuciones de Probabilidad Discretas|← Clase anterior]] | [[Clase 04 - Cinco Distribuciones Continuas|Clase siguiente →]]

# Distribuciones de Probabilidad Continuas

Las variables discretas se **cuentan** (número de éxitos, número de eventos): sus valores posibles son $0, 1, 2, 3, \dots$ y cada uno tiene asignada una probabilidad puntual. Las variables continuas se **miden** (tiempo, peso, altura, dinero): sus valores posibles llenan un intervalo completo de números reales. Este cambio trae dos consecuencias:

- La probabilidad de un valor exacto es cero: si $X$ es continua, $P(X=3.7) = 0$. Las preguntas siempre son sobre intervalos: $P(X<3.7)$, $P(2<X<5)$, etc.
- La probabilidad se mide como **área bajo una curva**. Cada distribución continua tiene una función de densidad $f(x)$, y $P(a<X<b)$ es el área bajo $f(x)$ entre $a$ y $b$. El área total bajo la curva siempre vale 1.

---

## Distribución Uniforme Continua

La **distribución Uniforme continua** modela una variable que puede tomar cualquier valor entre $a$ y $b$ con igual probabilidad. Su densidad es un rectángulo de base $(b-a)$ y altura $1/(b-a)$. Aparece en el tiempo de espera de un evento periódico, el ángulo de una ruleta, o el error de redondeo.

Se escribe $X \sim U(a,b)$:

$$
f(x) = \frac{1}{b-a}, \qquad a \le x \le b
$$

Como la densidad es constante, la probabilidad de un subintervalo se calcula multiplicando su longitud por la altura:

$$
P(c<X<d) = \frac{d-c}{b-a}
$$

$$
E(X) = \frac{a+b}{2} \qquad V(X) = \frac{(b-a)^2}{12}
$$

**Comandos de R:** `dunif(x, min, max)`, `punif(x, min, max)`, `qunif(p, min, max)`, `runif(n, min, max)`.

**Ejemplo:**

Un bus pasa cada 15 minutos exactos. Un estudiante llega en un momento aleatorio; $X \sim U(0,15)$. (a) ¿Probabilidad de esperar menos de 5 minutos? (b) Tiempo esperado de espera.

**Solución:**

$$
P(X<5) = \frac{5-0}{15-0} = 0.3333
$$

$E(X) = (0+15)/2 = 7.5$ min, $V(X) = 15^2/12 = 18.75$

**Verificación en R:**

```r
punif(5, min = 0, max = 15)   # 0.3333
(0 + 15) / 2                  # E(X) = 7.5
```

---

## Distribución Normal

La **distribución Normal** —la campana de Gauss— es la más importante del curso: aparece cuando muchas causas pequeñas e independientes se combinan para producir una variable (alturas, calificaciones, errores de medición). Se escribe $X \sim N(\mu, \sigma)$, con $\mu$ el centro de la campana y $\sigma$ su anchura:

$$
f(x) = \frac{1}{\sigma\sqrt{2\pi}} \cdot e^{-\frac{(x-\mu)^2}{2\sigma^2}}
$$

No es necesario evaluar esta densidad a mano — todo cálculo se hace con R o con la tabla Z. Sus propiedades: es simétrica alrededor de $\mu$; el área total es 1; $E(X)=\mu$ y $V(X)=\sigma^2$.

### La Regla 68-95-99.7

En cualquier Normal, sin importar $\mu$ y $\sigma$, la probabilidad de estar dentro de cierto número de desviaciones estándar de la media es siempre la misma:

| Intervalo    | Probabilidad | Interpretación                       |
| ------------- | ------------- | -------------------------------------- |
| $\mu \pm 1\sigma$ | 0.6827 (~68%) | Aproximadamente 2 de cada 3 valores    |
| $\mu \pm 2\sigma$ | 0.9545 (~95%) | La gran mayoría de los valores         |
| $\mu \pm 3\sigma$ | 0.9973 (~99.7%) | Prácticamente todos los valores      |

**Comandos de R:** `dnorm(x, mean, sd)`, `pnorm(x, mean, sd)`, `qnorm(p, mean, sd)`, `rnorm(n, mean, sd)`. En R, `sd` es la desviación estándar $\sigma$, no la varianza.

**Ejemplo:**

La altura de estudiantes de una universidad sigue $X \sim N(170, 8)$ cm. (a) ¿Probabilidad de que un estudiante mida más de 180 cm? (b) ¿Nota mínima para estar en el 10% superior de un curso con $N(75,10)$?

**Solución:**

$$
P(X>180) = 1 - P(X \le 180) \approx 0.1056
$$

**Verificación en R:**

```r
1 - pnorm(180, mean = 170, sd = 8)   # (a) 0.1056
qnorm(0.90, 75, 10)                  # (b) 87.82: umbral del 10% superior
```

---

## Normal Estándar y Estandarización (Z-score)

Cada Normal tiene su propia $\mu$ y $\sigma$. En vez de una tabla distinta para cada combinación de parámetros, se transforma cualquier problema a una única «Normal de referencia»: la **Normal estándar** $Z \sim N(0,1)$.

$$
Z = \frac{X-\mu}{\sigma}
$$

El valor $Z$ (z-score) se interpreta como «cuántas desviaciones estándar por encima o por debajo de la media está $X$». Permite comparar valores de distribuciones distintas (por ejemplo, notas de exámenes con escalas diferentes) y es la base de los intervalos de confianza y pruebas de hipótesis de la Unidad 2.

En R, `pnorm(z)` sin especificar `mean` ni `sd` asume la Normal estándar, y `qnorm(p)` invierte el proceso: dado un percentil, entrega el valor $z$ correspondiente.

**Ejemplo:**

Ana obtuvo 82 en un examen con media 75 y $\sigma=8$; Beto obtuvo 88 en otro con media 85 y $\sigma=6$. ¿Quién tuvo mejor desempeño relativo?

**Solución:**

$$
Z_{Ana} = \frac{82-75}{8} = 0.875 \qquad Z_{Beto} = \frac{88-85}{6} = 0.500
$$

**Verificación en R:**

```r
pnorm(0.875) * 100   # Ana: percentil 80.92
pnorm(0.500) * 100   # Beto: percentil 69.15
```

Aunque Beto tiene el puntaje bruto más alto, Ana tiene mejor desempeño relativo a su grupo: superó al 81% de sus compañeros, frente al 69% de Beto.

---

## Teorema Central del Límite (TCL)

El **Teorema Central del Límite** es el resultado más importante de la estadística: si se toman muchas muestras de tamaño $n$ de una variable $X$ (cualquiera que sea su distribución), el promedio de esas muestras se distribuye aproximadamente Normal cuando $n$ es suficientemente grande. Formalmente, si $X$ tiene media $\mu$ y desviación $\sigma$:

$$
\bar{X} \sim N\left(\mu, \frac{\sigma}{\sqrt{n}}\right)
$$

La media muestral se centra en la misma $\mu$, pero su desviación estándar —el **error estándar**— es $\sigma$ dividido entre $\sqrt{n}$: promediar reduce la variabilidad en un factor $\sqrt{n}$. Se considera "suficientemente grande" $n \ge 30$ para casi cualquier distribución original; si $X$ ya era Normal, el TCL aplica desde $n=1$.

Para una **suma** de $n$ observaciones (en vez de un promedio), el resultado análogo es $S \sim N(n\mu, \sqrt{n}\cdot\sigma)$.

**Ejemplo:**

Una fábrica produce clavos con peso individual de media 5 g y desviación 0.5 g. Se empacan en cajas de 100. ¿Probabilidad de que una caja pese más de 510 g?

**Solución:**

$$
S \sim N(100 \times 5, \sqrt{100}\times 0.5) = N(500, 5)
$$

**Verificación en R:**

```r
1 - pnorm(510, mean = 500, sd = 5)   # 0.0228
```

Solo el 2.28% de las cajas superará los 510 g — un cálculo clásico en control de calidad para fijar límites de rechazo aceptables.
