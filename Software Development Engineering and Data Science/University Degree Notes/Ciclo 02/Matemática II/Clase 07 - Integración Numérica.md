---
Fecha de creación: 2025-08-29T20:14:00
Materia:
  - Matemática II
Fecha de clase: 2025-08-29
---

# La Integración Numérica

## Notación Sigma ($\Sigma$)

La **notación sigma** es una forma concisa de expresar la suma de una serie de términos. Utiliza la letra griega mayúscula sigma ($\Sigma$) para representar la suma de una secuencia de valores según una regla específica. La estructura general de la notación sigma es:

$$
\sum_{i=m}^n a_i
$$

donde:

- $i$ es el índice de suma.
- $m$ es el valor inicial del índice.
- $n$ es el valor final del índice.
- $a_i$ es la expresión que se suma para cada valor de $i$.

**Ejemplo:** Sumar los primeros cinco número enteros positivos.

$$
\sum_{i=1}^5=1+2+3+4+5=15
$$

## Sumas de Riemann

1.  $$
    \sum_{i=1}^n k = kn
    $$
2.  $$
    \sum_{i=1}^n i = \frac{n(n+1)}{2} = \frac{n^2+n}{2}
    $$
3.  $$
    \sum_{i=1}^n i^2 = \frac{n(n+1)(2n+1)}{6} = \frac{2n^3+3n^2+n}{6}
    $$
4.  $$
    \sum_{i=1}^n i^3 = \frac{n^2(n+1)^2}{4} = \frac{n^4+2n^3+n^2}{4}
    $$

## Aproximación del Área de una Región Plana (Integración Numérica)

Para determinar el área de una región plana delimitada por la gráfica de una función y el eje $x$, una estrategia común es aproximar el área mediante sumas de rectángulos. El proceso implica dividir el intervalo en subintervalos de igual tamaño y construir rectángulos cuya altura esté definida por el valor de la función en un punto específico del subintervalo (como el extremo izquierdo o derecho).

(un excalidraw)

Para plantear matemáticamente el área total bajo la curva aproximada mediante la suma de las áreas de los rectángulos consideremos una región plana limitada en su parte superior (gráfica derecha) por la gráfica de una función continua y no negativa $f(x)$. La región está limitada en su parte inferior por el eje $x$ y en sus lados por las líneas verticales $x=a$ y $x=b$.

Para aproximar el área de esta región, subdividimos el intervalo $[a,b]$ en $n$ subintervalos de igual longitud:

$$
\Delta x=\frac{b-a}{n}
$$

(otro excalidraw)

Como $f(x)$ es continua, el Teorema del Valor Extremo garantiza la existencia de un mínimo $f(m_i)$ y un máximo $f(M_i)$ en cada subintervalo. Definiremos un rectángulo inscrito a aquel que se encuentra dentro de la i-ésima subregión y un rectángulo circunscrito al que se extiende fuea de la i-ésima región.

Dado que el área de un rectángulo es producto de su base por su altura, definimos:

**Suma Inferior:** Se calculo sumando las áreas de los rectángulos inscritos dentro de la curva, utilizando los valores mínimos de $f(x)$ en cada subintervalo.

$$
s(n)=\sum_{i=1}^n f(m_i)\Delta x
$$

**Suma Superior:** Se calcula sumando las áreas de los rectángulos circunscritos que están encima de la curva, utilizando los valores máximos de $f(x)$ en cada subintervalo.

$$
S(n)=\sum_{i=1}^n f(M_i)\Delta x
$$

La relación entre estas sumas y el área real de la región es: $s(n) <= \text{Área real} <= S(n)$

Lo representado en la desigualdad anterior se puede ver de manera gráfica en la imagen inferior, donde el área aproximada por rectángulos inscritos es menor al área real, contrario a los rectángulos cicunscritos que exceden el área real.

(otro excalidraw)

Este proceso de aproximación es lo que conocemos como aproximación de la integral.

## Hallar las Sumas Superior e Inferior de una Región

Determinemos la suma superior e inferior de la región delimitada por la gráfica de $f(x)=x^2$ y el eje $x$, en el intervalo $[0,2]$.

**Solución:**

Dividimos el intervalo $[0,2]$ en $n$ subintervalos, cada uno de ancho:

$$
\Delta x=\frac{2-0}{n}=\frac{2}{n}
$$

Dado que $f(x)=x^2$ es una función creciente en $[0,2]$, el valor mímino en cada subintervalo ocurre en el extremo izquierdo y el valor máximo en el extremo derecho:

Puntos terminales derechos:

$$
M_i = 0+\frac{2i}{n}=\frac{2i}{n}
$$

$$
f(M_i) = (\frac{2i}{n})^2=\frac{4i^2}{n^2}
$$

Puntos terminales izquierdos:

$$
m_i = 0+(i-1)\frac{2}{n}=\frac{2(i-1)}{n}
$$

$$
f(m_i) = (\frac{2(i-1)}{n})^2=\frac{4(i-1)^2}{n^2}
$$

Suma inferior utilizando los puntos terminales izquierdos:

(excalidraw)

$$
S(n)=\sum_{i=1}^n f(M_i)\Delta x = \sum_{i=1}^n(\frac{4i^2}{n^2})\frac{2}{n}
$$

Aplicando la fórmula de la sumatoria de cuadrados:

$$
\sum_{i=1}^n i^2=\frac{n(n+1)(2n+1)}{6}
$$

Sustituyendo:

$$
S(n)=\frac{8}{n^3}.\frac{n(n+1)(2n+1)}{6}
$$

Obtenemos:

$$
S(n)=\frac{8}{3}+\frac{4}{n}+\frac{4}{3n^2}
$$

Cuando $n\to\infty$ ambas sumas tienden a $\frac{8}{3}$, lo que indica que el área de la región es:

$$
\lim_{n\to\infty}S(n)=\lim_{n\to\infty}\frac{8}{3}+\frac{4}{n}+\frac{4}{3n^2}=\frac{8}{3}
$$

$$
\lim_{n\to\infty}s(n)=\lim_{n\to\infty}\frac{8}{3}+\frac{4}{n}+\frac{4}{3n^2}=\frac{8}{3}
$$

## Teorema sobre los Límites de las Sumas Superior e Inferior

El siguiente teorema establece que la convergencia de los límites de las sumas superior e inferior, cuando $n\to\infty$, no es una simple coincidencia. Este resultado se cumple para cualquier función continua y no negativa en el intervalo cerrado $[a,b]$. Su demostración es más apropiada para un curso avanzado de cálculo.

Si $f$ es una función continua y no negativa en el intervalo $[a,b]$ entonces los límites de las sumas inferior y superior, al aumentar la cantidad de subintervalos, existen y son iguales:

$$
\lim_{n\to\infty} S(n)=\lim_{n\to\infty}\sum_{i=1}^n f(M_i)\Delta x
$$

$$
\lim_{n\to\infty} s(n)=\lim_{n\to\infty}\sum_{i=1}^n f(m_i)\Delta x
$$

donde $\Delta x=\frac{b-a}{n}$ y $f(m_i)$ y $f(M_i)$ representan los valores mínimo y máximo de $f$ en el subintervalo i-ésimo.

## Definición del Área de una Región en el Plano

Si $f$ es una función continua y no negativa en $[a,b]$, en el área de la región delimitada por la gráfica de $f$, el eje $x$ y las líneas verticales en $x=a$ y $x=b$ se define como:

$$
Área=\lim_{n\to\infty}\sum_{i=1}^n f(c_i)\Delta x
$$

donde

$$
\Delta x=\frac{b-a}{n}
$$

es un punto dentro del i-ésimo subintervalo.
