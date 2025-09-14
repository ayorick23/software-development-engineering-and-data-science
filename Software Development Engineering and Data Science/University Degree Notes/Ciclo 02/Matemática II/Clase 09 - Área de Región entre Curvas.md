---
Fecha de creación: 2025-09-10T20:18:00
Materia:
  - Matemática II
Fecha de clase: 2025-09-10
---
# Área de Región entre Curvas

Sean dos funciones continuas $f(x)$ y $g(x)$ en el intervalo cerrado $[a, b]$, donde: $f(x) >= g(x)$ en $[a,b]$, es decir, la gráfica de $f$ está por encima de la de $g$.

Entonces, el área entre las dos curvas se obtiene restando el área bajo $g(x)$ al área bajo $f(x)$.

$$
Área = \int_a^b [f(x) - g(x)] dx
$$

## Interpretación Gráfica

Imaginemos la región entre las curvas f(x) y g(x) como una serie de rectángulos verticales. La altura de cada rectángulo es $f(x_i) - g(x_i)$ y su anchura es $\Delta x$.

![[Drawing 2025-09-13 12.02.03.excalidraw]]

Sumando las áreas de todos esos rectángulos y haciendo el límite cuando $\Delta x \to 0$, obtenemos la integral definida:

$$
\lim_{n \to \infty} \sum_{i=1}^n [f(x_i) - g(x_i)]\Delta x = \int_a^b [f(x) - g(x)] dx
$$

### Condiciones para Aplicar la Fórmula

Para que la fórmula funcione correctamente, se deben cumplir los siguientes puntos:

- $f(x)$ y $g(x)$ deben ser continuas en $[a, b]$.
- En todo el intervalo, $f(x) >= g(x)$, o se debe dividir el intervalo donde cambia el orden.

## Cálculo de Áreas con Rectángulos Horizontales

Al calcular el área entre dos curvas, generalmente se emplea la integración respecto a la variable $x$. Sin embargo, en algunas situaciones es más conveniente integrar respecto a la variable $y$, especialmente cuando las funciones están definidas como $x=f(y)$ y $x=g(y)$, o cuando la región está limitada lateralmente por funciones de $y$.

Si la región está delimitada por funciones de $y$, o si las gráficas están dadas como $x=f(y)$ y $x=g(y)$, puede ser más sencillo trabajar con **rectángulos horizontales**.

En este caso, los límites de integración serán de $y$, desde $y=c$ hasta $y=d$, y la base del rectángulo será:

$$
Base = f(y) - g(y)
$$

La fórmula para el área es:

$$
Área = \int_c^d [f(y) - g(y)]dy
$$

### Determinación de los Límites de Integración

Ya sea que se integren rectángulo verticales u horizontales, los **límites de integración** se obtienen a partir de los **puntos de intersección** entre las curvas involucradas o de los valores extremos proporcionados en el enunciado del problema.

- Si se integrará respecto a $x$, los límites son $x_1$ y $x_2$.
- Si se integrará respecto a $y$, los límites son $y_1$ y $y_2$.

#### Elección del Método

Usar **rectángulos verticales** cuando las funciones expresadas como $y=f(x)$, y el área está delimitada por valores de $x$.

Usar **rectángulos horizontales** cuando las funciones están expresadas como $x=f(y)$, o cuando la región está más naturalmente delimitada en la dirección vertical (respecto a $y$).

## La Integración como un Proceso de Acumulación

La integración se puede entender como un **proceso de acumulación** de cantidades pequeñas. Este enfoque es fundamental en cálculo y nos permite resolver una amplia variedad de problemas en los que se desea conocer una cantidad total que resulta de sumar muchas partes pequeñas.

Se abordará cómo usar la integración definida para encontrar el área entre dos curvas, y para ello se parte de una construcción visual e intuitiva: la de un rectángulo como elemento representativo.

$$
A = (altura)(ancho) \to \Delta A = [f(x) - g(x)]\Delta x \to A = \int_a^b [f(x) - g(x)]dx
$$

En esta idea, cada aplicación de la integral se basa en la suma acumulativa de elementos representativos adecuados:

En el caso de áreas: **rectángulos verticales u horizontales.**

En el caso de volúmenes: **discos, arandelas o capas cilíndricas**.

En otros casos, como trabajo realizado por una fuerza o acumulación de masa: se usarán elementos como **fuerza X distancia**, **densidad X longitud**, etc.

Cada fórmula de integración sigue el mismo patrón:

**Fórmula total = Suma acumulada de elementos representativos apropiados**

Este enfoque no solo ayuda a comprender mejor el significado de la integral, sino que permite desarrollar nuevas fórmulas para resolver problemas reales.

## Cálculo de Volúmenes

Siguiendo con la idea de la integral como un proceso de acumulación se estudiará un tipo particular, el del volumen de un sólido tridimensional cuyas secciones transversales son similares. Por lo común se emplean sólido de revolución en ingeniería y manufactura. Algunos ejemplos son ejes, embudos, píldoras, botellas y pistones, o piezas mecánicas.

