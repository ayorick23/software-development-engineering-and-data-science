---
Fecha de creación: 2026-04-22T18:00:00
Materia:
  - Matemática III
Fecha de clase: 2026-04-22
---
[[Clase 12 - Integrales Dobles|← Clase anterior]] | [[Clase 14 - Aplicaciones de Integrales Múltiples|Clase siguiente →]]

# Integrales Dobles en Coordenadas Polares
(convierte la [[Clase 12 - Integrales Dobles|Integral Doble]] de la Clase 12 a coordenadas polares cuando la región $D$ es circular o angular, un caso donde las coordenadas cartesianas complican innecesariamente los límites de integración)

## ¿Por Qué Cambiar a Coordenadas Polares?

Regiones como discos, sectores circulares o anillos se describen de forma mucho más simple con radio $r$ y ángulo $\theta$ que con $x$ e $y$. Por ejemplo, el disco $x^2+y^2 \leq 4$ requiere límites con raíces cuadradas en cartesianas, pero en polares es simplemente $0 \leq r \leq 2$, $0 \leq \theta \leq 2\pi$.

## Conversión de Coordenadas

$$
x = r\cos\theta, \qquad y = r\sin\theta, \qquad x^2+y^2 = r^2
$$

## El Jacobiano: por qué $dA \neq dr\,d\theta$

Al cambiar de variable en una integral doble, el elemento de área no se transforma de manera directa — hay que multiplicar por el **Jacobiano** de la transformación, que en coordenadas polares es $r$:

$$
dA = dx\,dy = r\, dr\, d\theta
$$

Geométricamente: un pequeño rectángulo polar de lados $dr$ y $d\theta$ no tiene área $dr\cdot d\theta$, sino aproximadamente $r\,dr\,d\theta$, porque un mismo incremento angular $d\theta$ cubre más distancia real mientras más lejos del origen esté ($r$ más grande). Olvidar este factor $r$ es el error más común al hacer este cambio de variable.

## Fórmula General

$$
\iint_D f(x,y)\, dA = \int_\alpha^\beta \int_{r_1(\theta)}^{r_2(\theta)} f(r\cos\theta, r\sin\theta)\, r\, dr\, d\theta
$$

## Ejemplo

Calcular $\displaystyle\iint_D e^{-(x^2+y^2)}\, dA$, donde $D$ es el disco $x^2+y^2 \leq 1$ (una integral prácticamente imposible de resolver en cartesianas, ya que $e^{-x^2}$ no tiene antiderivada elemental):

$$
\int_0^{2\pi}\int_0^1 e^{-r^2}\, r\, dr\, d\theta
$$

Con la sustitución $u=r^2$, $du=2r\,dr$, la integral interna es:

$$
\int_0^1 e^{-r^2} r\, dr = \left[-\frac{1}{2}e^{-r^2}\right]_0^1 = \frac{1}{2}\left(1-e^{-1}\right)
$$

Multiplicando por la integral angular ($\int_0^{2\pi} d\theta = 2\pi$):

$$
\iint_D e^{-(x^2+y^2)}\, dA = \pi\left(1-e^{-1}\right)
$$

Este ejemplo ilustra el motivo principal para cambiar a polares: no solo simplifica la región, sino que a menudo simplifica también al integrando.

## Introducción a Integrales Triples

De la misma manera que la integral doble extiende la integral simple a una dimensión más, la **integral triple** de $f(x,y,z)$ sobre una región sólida $E$ del espacio calcula magnitudes en $\mathbb{R}^3$ (masa, promedio, etc., en vez de volumen bajo una superficie):

$$
\iiint_E f(x,y,z)\, dV = \int\int\int_E f(x,y,z)\, dz\, dy\, dx
$$

Se evalúa igual que una integral doble: de adentro hacia afuera, con los límites de la variable más interna pudiendo depender de las variables externas aún no integradas. Cuando $f(x,y,z)=1$, la integral triple da directamente el **volumen** del sólido $E$ — la aplicación más inmediata, que se explora en la [[Clase 14 - Aplicaciones de Integrales Múltiples|Clase 14]].
