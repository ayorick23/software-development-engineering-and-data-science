---
Fecha de creación: 2026-08-26T18:24:00
Materia:
  - Estadística Inferencial
Fecha de clase: 2026-09-02
---
[[Clase 05 - Síntesis Unidad 1|← Clase anterior]] | [[Clase 08 - IC para Varianza y Proporciones|Clase siguiente →]]

# Estimación e Intervalos de Confianza

La estimación estadística y los intervalos de confianza permiten calcular un parámetro desconocido de una población a partir de los datos obtenidos de una muestra.

---

## ¿Qué es un Estimador?

Un **estimador** es una fórmula que toma los datos muestrales y produce un único número como aproximación de un parámetro poblacional desconocido. El resultado de aplicar esa fórmula se llama **estimación puntual**.

La notación distingue ambos mundos con claridad:

| **Concepto**        | **Población (desconocido)** | **Muestra (calculable)** | **En R**             |
| ------------------- | --------------------------- | ------------------------ | -------------------- |
| Media               | $μ$ (mu)                    | $x̄$ (equis barra)       | ``mean(datos)``      |
| Desviación estándar | $σ$ (sigma)                 | $s$                      | ``sd(datos)``        |
| Varianza            | $σ²$                        | $s²$                     | ``var(datos)``       |
| Proporción          | $p$                         | $p̂$ (pe gorro)          | ``mean(datos == 1)`` |

> [!NOTE] La regla mnemotécnica es simple: **letras griegas para la población, letras latinas para la muestra**. El sombrero (^) sobre un símbolo indica «esto es una estimación de».

La media muestral estima la media poblacional, la desviación muestral estima la desviación poblacional, la proporción muestral estima la proporción poblacional.

### Un Estimador es una Variable Aleatoria

Si se toma una muestra de 40 servidores y se calcula la media, se obtiene un número; otra muestra de 40 servidores de la misma población produce un número distinto. La media muestral $\bar{x}$ no es un valor fijo: cambia de muestra en muestra, y como tal es una variable aleatoria con su propia distribución, la **distribución muestral**.

Por el [[Clase 03 - Distribuciones de Probabilidad Continuas#Teorema Central del Límite (TCL)|Teorema Central del Límite]], si $n$ es suficientemente grande (o la población ya es Normal), la media muestral se distribuye aproximadamente Normal, centrada en $\mu$ y con desviación $\sigma/\sqrt{n}$. Esa desviación tiene nombre propio y es la pieza central de todo lo que sigue.

### El Error Estándar

$$
\text{Error estándar} = \frac{\sigma}{\sqrt{n}}
$$

El error estándar mide cuánto varía la media muestral de una muestra a otra — es la desviación estándar de $\bar{x}$, no la de las observaciones individuales. $\sigma$ (o $s$) describe la dispersión de los datos individuales; $\sigma/\sqrt{n}$ describe la dispersión de la media muestral, es decir, qué tan distinto sería el resultado si se repitiera el estudio con otra muestra.

El $\sqrt{n}$ en el denominador explica por qué muestras grandes producen estimaciones más confiables, y también por qué ese efecto tiene rendimientos decrecientes: para reducir el error estándar a la mitad no basta con duplicar la muestra, hay que cuadruplicarla. Cuando $\sigma$ es desconocida —el caso habitual— se estima con $s$, y el error estándar se calcula como $s/\sqrt{n}$.

### Por qué un Solo Número no Basta

Reportar «el tiempo de respuesta promedio es 250 ms» no dice nada sobre la precisión de esa afirmación: no distingue entre una estimación basada en pocas observaciones muy dispersas y otra basada en miles de observaciones consistentes. Además, la probabilidad de que $\bar{x}$ sea exactamente igual a $\mu$ es prácticamente cero — la estimación puntual está casi con certeza equivocada, solo que no se sabe por cuánto. La solución es reportar un rango acompañado de una declaración explícita sobre su confiabilidad: un **intervalo de confianza**.

---

## Intervalos de Confianza: la Idea

### La Estructura Común

Todos los intervalos de confianza —para la media, la varianza, o proporciones— comparten la misma anatomía:

$$
\text{estimación puntual} \pm \text{margen de error}
$$

$$
\text{margen de error} = \text{valor crítico} \times \text{error estándar}
$$

La estimación puntual ($\bar{x}$) sitúa el centro del intervalo; el error estándar mide la incertidumbre inherente al muestreo; el valor crítico traduce el nivel de confianza exigido en un multiplicador. Lo único que cambia entre situaciones es de dónde sale el valor crítico y cómo se calcula el error estándar.

### Qué Significa "95% de Confianza"

El nivel de confianza es la proporción de intervalos que capturarían el verdadero $\mu$ si el estudio se repitiera muchísimas veces — es una propiedad del procedimiento, no del intervalo particular obtenido. Una simulación de 10,000 muestras de tamaño 20 de una población con $\mu=100$ conocida, construyendo el intervalo del 95% de cada una, produce una tasa de captura cercana al 95%:

```r
set.seed(2026)
mu <- 100; sigma <- 15; n <- 20; M <- 10000
capturas <- 0
for (i in 1:M) {
  muestra <- rnorm(n, mu, sigma)
  t_crit  <- qt(0.975, n - 1)
  ee      <- sd(muestra) / sqrt(n)
  if (abs(mean(muestra) - mu) <= t_crit * ee) capturas <- capturas + 1
}
capturas / M   # ~0.9498
```

De los 10,000 intervalos, cerca del 95% contenía el verdadero valor de $\mu$. Esa es la naturaleza exacta de la garantía: el método acierta el 95% de las veces a la larga; no dice nada sobre si un intervalo específico es uno de los que aciertan.

---

## IC para la Media con σ Conocida

Este caso se presenta cuando la desviación estándar poblacional se conoce de antemano —por registros históricos extensos o especificación técnica del instrumento de medición.

$$
\bar{x} \pm z \cdot \frac{\sigma}{\sqrt{n}}
$$

El valor crítico $z$ sale de la Normal estándar:

| Nivel de confianza | Valor crítico $z$ | En R |
| -------------------- | -------------------- | ------ |
| 90%                  | 1.645                | `qnorm(0.95)` |
| 95%                  | 1.960                | `qnorm(0.975)` |
| 99%                  | 2.576                | `qnorm(0.995)` |

Para un intervalo bilateral del 95%, el 5% restante se reparte entre las dos colas (2.5% de cada lado), por lo que se pide el cuantil 0.975, no 0.95.

**Ejemplo:**

Un equipo mide el tiempo de respuesta de 40 servidores y obtiene una media de 250 ms; por registros históricos, $\sigma = 30$ ms. Construir un IC del 95% para el tiempo de respuesta promedio poblacional.

**Solución:**

```r
ee     <- 30 / sqrt(40)       # 4.7434
z      <- qnorm(0.975)        # 1.9600
margen <- z * ee              # 9.2969
c(250 - margen, 250 + margen)
# 240.7031  259.2969
```

IC 95% = (240.70, 259.30) ms. Con 95% de confianza, el tiempo de respuesta promedio de la población de servidores se encuentra en ese rango. Mantener el mismo tamaño de muestra y variar solo el nivel de confianza ilustra el intercambio inevitable: más confianza exige un intervalo más ancho (90% → ancho 15.60; 95% → 18.59; 99% → 24.44).

---

## IC para la Media con σ Desconocida

En la práctica casi nunca se conoce $\sigma$: si se supiera, probablemente ya se conocería también $\mu$. La solución es estimar $\sigma$ con la desviación muestral $s$, lo que introduce una fuente adicional de incertidumbre — usar la Normal como si $\sigma$ fuera conocida subestimaría esa incertidumbre y produciría intervalos demasiado estrechos. La corrección es usar la distribución [[Clase 04 - Cinco Distribuciones Continuas#Distribución t de Student|t de Student]] con $n-1$ grados de libertad, cuya cola más gruesa compensa la incertidumbre adicional:

$$
\bar{x} \pm t_{(n-1)} \cdot \frac{s}{\sqrt{n}}
$$

**Ejemplo:**

Se miden 15 instancias de un microservicio y se obtienen los siguientes consumos de memoria (MB):

```r
consumo <- c(512, 498, 530, 545, 505, 520, 488,
             535, 512, 525, 499, 540, 518, 507, 530)
```

**Solución:**

```r
n  <- length(consumo)        # 15
xb <- mean(consumo)          # 517.6000
s  <- sd(consumo)            # 16.6124
df <- n - 1                  # 14
t  <- qt(0.975, df)          # 2.1448
ee <- s / sqrt(n)            # 4.2893
margen <- t * ee             # 9.1996
c(xb - margen, xb + margen)
# 508.4004  526.7996
```

IC 95% = (508.40, 526.80) MB. Usar Z en lugar de t en este caso produciría un intervalo (509.19, 526.01) aproximadamente un 9% más estrecho — una precisión falsa que no alcanza la cobertura del 95% declarada. La diferencia entre t y Z se atenúa conforme $n$ crece (con $n>100$ es prácticamente nula), pero usar t siempre que $\sigma$ sea desconocida nunca está mal.

**Con R:** la función `t.test()` construye el intervalo completo en un solo paso.

```r
t.test(consumo)$conf.int              # 508.4004 526.7996
t.test(consumo, conf.level = 0.99)$conf.int
```

---

## Factores que Afectan el Ancho del Intervalo

Un intervalo angosto es un intervalo informativo. Tres factores determinan su ancho:

- **Tamaño de muestra ($n$):** es la palanca principal. Por el efecto del $\sqrt{n}$, cuadruplicar la muestra reduce el ancho a la mitad — cada mejora sucesiva de precisión cuesta cuatro veces más que la anterior.
- **Variabilidad de los datos ($s$):** a mayor dispersión de las observaciones, más ancho el intervalo. No se controla la variabilidad natural del fenómeno, pero sí la variabilidad añadida por el proceso de medición.
- **Nivel de confianza:** subirlo ensancha el intervalo; es la única palanca que se maneja bajando la exigencia en vez de mejorar el estudio, por lo que debe fijarse antes de ver los datos.

### Determinar el Tamaño de Muestra por Adelantado

Si se conoce el margen de error máximo tolerable $E$ antes de recolectar datos, se despeja $n$ de la fórmula del margen:

$$
n \ge \left(\frac{z \cdot \sigma}{E}\right)^2
$$

El resultado siempre se redondea hacia arriba, porque un tamaño de muestra fraccionario no existe y redondear hacia abajo incumpliría el requisito.

**Ejemplo:** estimar una media con margen de error no mayor a 3 unidades, 95% de confianza, y $\sigma \approx 12$ de estudios previos.

```r
z <- qnorm(0.975)       # 1.9600
n <- (z * 12 / 3)^2     # 61.4633
ceiling(n)              # 62
```

Se requieren 62 observaciones. Esta fórmula usa $z$ y $\sigma$, no $t$ y $s$: antes de recolectar los datos no se dispone de $s$, así que se trabaja con la mejor estimación disponible de $\sigma$.

---

## Interpretación Correcta de un Intervalo de Confianza

Ante un intervalo como (508.40, 526.80), la afirmación «hay un 95% de probabilidad de que $\mu$ esté en ese rango» es **incorrecta**: $\mu$ es una constante (desconocida, pero fija), y los límites del intervalo ya están calculados — también son fijos. $\mu$ está dentro del intervalo o no lo está; no hay probabilidad involucrada en dos constantes.

Lo aleatorio no es $\mu$, es el intervalo: cada muestra produce un intervalo distinto, y el 95% de esos intervalos posibles contiene a $\mu$. Formulaciones correctas:

- «Con un nivel de confianza del 95%, el valor promedio se encuentra entre estos límites.»
- «El intervalo fue construido con un método que captura el verdadero valor en el 95% de los casos.»

Otros errores frecuentes: interpretar el intervalo como si describiera el 95% de las observaciones individuales (el intervalo es para la media, no para los datos individuales, que son mucho más dispersos); o asumir que un intervalo angosto implica una muestra representativa (el ancho depende de $n$ y $s$, no de si el muestreo fue correcto — un muestreo sesgado produce intervalos angostos alrededor del valor equivocado).

