---
Fecha de creación: 2026-04-29T18:00:00
Materia:
  - Matemática III
Fecha de clase: 2026-04-29
---
[[Clase 13 - Integrales Dobles en Coordenadas Polares|← Clase anterior]]

# Aplicaciones de Integrales Múltiples
(cierre del curso: aplica las [[Clase 12 - Integrales Dobles|Integrales Dobles]] y las [[Clase 13 - Integrales Dobles en Coordenadas Polares#Introducción a Integrales Triples|Integrales Triples]] de las Clases 12 y 13 a problemas físicos concretos)

Más allá de calcular áreas y volúmenes, las integrales múltiples permiten calcular propiedades físicas de una región o un sólido cuando la densidad no es uniforme.

## Volumen de un Sólido

Ya visto como caso particular: el volumen entre la superficie $z=f(x,y)$ y la región $D$ del plano $xy$ es

$$
V = \iint_D f(x,y)\, dA
$$

y el volumen de un sólido $E$ definido directamente en el espacio es $V=\iiint_E dV$.

## Masa de una Lámina o de un Sólido

Si una lámina plana ocupa la región $D$ y tiene una **densidad superficial** variable $\rho(x,y)$ (masa por unidad de área, en vez de una densidad constante), su masa total es:

$$
m = \iint_D \rho(x,y)\, dA
$$

Para un sólido $E$ con densidad volumétrica $\rho(x,y,z)$:

$$
m = \iiint_E \rho(x,y,z)\, dV
$$

## Momentos y Centro de Masa

El **centro de masa** $(\bar{x},\bar{y})$ de una lámina es el punto donde se podría concentrar toda su masa sin alterar su comportamiento respecto a la rotación. Se calcula a partir de los **momentos** respecto a cada eje:

$$
M_y = \iint_D x\,\rho(x,y)\, dA, \qquad M_x = \iint_D y\,\rho(x,y)\, dA
$$

$$
\bar{x} = \frac{M_y}{m}, \qquad \bar{y} = \frac{M_x}{m}
$$

Si la densidad es constante, el centro de masa coincide con el **centroide** — el "centro geométrico" de la región, que depende solo de su forma.

### Ejemplo

Encontrar el centro de masa de la lámina triangular con vértices $(0,0)$, $(1,0)$, $(0,1)$ y densidad constante $\rho=1$ (equivalente al centroide):

La región es $D = \{(x,y) : 0 \leq x \leq 1,\ 0 \leq y \leq 1-x\}$, con masa (área) $m = \frac{1}{2}$.

$$
M_y = \int_0^1\int_0^{1-x} x\, dy\, dx = \int_0^1 x(1-x)\, dx = \frac{1}{6}
$$

Por simetría del triángulo, $M_x = \frac{1}{6}$ también.

$$
\bar{x} = \bar{y} = \frac{1/6}{1/2} = \frac{1}{3}
$$

El centroide es $\left(\frac{1}{3},\frac{1}{3}\right)$, consistente con la propiedad geométrica conocida de que el centroide de un triángulo está a un tercio de cada lado.

## Momento de Inercia

El **momento de inercia** mide la resistencia de una masa a cambiar su velocidad de rotación alrededor de un eje — el análogo rotacional de la masa en el movimiento lineal. Respecto al eje $x$ y al eje $y$:

$$
I_x = \iint_D y^2\,\rho(x,y)\, dA, \qquad I_y = \iint_D x^2\,\rho(x,y)\, dA
$$

Nótese la diferencia con los momentos $M_x$, $M_y$ usados para el centro de masa: aquí la distancia al eje se eleva al cuadrado, porque la inercia rotacional depende del cuadrado de la distancia al eje de giro.

## Valor Promedio de una Función

Generalizando el valor promedio de una función de una variable, el valor promedio de $f(x,y)$ sobre una región $D$ es:

$$
f_{prom} = \frac{1}{A(D)}\iint_D f(x,y)\, dA
$$

es decir, la integral dividida entre el área de la región — el mismo principio usado para el promedio de una función en cálculo de una variable.

## Resumen del Curso

Este curso avanzó de lo geométrico a lo analítico y de vuelta a lo aplicado:

1. **Vectores y geometría del espacio** ([[Clase 01 - Sistemas Coordenados Cartesianos|Clase 01]]–[[Clase 04 - Superficies Cuádricas|04]]): cómo describir puntos, direcciones y superficies en $\mathbb{R}^3$.
2. **Funciones de varias variables y sus derivadas** ([[Clase 07 - Funciones de Varias Variables|Clase 07]]–[[Clase 10 - Multiplicadores de Lagrange|10]]): cómo cambian esas superficies, y cómo optimizarlas.
3. **Integrales múltiples** ([[Clase 12 - Integrales Dobles|Clase 12]]–14): cómo acumular esas funciones sobre una región para obtener volumen, masa, centros de masa e inercia.
