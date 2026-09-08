---
Fecha de creación: 2026-08-26T18:24:00
Materia:
  - Estadística Inferencial
Fecha de clase: 2026-09-02
---
[[Clase 07 - Estimación e Intervalos de Confianza|← Clase anterior]]

# IC para Varianza y Proporciones

Esta clase extiende los intervalos de confianza vistos en [[Clase 07 - Estimación e Intervalos de Confianza|la clase anterior]] a otros dos parámetros: la varianza (y con ella la desviación estándar) y la proporción. La anatomía se mantiene —estimación puntual ± valor crítico × error estándar—, salvo por el intervalo de varianza, que no es simétrico.

| Parámetro                  | Estimador | Distribución del valor crítico | ¿Simétrico? |
| ---------------------------- | ----------- | --------------------------------- | ------------- |
| Media $\mu$ ($\sigma$ conocida)  | $\bar{x}$ | Normal (Z)                       | Sí            |
| Media $\mu$ ($\sigma$ desconocida) | $\bar{x}$ | t de Student, $df=n-1$          | Sí            |
| Varianza $\sigma^2$          | $s^2$     | Chi-cuadrado, $df=n-1$            | No            |
| Proporción $p$               | $\hat{p}$ | Normal (Z)                       | Sí            |

---

## IC para la Varianza y la Desviación Estándar

Estimar la varianza responde a una pregunta distinta de la media: no "¿cuánto vale en promedio?" sino "¿qué tan consistente es el proceso?". Es central en control de calidad industrial, precisión de instrumentos, riesgo financiero (la volatilidad es la desviación estándar de un activo) y tiempos de servicio.

### Por qué el Intervalo no es Simétrico

Para la media, el intervalo se construye como $\bar{x} \pm$ margen porque la distribución de referencia (Normal o t) es simétrica alrededor de cero. La distribución [[Clase 04 - Cinco Distribuciones Continuas#Distribución Chi-cuadrado (χ²)|Chi-cuadrado]], que gobierna el comportamiento muestral de $s^2$, solo toma valores positivos y está sesgada hacia la derecha — no hay simetría que aprovechar. Además, la varianza no puede ser negativa: un intervalo simétrico alrededor de un $s^2$ pequeño podría producir un límite inferior negativo. Por eso el intervalo se construye con dos cuantiles distintos de la chi-cuadrado, uno para cada extremo, y no queda centrado en $s^2$.

### La Fórmula

$$
\frac{(n-1)s^2}{\chi^2_{\text{sup}}} < \sigma^2 < \frac{(n-1)s^2}{\chi^2_{\text{inf}}}
$$

donde, para un nivel de confianza $(1-\alpha)$: $\chi^2_{\text{sup}}$ es el cuantil $1-\alpha/2$ (`qchisq(1 - alpha/2, n-1)`) y $\chi^2_{\text{inf}}$ es el cuantil $\alpha/2$ (`qchisq(alpha/2, n-1)`). El límite inferior se calcula dividiendo entre el cuantil **superior**, y el límite superior dividiendo entre el cuantil **inferior** — están cruzados, porque dividir entre un número mayor da un resultado menor. Para la desviación estándar basta tomar la raíz cuadrada de ambos límites, ya que la raíz cuadrada preserva el orden.

**Ejemplo:**

Una máquina llenadora se somete a control: en 20 botellas, la desviación estándar muestral es $s=2.4$ ml. Construir IC del 95% para $\sigma^2$ y $\sigma$.

**Solución:**

```r
n <- 20; s <- 2.4
df <- n - 1                       # 19
chi_sup <- qchisq(0.975, df)      # 32.8523
chi_inf <- qchisq(0.025, df)      #  8.9065
li <- df * s^2 / chi_sup          #  3.3313
ls <- df * s^2 / chi_inf          # 12.2876
sqrt(c(li, ls))                   # 1.8252  3.5054
```

IC 95% para $\sigma^2$ = (3.33, 12.29) ml²; IC 95% para $\sigma$ = (1.83, 3.51) ml. Nótese la asimetría: la estimación puntual es $s^2=5.76$, pero el límite inferior está a 2.43 unidades por debajo mientras el superior está a 6.53 por encima — la cola derecha es casi tres veces más larga.

### Sin Función Directa en R

A diferencia de `t.test()` para la media, R no incluye una función base que produzca directamente el intervalo para la varianza; se construye con los pasos anteriores, por ejemplo envueltos en una función propia:

```r
ic_varianza <- function(datos, conf = 0.95) {
  n  <- length(datos)
  s2 <- var(datos)
  df <- n - 1
  alpha <- 1 - conf
  li <- df * s2 / qchisq(1 - alpha/2, df)
  ls <- df * s2 / qchisq(alpha/2, df)
  c(varianza_inf = li, varianza_sup = ls,
    desviacion_inf = sqrt(li), desviacion_sup = sqrt(ls))
}
```

### El Supuesto de Normalidad

El intervalo para la media tolera desviaciones de la normalidad gracias al Teorema Central del Límite, siempre que $n$ sea grande. El intervalo para la varianza **no** goza de esa protección: depende de que la población sea Normal y es sensible a colas pesadas o valores atípicos, sin importar el tamaño de muestra. Antes de reportarlo conviene verificar el supuesto:

```r
shapiro.test(datos)           # H0: los datos provienen de una Normal
qqnorm(datos); qqline(datos)  # inspección visual
```

---

## IC para una Proporción

Muchas preguntas prácticas son sobre porcentajes (satisfacción, defectos, abandono) más que sobre promedios. El parámetro es $p$, la proporción real de éxitos en la población, y su estimador es la proporción muestral:

$$
\hat{p} = \frac{x}{n}
$$

donde $x$ es el número de éxitos observados. $x$ sigue una distribución Binomial($n,p$), y por el TCL, cuando $n$ es suficientemente grande, $\hat{p}$ se distribuye aproximadamente Normal — de ahí que el valor crítico vuelva a salir de la Normal:

$$
\hat{p} \pm z \cdot \sqrt{\frac{\hat{p}(1-\hat{p})}{n}}
$$

El producto $\hat{p}(1-\hat{p})$ alcanza su máximo en $\hat{p}=0.5$ y disminuye conforme la proporción se acerca a 0 o a 1: a igual tamaño de muestra, las proporciones extremas se estiman con mayor precisión que las intermedias.

### Condiciones de Validez

El intervalo se apoya en una aproximación Normal a la Binomial, que requiere suficientes casos de ambas categorías:

$$
n\hat{p} \ge 10 \qquad \text{y} \qquad n(1-\hat{p}) \ge 10
$$

Si no se cumplen (por ejemplo, 2 defectuosos en 300 piezas), la fórmula puede producir un límite inferior negativo — imposible para una proporción — y conviene usar un método exacto como `binom.test()`.

**Ejemplo:**

Una encuesta a 400 clientes registra 148 satisfechos. Construir IC del 95% para la proporción de clientes satisfechos.

**Solución:**

```r
n <- 400; x <- 148
p_hat <- x / n                        # 0.3700
n * p_hat; n * (1 - p_hat)            # 148 y 252, ambos >= 10: OK
ee     <- sqrt(p_hat * (1 - p_hat) / n)   # 0.0241
z      <- qnorm(0.975)                     # 1.9600
margen <- z * ee                           # 0.0473
c(p_hat - margen, p_hat + margen)
# 0.3227  0.4173
```

IC 95% = (32.3%, 41.7%), con margen de error de ±4.7 puntos porcentuales.

### Tres Métodos en R

`prop.test()` y `binom.test()` no reproducen exactamente el cálculo manual anterior, porque usan métodos distintos:

- **Wald** (el cálculo manual): el más simple, funciona bien con muestras grandes y proporciones alejadas de 0 y 1, pero se comporta mal en los extremos.
- **`prop.test()`** — método de Wilson con corrección por continuidad: más preciso que Wald, especialmente con muestras pequeñas; es el que usa R por defecto.
- **`binom.test()`** — método exacto de Clopper-Pearson: no usa aproximación Normal, se basa directamente en la Binomial. Es el más conservador y válido incluso cuando fallan las condiciones $n\hat{p}\ge10$.

```r
prop.test(148, 400)$conf.int   # 0.3229  0.4196
binom.test(148, 400)$conf.int  # 0.3226  0.4194
```

---

## Determinación del Tamaño de Muestra

### Recordatorio: Media

$$
n \ge \left(\frac{z \cdot \sigma}{E}\right)^2
$$

### Para una Proporción

El margen de error de una proporción es $E = z\sqrt{p(1-p)/n}$; despejando:

$$
n \ge \frac{z^2 \cdot p(1-p)}{E^2}
$$

La fórmula necesita $p$, que es justamente lo que se busca estimar. Dos salidas: usar una estimación previa (de un estudio piloto o datos históricos, la opción más eficiente), o asumir el **peor caso** $p=0.5$ cuando no hay información previa —como $p(1-p)$ alcanza su máximo en 0.5, esa elección produce el tamaño de muestra más grande posible, el que garantiza el margen deseado sea cual sea el valor real de $p$:

$$
n \ge \left(\frac{z}{2E}\right)^2
$$

**Ejemplo:**

Estimar la proporción de hogares con banda ancha en un municipio, margen de error no mayor a 3 puntos, 95% de confianza.

```r
z <- qnorm(0.975); E <- 0.03

# Caso A: sin información previa (peor caso)
n <- (z / (2 * E))^2       # 1067.0719
ceiling(n)                 # 1068

# Caso B: estudio previo sugiere p ~ 0.20
n <- z^2 * 0.20 * 0.80 / E^2   # 682.9260
ceiling(n)                     # 683
```

Con información previa se requieren 683 hogares en vez de 1068 — un 36% menos. Reducir el margen a la mitad cuadruplica la muestra (el mismo efecto del $\sqrt{n}$ visto para la media); subir la confianza es comparativamente más barato: pasar del 95% al 99% aumenta la muestra un 73%, mientras que reducir el margen de 3 a 2 puntos la aumenta un 125%.
