---
Fecha de creación: 2026-08-12T18:49:00
Materia:
  - Estadística Inferencial
Fecha de clase: 2026-08-12
---
[[Clase 04 - Cinco Distribuciones Continuas|← Clase anterior]] | [[Clase 07 - Estimación e Intervalos de Confianza|Clase siguiente →]]

# Síntesis Unidad 1

Esta clase cierra la Unidad 1. Hasta ahora cada ejercicio venía con el modelo ya dado; en la práctica real ese modelo no se conoce de antemano — hay datos, y hay que decidir cuál distribución los describe mejor y verificar esa decisión con evidencia. Este cierre reúne las diez distribuciones vistas, un criterio para elegir el modelo correcto, y la prueba de bondad de ajuste con chi-cuadrado, la primera prueba de hipótesis formal del curso.

---

## Síntesis de las Diez Distribuciones

### Cinco Discretas

| Distribución       | ¿Qué modela?                                    | Parámetros | Pista para identificarla                     |
| -------------------- | ------------------------------------------------- | ----------- | ----------------------------------------------- |
| Binomial            | Éxitos en $n$ intentos independientes           | $n, p$     | Hay un $n$ fijo; se cuentan éxitos             |
| Poisson             | Eventos en un intervalo de tiempo o espacio     | $\lambda$  | No hay $n$; se da una tasa promedio            |
| Geométrica          | Intentos hasta el primer éxito                  | $p$        | Se espera el primer éxito                       |
| Binomial Negativa   | Intentos hasta el r-ésimo éxito                 | $r, p$     | Se espera hasta $r$ éxitos                      |
| Hipergeométrica     | Éxitos en muestra sin reemplazo de población pequeña | $N, K, n$ | Muestreo sin reemplazo, población no enorme |

### Cinco Continuas

| Distribución            | ¿Qué modela?                                | Parámetros | Pista para identificarla                    |
| ------------------------ | --------------------------------------------- | ----------- | ---------------------------------------------- |
| Uniforme continua       | Cualquier valor en $(a,b)$ igualmente probable | $a, b$     | El enunciado dice "igual de probable" en un rango |
| Normal                  | Mediciones naturales alrededor de $\mu$      | $\mu, \sigma$ | Altura, peso, error, calificaciones, IQ, tiempo |
| Exponencial             | Tiempo hasta el próximo evento con tasa constante | $\lambda$ | Vida útil sin desgaste; tiempo entre llegadas |
| Gamma                   | Tiempo hasta el k-ésimo evento               | $k, \lambda$ | Esperar varios eventos, no solo uno          |
| χ², t, F (muestrales)   | Distribuciones de la inferencia estadística  | $df$ (y $df_1, df_2$ para F) | Valores críticos, p-valores, comparación de varianzas |

### Relaciones entre Distribuciones

- Exponencial y Poisson son dos perspectivas del mismo proceso: si los eventos por hora siguen Poisson($\lambda$), el tiempo entre eventos sigue Exponencial($\lambda$).
- La Exponencial es la Gamma con $k=1$; la Chi² también es un caso particular de la Gamma.
- La t se acerca a la Normal estándar cuando $df$ crece (con $df>30$, prácticamente indistinguibles).
- La F es una razón de dos Chi² divididas entre sus grados de libertad.
- Aproximación Poisson a la Binomial: si $n$ es grande y $p$ pequeño ($n\ge30$, $p\le0.05$), Binomial($n,p$) $\approx$ Poisson($np$).
- Aproximación Binomial a la Hipergeométrica: si $n/N < 5\%$, el muestreo sin reemplazo se comporta como con reemplazo.
- Aproximación Normal a la Binomial (por el TCL): si $np$ y $n(1-p)$ son ambos mayores que 5, Binomial($n,p$) $\approx N(np, \sqrt{np(1-p)})$.

---

## Cómo Elegir el Modelo Correcto

Ante un enunciado nuevo, tres preguntas en orden suelen resolver la identificación del modelo:

1. **¿La variable se cuenta o se mide?** Contar (éxitos, eventos, intentos) lleva a las cinco discretas; medir (tiempo, longitud, peso, dinero) lleva a las cinco continuas.
2. **¿Qué representa exactamente?** Si es discreta: ¿éxitos en $n$ intentos?, ¿eventos por unidad de tiempo?, ¿intentos hasta un éxito?, ¿muestreo sin reemplazo? Si es continua: ¿valor uniforme en un rango?, ¿medición natural?, ¿tiempo hasta un evento?, ¿un estadístico de inferencia?
3. **¿Qué parámetros da el enunciado?** $n$ y $p$ sugieren Binomial (o Binomial Negativa si se buscan $r$ éxitos); $\lambda$ como tasa sugiere Poisson (o Exponencial si se mide tiempo); solo $p$ sugiere Geométrica.

| Si el enunciado dice…                                  | Entonces la distribución es… |
| --------------------------------------------------------- | ------------------------------- |
| $n$ fijo, se cuentan éxitos                              | Binomial                        |
| Tasa $\lambda$ por unidad de tiempo o espacio, sin $n$   | Poisson                          |
| Se espera hasta el primer éxito                          | Geométrica                      |
| Se espera hasta el r-ésimo éxito                         | Binomial Negativa               |
| Muestreo sin reemplazo de población pequeña              | Hipergeométrica                 |
| Rango equiprobable $(a,b)$                               | Uniforme continua                |
| Medición natural (altura, peso, error, calificación)      | Normal                            |
| Tiempo hasta el próximo evento                            | Exponencial                      |
| Tiempo hasta el k-ésimo evento                            | Gamma                             |
| Prueba de hipótesis o valor crítico                      | χ², t o F según el caso           |

---

## Prueba de Bondad de Ajuste (χ²)

### La Idea Intuitiva

La prueba compara dos cosas categoría por categoría: las **frecuencias observadas** ($O$, lo que se contó en los datos) y las **frecuencias esperadas** ($E$, lo que el modelo teórico predice para ese tamaño de muestra). Si $O$ y $E$ son parecidos, el modelo es compatible con los datos; si son muy distintos, hay evidencia en contra del modelo.

### El Estadístico Chi-cuadrado

$$
\chi^2 = \sum \frac{(O-E)^2}{E}
$$

Se eleva al cuadrado para que las diferencias positivas y negativas no se cancelen y para penalizar más las diferencias grandes; se divide entre $E$ para normalizar por lo esperado (una diferencia de 2 sobre 10 pesa más que una diferencia de 2 sobre 100). Valores pequeños de $\chi^2$ indican compatibilidad con el modelo; valores grandes, evidencia en contra.

### Grados de Libertad

$$
df = k - 1 - m
$$

donde $k$ es el número de categorías y $m$ es el número de parámetros de la distribución estimados desde los datos. El $-1$ aparece porque las frecuencias observadas suman el total $n$; cada parámetro estimado resta un grado de libertad adicional.

### Los Cinco Pasos del Test

1. Plantear las hipótesis: $H_0$ — los datos siguen la distribución propuesta; $H_1$ — no la siguen.
2. Calcular las frecuencias esperadas $E_i = n \cdot P(\text{categoría } i)$ según el modelo (estimando parámetros si hace falta).
3. Verificar que todas las $E \ge 5$; si alguna no cumple, agrupar categorías vecinas.
4. Calcular $\chi^2 = \sum (O-E)^2/E$.
5. Comparar con el valor crítico $\chi^2(1-\alpha, df)$ y decidir — se rechaza $H_0$ si $\chi^2$ lo supera, o si el p-valor es menor que $\alpha$.

**Ejemplo — el dado sospechoso:**

Un dado se lanza 60 veces. Bajo $H_0$ (dado justo), se espera $E_i = 60/6 = 10$ en cada cara.

| Cara | 1 | 2 | 3 | 4 | 5 | 6 |
| ---- | - | - | - | - | - | - |
| $O$  | 8 | 12 | 9 | 11 | 13 | 7 |
| $E$  | 10 | 10 | 10 | 10 | 10 | 10 |

$$
\chi^2 = \frac{(8-10)^2}{10}+\frac{(12-10)^2}{10}+\dots = 0.4+0.4+0.1+0.1+0.9+0.9 = 2.8
$$

Con $df = 6-1-0=5$, el valor crítico al 5% es $\chi^2(0.95,5)=11.07$. Como $2.8 \ll 11.07$, no se rechaza $H_0$: los datos son compatibles con un dado justo.

**Verificación en R:**

```r
observado <- c(8, 12, 9, 11, 13, 7)
chisq.test(x = observado, p = rep(1/6, 6))
# X-squared = 2.8, df = 5, p-value = 0.7308
```

### El Requisito E ≥ 5 y el Agrupamiento

El test depende de una aproximación que solo funciona bien cuando las frecuencias esperadas son suficientemente grandes: la regla práctica exige $E_i \ge 5$ en todas las categorías. Cuando alguna no cumple, se agrupan categorías vecinas hasta lograrlo, lo que reduce $k$ y por lo tanto los grados de libertad. Nota importante: `chisq.test()` siempre reporta $df = k-1$, sin restar los parámetros estimados ($m$); si se estimó algún parámetro desde los datos, el p-valor correcto debe recalcularse manualmente con `1 - pchisq(chi2, df = k - 1 - m)`.

### "No Rechazar" no es "Demostrar"

Rechazar $H_0$ es una afirmación fuerte: hay evidencia estadística en contra del modelo. No rechazar $H_0$ es una afirmación débil: solo indica que los datos son compatibles con el modelo, no que el modelo sea correcto — con una muestra mayor podría detectarse un desajuste que la muestra actual no tiene poder para revelar. La formulación correcta es "los datos son consistentes con la hipótesis" o "no hay evidencia en contra", nunca "se demostró que el modelo es correcto".
