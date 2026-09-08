---
Fecha de creación: 2026-07-15T18:11:00
Materia:
  - Estadística Inferencial
Fecha de clase: 2026-07-15
---
[[Clase 01 - Distribuciones de Probabilidad Discretas - Introducción a R|← Clase anterior]] | [[Clase 03 - Distribuciones de Probabilidad Continuas|Clase siguiente →]]

# Distribuciones de Probabilidad Discretas

Esta clase continúa con las distribuciones discretas iniciadas en [[Clase 01 - Distribuciones de Probabilidad Discretas - Introducción a R|la clase anterior]] (Binomial y Poisson). Las tres distribuciones que siguen —Geométrica, Binomial Negativa e Hipergeométrica— nacen de modificar una sola pieza del escenario Binomial: en lugar de fijar el número de ensayos y contar éxitos, se fija el número de éxitos buscados y se cuenta cuántos ensayos hacen falta, o bien se elimina el reemplazo entre extracciones.

| Distribución       | Qué cambia respecto a la Binomial               | Pregunta que responde                  |
| ------------------ | ------------------------------------------------ | --------------------------------------- |
| Binomial            | — (n ensayos fijos, se cuentan éxitos)            | ¿Cuántos éxitos en n ensayos?           |
| Geométrica          | Ya no hay n fijo: se repite hasta el primer éxito | ¿Cuántos ensayos hasta el primer éxito? |
| Binomial Negativa   | Se repite hasta el r-ésimo éxito                  | ¿Cuántos ensayos hasta r éxitos?         |
| Hipergeométrica     | Extracción sin reemplazo: cae la independencia    | ¿Cuántos éxitos en n extracciones de una población finita? |

---

## Distribución Geométrica

La **distribución Geométrica** modela el número de ensayos tipo Bernoulli (dos resultados, probabilidad de éxito $p$ constante, independencia) que se necesitan hasta obtener el **primer éxito**. Si $X$ es el número del ensayo en el que ocurre ese primer éxito, sus valores posibles son $1, 2, 3, \dots$, sin límite superior.

### Función de Probabilidad

Para que la primera venta ocurra en el ensayo $k$, los primeros $k-1$ ensayos deben ser fracasos y el $k$-ésimo debe ser éxito. Existe un único camino posible para cada valor de $k$ (a diferencia de la Binomial, donde varios órdenes producían el mismo número de éxitos):

$$
P(X=k) = (1-p)^{k-1} \cdot p, \qquad k = 1, 2, 3, \dots
$$

Como cada valor de $k$ multiplica un factor adicional $(1-p) < 1$, las probabilidades siempre decrecen: el valor más probable es **siempre $k=1$**, sin importar qué tan pequeño sea $p$. Esto no contradice que la espera promedio pueda ser alta: la cola larga de rachas poco frecuentes pero posibles es la que eleva el promedio, aunque cada valor individual grande sea poco probable. Media y moda son conceptos distintos.

### Valor Esperado y Varianza

$$
E(X) = \frac{1}{p} \qquad V(X) = \frac{1-p}{p^2}
$$

La intuición: si el éxito ocurre 1 de cada $1/p$ veces en promedio, la espera del primer éxito cuesta, en promedio, ese mismo espaciado.

### Falta de Memoria

La Geométrica es la única distribución discreta sin memoria: si ya se acumulan $t$ fracasos, la probabilidad de que la espera se extienda $k$ ensayos más es la misma que al inicio del proceso, porque cada ensayo es independiente del anterior. La probabilidad de que la espera supere $k$ ensayos es:

$$
P(X > k) = (1-p)^k
$$

Esta propiedad desmonta la llamada falacia del jugador: tras una racha de fracasos, el próximo ensayo no está «más obligado» a ser éxito — el proceso se reinicia en cada intento.

**Ejemplo:**

Un vendedor cierra el 20% de sus llamadas ($p = 0.2$). (a) ¿Probabilidad de que la primera venta ocurra exactamente en la cuarta llamada? (b) ¿Probabilidad de necesitar más de 5 llamadas? (c) Número esperado de llamadas hasta la primera venta.

**Solución:**

- (a) $P(X=4) = 0.8^3 \cdot 0.2 = 0.1024$
- (b) $P(X>5) = 0.8^5 = 0.32768$
- (c) $E(X) = 1/0.2 = 5$ llamadas

**Verificación en R:**

Las funciones `dgeom`/`pgeom`/`rgeom` de R no cuentan ensayos, sino **fracasos antes del primer éxito** ($Y = X - 1$). Para $P(X=k)$ se usa `dgeom(k - 1, p)`.

```r
dgeom(4 - 1, prob = 0.2)   # (a) 0.1024  <- k - 1 fracasos
0.8^5                      # (b) 0.32768
1 / 0.2                    # (c) E(X) = 5
```

---

## Distribución Binomial Negativa

La **distribución Binomial Negativa** generaliza la Geométrica: en lugar de esperar el primer éxito, se espera el **r-ésimo éxito**. Es su caso particular con $r=1$. Sea $X$ el número del ensayo en el que ocurre ese r-ésimo éxito; sus valores posibles son $r, r+1, r+2, \dots$

### Función de Probabilidad

Para que el r-ésimo éxito ocurra en el ensayo $k$, la posición $k$ debe ser éxito (forzada), y entre los primeros $k-1$ ensayos debe haber exactamente $r-1$ éxitos en cualquier orden — un conteo de $C(k-1, r-1)$ caminos:

$$
P(X=k) = \binom{k-1}{r-1} \cdot p^r \cdot (1-p)^{k-r}, \qquad k = r, r+1, \dots
$$

El coeficiente es $\binom{k-1}{r-1}$ y no $\binom{k}{r}$: la posición final está forzada a ser éxito y no participa del acomodo libre de los $r-1$ éxitos restantes.

### Valor Esperado y Varianza

$$
E(X) = \frac{r}{p} \qquad V(X) = \frac{r(1-p)}{p^2}
$$

Esperar $r$ éxitos equivale a esperar el primero, luego el segundo desde cero (por la falta de memoria), y así sucesivamente: las esperas promedio se suman.

**Ejemplo:**

El mismo vendedor ($p=0.2$) tiene una meta de $r=3$ ventas. ¿Cuál es la probabilidad de que la tercera venta ocurra exactamente en la llamada 10? ¿Cuántas llamadas se esperan para la meta?

**Solución:**

$$
P(X=10) = \binom{9}{2} \cdot 0.2^3 \cdot 0.8^7 = 36 \times 0.008 \times 0.2097 \approx 0.0604
$$

$E(X) = 3/0.2 = 15$ llamadas.

**Verificación en R:**

`dnbinom` también cuenta fracasos ($k - r$), y el parámetro $r$ se llama `size`.

```r
dnbinom(10 - 3, size = 3, prob = 0.2)   # P(X = 10) = 0.06040
3 / 0.2                                 # E(X) = 15 llamadas
```

---

## Distribución Hipergeométrica

La **distribución Hipergeométrica** aparece cuando se extrae una muestra **sin reemplazo** de una población finita, lo que elimina la independencia entre extracciones y hace que $p$ deje de ser constante — el escenario que la Binomial no puede modelar (por ejemplo, contar ases al sacar cartas de una baraja). Tiene tres parámetros: $N$ (tamaño de la población), $K$ (número de éxitos en la población) y $n$ (tamaño de la muestra extraída). $X$ es el número de éxitos entre los extraídos.

### Función de Probabilidad

Al no haber independencia, no se puede usar la regla del producto; se cuenta con la razón de casos favorables entre casos totales, ya que cada grupo posible de $n$ elementos es igualmente probable:

$$
P(X=k) = \frac{\binom{K}{k} \cdot \binom{N-K}{n-k}}{\binom{N}{n}}
$$

### Valor Esperado y Varianza

$$
E(X) = n \cdot \frac{K}{N} \qquad V(X) = n \cdot \frac{K}{N} \cdot \left(1 - \frac{K}{N}\right) \cdot \frac{N-n}{N-1}
$$

La esperanza coincide con la de una Binomial equivalente ($p = K/N$), pero la varianza se reduce por el **factor de corrección para población finita** $(N-n)/(N-1)$, siempre menor que 1: extraer sin reemplazo recorta los escenarios extremos porque cada extracción «informa» sobre las siguientes.

**Ejemplo:**

Una caja trae 20 productos, 6 defectuosos. Se extraen 5 sin reemplazo. ¿Probabilidad de exactamente 2 defectuosos? ¿Número esperado de defectuosos en la muestra?

**Solución:**

$$
P(X=2) = \frac{\binom{6}{2}\binom{14}{3}}{\binom{20}{5}} = \frac{15 \times 364}{15504} \approx 0.35217
$$

$E(X) = 5 \times 6/20 = 1.5$

**Verificación en R:**

`dhyper(x, m, n, k)` usa nombres poco intuitivos: `m` = éxitos en la población ($K$), `n` = fracasos en la población ($N-K$), `k` = tamaño de la muestra.

```r
dhyper(2, m = 6, n = 14, k = 5)   # 0.35217
5 * 6/20                          # E(X) = 1.5
```

### Aproximación por la Binomial

Cuando la muestra es pequeña frente a la población (regla práctica: $n < 5\% \cdot N$), extraer sin reemplazo apenas altera las probabilidades y la Binomial con $p = K/N$ aproxima bien a la Hipergeométrica — el factor de corrección se acerca a 1.

---

## Simulación en R

Cada distribución de R tiene una tercera función, la que empieza con `r`, que genera datos que obedecen la distribución. Simular sirve para verificar cálculos analíticos, observar la frecuencia a largo plazo en acción, y resolver problemas donde la fórmula exacta es difícil de evaluar.

Toda simulación reproducible arranca fijando la semilla con `set.seed()`, de modo que la secuencia pseudoaleatoria sea la misma en cualquier ejecución. El patrón general es **simular, contar, comparar**:

```r
set.seed(2026)

# 100,000 dias del vendedor: en que llamada cae la primera venta
esperas <- rgeom(100000, prob = 0.2) + 1   # +1: rgeom da fracasos
mean(esperas)        # ~5: confirma E(X) = 1/p
mean(esperas == 4)   # ~0.1024: confirma P(X = 4)

# 100,000 cajas inspeccionadas (hipergeometrica)
defectuosos <- rhyper(100000, m = 6, n = 14, k = 5)
mean(defectuosos == 2)   # ~0.352
```

La comparación `mean(esperas == 4)` funciona porque el promedio de un vector de `TRUE`/`FALSE` es exactamente la proporción de veces que ocurrió el evento — la definición de frecuencia relativa expresada en código.

---

## Comparación: Las Cinco Distribuciones Discretas

| Distribución       | Qué es aleatorio                    | Parámetros | $E(X)$ | En R                                          |
| ------------------- | ------------------------------------- | ---------- | ------ | ---------------------------------------------- |
| Binomial            | N.º de éxitos en $n$ ensayos          | $n, p$     | $np$   | `dbinom(k, n, p)`                              |
| Poisson             | N.º de eventos en un intervalo        | $\lambda$  | $\lambda$ | `dpois(k, lambda)`                           |
| Geométrica          | N.º del ensayo del 1.º éxito          | $p$        | $1/p$  | `dgeom(k-1, p)` — cuenta fracasos              |
| Binomial Negativa   | N.º del ensayo del r-ésimo éxito      | $r, p$     | $r/p$  | `dnbinom(k-r, size=r, p)` — cuenta fracasos    |
| Hipergeométrica     | N.º de éxitos sin reemplazo           | $N, K, n$  | $nK/N$ | `dhyper(k, m=K, n=N-K, k=n)`                   |

El diagnóstico ante un enunciado nuevo se resuelve con dos preguntas: (1) ¿qué está fijo y qué es aleatorio? — si se fijan los ensayos y se cuentan éxitos, es Binomial (o Hipergeométrica); si se fijan los éxitos y se cuentan ensayos, es Geométrica ($r=1$) o Binomial Negativa ($r>1$); si no hay ensayos sino un flujo por intervalo, es Poisson. (2) ¿hay reemplazo o independencia? — solo aplica al primer caso: con reemplazo o población infinita, Binomial; sin reemplazo y población finita, Hipergeométrica.
