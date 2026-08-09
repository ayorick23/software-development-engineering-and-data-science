---
Fecha de creación: 2026-04-15T18:00:00
Materia:
  - Matemática III
Fecha de clase: 2026-04-15
---
[[Clase 10 - Multiplicadores de Lagrange|← Clase anterior]] | [[Clase 13 - Integrales Dobles en Coordenadas Polares|Clase siguiente →]]

# Integrales Dobles
(cierra la [[Clase 10 - Multiplicadores de Lagrange|Unidad de optimización]] y abre la unidad de integración múltiple, la contraparte de las derivadas parciales estudiadas desde la [[Clase 07 - Funciones de Varias Variables|Clase 07]])

Así como la integral definida de una variable calcula el área bajo una curva, la **integral doble** de una función de dos variables $f(x,y)$ calcula el **volumen** bajo la superficie $z=f(x,y)$ y sobre una región $D$ del plano $xy$.

## Definición sobre un Rectángulo

Para una región rectangular $R = [a,b]\times[c,d]$, la integral doble se define, igual que en una variable, como el límite de una suma de Riemann de volúmenes de prismas rectangulares:

$$
\iint_R f(x,y)\, dA = \lim_{m,n\to\infty} \sum_{i=1}^m \sum_{j=1}^n f(x_i,y_j)\,\Delta A
$$

## Integrales Iteradas y el Teorema de Fubini

En la práctica, una integral doble se calcula como dos integrales de una variable aplicadas sucesivamente, una integral **iterada**:

$$
\iint_R f(x,y)\, dA = \int_a^b \int_c^d f(x,y)\, dy\, dx = \int_c^d \int_a^b f(x,y)\, dx\, dy
$$

El **Teorema de Fubini** garantiza que, si $f$ es continua en $R$, ambos órdenes de integración dan el mismo resultado — se puede integrar primero respecto a $x$ o primero respecto a $y$, lo que convenga más.

### Ejemplo

Calcular $\displaystyle\iint_R (x^2+y)\, dA$ sobre $R=[0,2]\times[0,1]$:

$$
\int_0^2 \int_0^1 (x^2+y)\, dy\, dx = \int_0^2 \left[x^2y+\frac{y^2}{2}\right]_0^1 dx = \int_0^2 \left(x^2+\frac{1}{2}\right)dx = \left[\frac{x^3}{3}+\frac{x}{2}\right]_0^2 = \frac{8}{3}+1 = \frac{11}{3}
$$

## Integrales Dobles sobre Regiones Generales

Cuando la región $D$ no es un rectángulo, se describe mediante límites de integración que dependen de la otra variable:

**Tipo I** (región entre dos curvas $y=g_1(x)$ y $y=g_2(x)$, para $x \in [a,b]$):

$$
\iint_D f(x,y)\, dA = \int_a^b \int_{g_1(x)}^{g_2(x)} f(x,y)\, dy\, dx
$$

**Tipo II** (región entre dos curvas $x=h_1(y)$ y $x=h_2(y)$, para $y \in [c,d]$):

$$
\iint_D f(x,y)\, dA = \int_c^d \int_{h_1(y)}^{h_2(y)} f(x,y)\, dx\, dy
$$

Elegir el tipo correcto (o cambiar el orden de integración) es a menudo la parte más importante del problema — de la misma forma en que, en la [[Clase 09 - Derivadas Direccionales, Gradientes y Fórmula de Taylor|Clase 09]], elegir la dirección correcta era clave para la derivada direccional.

## Propiedades

- **Linealidad:** $\iint_D [f(x,y)+g(x,y)]\,dA = \iint_D f\,dA + \iint_D g\,dA$
- **Aditividad sobre regiones:** si $D = D_1 \cup D_2$ sin traslape, $\iint_D f\,dA = \iint_{D_1} f\,dA + \iint_{D_2} f\,dA$
- Si $f(x,y) \geq 0$ en $D$, la integral representa el volumen exacto bajo la superficie.

## Cálculo del Área de una Región

Un caso particular útil: si $f(x,y)=1$, la integral doble da directamente el área de la región $D$:

$$
A(D) = \iint_D 1\, dA
$$
