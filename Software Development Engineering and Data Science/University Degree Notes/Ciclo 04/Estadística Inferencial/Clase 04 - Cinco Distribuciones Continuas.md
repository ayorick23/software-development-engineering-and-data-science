---
Fecha de creación: 2026-07-29T18:48:00
Materia:
  - Estadística Inferencial
Fecha de clase: 2026-07-29
---
[[Clase 03 - Distribuciones de Probabilidad Continuas|← Clase anterior]] | [[Clase 05 - Síntesis Unidad 1|Clase siguiente →]]

# Cinco Distribuciones Continuas

Esta clase reúne cinco distribuciones continuas adicionales a las vistas en [[Clase 03 - Distribuciones de Probabilidad Continuas|la clase anterior]]. La Exponencial y la Gamma tienen aplicación directa en ingeniería (confiabilidad, teoría de colas); la Chi-cuadrado, la t de Student y la F de Snedecor son las tres **distribuciones muestrales** que sostienen toda la inferencia estadística de la Unidad 2.

| Distribución | ¿Qué modela? | Uso principal |
| ------------- | ------------- | --------------- |
| Exponencial   | Tiempo hasta el próximo evento | Confiabilidad, teoría de colas |
| Gamma         | Tiempo hasta el k-ésimo evento (generaliza la Exponencial) | Vida útil de sistemas, tiempos acumulados |
| Chi² (χ²)     | Suma de cuadrados de normales estándar | Bondad de ajuste, prueba de varianza, tablas de contingencia |
| t de Student  | Media muestral con $\sigma$ desconocida | Intervalos de confianza y pruebas de hipótesis sobre la media |
| F de Snedecor | Razón de dos Chi² independientes | Comparación de varianzas, ANOVA |

Relaciones a notar: la Exponencial es la Gamma con $k=1$; la Chi² es también un caso particular de la Gamma; la t se acerca a la Normal estándar cuando los grados de libertad crecen (con $df>30$ son casi idénticas); la F es una razón de dos Chi² divididas entre sus grados de libertad.

---

## Distribución Exponencial

La **distribución Exponencial** modela el tiempo hasta que ocurre el próximo evento en un proceso donde los eventos son independientes y ocurren a una tasa constante $\lambda$. Si el número de eventos por unidad de tiempo sigue Poisson($\lambda$), el tiempo entre eventos sigue Exponencial($\lambda$) — son dos caras del mismo proceso.

Se escribe $X \sim \text{Exp}(\lambda)$:

$$
f(x) = \lambda \cdot e^{-\lambda x}, \qquad x \ge 0
$$

$$
E(X) = \frac{1}{\lambda} \qquad V(X) = \frac{1}{\lambda^2}
$$

Es la única distribución continua con **falta de memoria**: dado que un componente ya lleva funcionando $t_0$, la probabilidad de que dure $s$ horas más es la misma que si acabara de empezar, $P(X>t_0+s \mid X>t_0) = P(X>s)$. Esto la hace apropiada para componentes electrónicos sin desgaste acumulativo, pero no para componentes que envejecen de forma gradual.

**Comandos de R:** `dexp(x, rate)`, `pexp(x, rate)`, `qexp(p, rate)`, `rexp(n, rate)`. R usa `rate` = $\lambda$, no la media: si se da la media (por ejemplo, "vida útil promedio 1000 horas"), $\lambda = 1/1000$.

**Ejemplo:**

A un call center llegan en promedio 5 llamadas por hora, $X \sim \text{Exp}(\lambda=5)$. ¿Probabilidad de que la próxima llamada tarde más de 20 minutos (1/3 hora)?

**Solución:**

$$
P(X>1/3) = 1 - \left(1-e^{-5/3}\right)
$$

**Verificación en R:**

```r
1 - pexp(1/3, rate = 5)   # 0.1889
```

---

## Distribución Gamma

La **distribución Gamma** modela el tiempo hasta el **k-ésimo** evento en un proceso de eventos independientes con tasa constante $\lambda$; generaliza la Exponencial, que es su caso particular con $k=1$. Se escribe $X \sim \text{Gamma}(\text{shape}=k, \text{rate}=\lambda)$:

$$
E(X) = \frac{k}{\lambda} \qquad V(X) = \frac{k}{\lambda^2}
$$

No hace falta memorizar su densidad: en la práctica se trata como una caja negra, identificando únicamente sus dos parámetros.

**Comandos de R:** `dgamma(x, shape, rate)`, `pgamma(x, shape, rate)`, `qgamma(p, shape, rate)`, `rgamma(n, shape, rate)`.

**Ejemplo:**

Un sistema opera con 4 componentes en serie, cada uno con vida Exponencial de media 100 horas. ¿Probabilidad de que el sistema opere más de 500 horas en total?

**Solución:**

Sea $T$ el tiempo total, $T \sim \text{Gamma}(\text{shape}=4, \text{rate}=1/100)$.

**Verificación en R:**

```r
1 - pgamma(500, shape = 4, rate = 1/100)   # 0.265
4 * 100                                     # E(X) = 400 horas
```

---

## Distribución Chi-cuadrado (χ²)

La **distribución Chi-cuadrado** aparece al sumar cuadrados de variables Normales estándar. Se usa principalmente en tres pruebas: bondad de ajuste (verificar si datos observados siguen una distribución teórica), prueba de independencia en tablas de contingencia, y prueba sobre la varianza poblacional. También describe la distribución muestral de $(n-1)s^2/\sigma^2$ cuando la población es Normal.

Se escribe $X \sim \chi^2(df)$, con $df$ los grados de libertad:

$$
E(X) = df \qquad V(X) = 2 \cdot df
$$

Es asimétrica hacia la derecha y solo toma valores positivos.

**Comandos de R:** `dchisq(x, df)`, `pchisq(x, df)`, `qchisq(p, df)`, `rchisq(n, df)`.

**Ejemplo:**

Valor crítico para una prueba de bondad de ajuste con 6 categorías ($df=5$) al nivel $\alpha=0.05$.

**Verificación en R:**

```r
qchisq(0.95, df = 5)   # 11.07
```

Si el estadístico $\chi^2$ calculado a partir de los datos supera 11.07, se rechaza la hipótesis nula al 5%.

---

## Distribución t de Student

La **distribución t de Student** se usa para hacer inferencia sobre la media de una población Normal cuando **no se conoce $\sigma$** — el caso habitual en la práctica, donde $\sigma$ se estima con la desviación muestral $s$. Al sustituir $\sigma$ por $s$ en el z-score, la distribución del cociente deja de ser Normal estándar y pasa a ser t de Student.

Se escribe $T \sim t(df)$, con $df = n-1$ para una muestra de tamaño $n$. Es simétrica alrededor de 0 como la Normal estándar, pero con colas más gruesas — más gruesas cuanto menor sea $df$.

Regla práctica: usar t siempre que $\sigma$ sea desconocida (casi siempre). Con $n>30$, t y Z son prácticamente indistinguibles.

**Comandos de R:** `dt(x, df)`, `pt(x, df)`, `qt(p, df)`, `rt(n, df)`.

**Ejemplo:**

Valor crítico para un intervalo de confianza del 95% con una muestra de tamaño $n=15$ ($df=14$).

**Verificación en R:**

```r
qt(0.975, df = 14)   # 2.1448
qnorm(0.975)         # 1.9600  (valor Z equivalente, para comparar)
```

Como $2.1448 > 1.96$, el intervalo con t es más ancho: refleja la incertidumbre adicional de no conocer $\sigma$. Conforme $n$ crece, el valor crítico t converge al de la Normal.

---

## Distribución F de Snedecor

La **distribución F de Snedecor** se usa principalmente para comparar varianzas de dos poblaciones normales (prueba de razón de varianzas), en análisis de varianza (ANOVA) para comparar las medias de tres o más grupos, y en regresión múltiple. Se escribe $X \sim F(df_1, df_2)$, con $df_1$ (numerador) y $df_2$ (denominador); el orden importa, $F(5,10) \ne F(10,5)$. Es asimétrica hacia la derecha y solo toma valores positivos.

**Comandos de R:** `df(x, df1, df2)`, `pf(x, df1, df2)`, `qf(p, df1, df2)`, `rf(n, df1, df2)`.

**Ejemplo:**

Se comparan las varianzas de dos muestras con $n_1=9$, $n_2=11$ ($df_1=8$, $df_2=10$). Valor crítico al nivel $\alpha=0.05$.

**Verificación en R:**

```r
qf(0.95, df1 = 8, df2 = 10)   # 3.0717
```

Si la razón $s_1^2/s_2^2$ supera 3.07, hay evidencia al 5% de que las varianzas poblacionales son distintas.

---

## Resumen: Las Cinco Distribuciones Continuas

| Distribución  | Parámetros                | En R              | $E(X)$          | Se usa para…                              |
| ------------- | ------------------------- | ----------------- | --------------- | ----------------------------------------- |
| Exponencial   | rate = $\lambda$          | `dexp`/`pexp`     | $1/\lambda$     | Tiempo hasta el próximo evento            |
| Gamma         | shape=$k$, rate=$\lambda$ | `dgamma`/`pgamma` | $k/\lambda$     | Tiempo hasta el k-ésimo evento            |
| Chi² (χ²)     | $df$                      | `dchisq`/`pchisq` | $df$            | Bondad de ajuste, independencia, varianza |
| t de Student  | $df$                      | `dt`/`pt`         | $0$             | Media con $\sigma$ desconocida            |
| F de Snedecor | $df_1, df_2$              | `df`/`pf`         | $df_2/(df_2-2)$ | Razón de varianzas, ANOVA                 |

Cinco reglas prácticas: si dan una tasa de eventos por unidad de tiempo, es Exponencial; si preguntan por el tiempo hasta $k$ eventos, es Gamma; si el problema menciona bondad de ajuste o tablas de contingencia, es χ²; si se trabaja con la media y la varianza poblacional es desconocida, es t; si se comparan dos varianzas o se hace ANOVA, es F.
