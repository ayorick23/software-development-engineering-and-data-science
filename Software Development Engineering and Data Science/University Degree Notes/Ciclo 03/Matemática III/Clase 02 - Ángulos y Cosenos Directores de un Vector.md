---
Fecha de creación: 2026-01-28T18:20:00
Materia:
  - Matemática III
Fecha de clase: 2026-01-28
---
[[Clase 01 - Sistemas Coordenados Cartesianos|← Clase anterior]] | [[Clase 03 - Producto Cruz, Rectas y Planos en el Espacio|Clase siguiente →]]

# Ángulos y Cosenos Directores de un Vector
(extiende a $\mathbb{R}^3$ la [[Clase 01 - Sistemas Coordenados Cartesianos#Dirección|Dirección de un Vector]] vista en la Clase 01, donde un solo ángulo $\theta$ bastaba en $\mathbb{R}^2$)

Mientras que la magnitud de un vector v nos indica su "longitud" o "intensidad", su dirección especifica su orientación en el espacio. Una forma precisa de describir esta dirección en $\mathbb{R}^3$ es mediante los ángulos directores y los cosenos directores. Estos nos indican cómo se alinea el vector con respecto a los ejes coordenados positivos.

## Ángulos Directores

Dado un vector $v = (x, y, z)$ no nulo, sus **ángulos directores** $\alpha$, $\beta$ y $\gamma$ son los ángulos que $v$ forma, respectivamente, con los semiejes positivos $x$, $y$ y $z$. Por convención, cada ángulo director está entre $0$ y $\pi$ radianes ($0°$ a $180°$).

## Cosenos Directores

Los **cosenos directores** son, simplemente, el coseno de cada uno de esos tres ángulos:

$$
\cos\alpha = \frac{x}{|v|}, \qquad \cos\beta = \frac{y}{|v|}, \qquad \cos\gamma = \frac{z}{|v|}
$$

donde $|v| = \sqrt{x^2+y^2+z^2}$ es la [[Clase 01 - Sistemas Coordenados Cartesianos#Norma o Magnitud|norma]] del vector (extendida a tres dimensiones).

En otras palabras: cada coseno director es la proyección relativa del vector sobre ese eje.

## Propiedad Fundamental

Los cosenos directores de cualquier vector no nulo satisfacen siempre:

$$
\cos^2\alpha + \cos^2\beta + \cos^2\gamma = 1
$$

Esto significa que el vector $(\cos\alpha, \cos\beta, \cos\gamma)$ es precisamente el **vector unitario** en la misma dirección que $v$ — es decir, $\hat{v} = \dfrac{v}{|v|}$.

## Ejemplo

Para $v = (1, 2, 2)$:

$$
|v| = \sqrt{1^2+2^2+2^2} = \sqrt{9} = 3
$$

$$
\cos\alpha = \frac{1}{3}, \qquad \cos\beta = \frac{2}{3}, \qquad \cos\gamma = \frac{2}{3}
$$

Verificación: $\left(\frac{1}{3}\right)^2+\left(\frac{2}{3}\right)^2+\left(\frac{2}{3}\right)^2 = \frac{1+4+4}{9} = 1$ ✓