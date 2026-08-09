---
Fecha de creación: 2026-02-04T18:00:00
Materia:
  - Matemática III
Fecha de clase: 2026-02-04
---
[[Clase 02 - Ángulos y Cosenos Directores de un Vector|← Clase anterior]] | [[Clase 04 - Superficies Cuádricas|Clase siguiente →]]

# Producto Cruz, Rectas y Planos en el Espacio
(usa los [[Clase 02 - Ángulos y Cosenos Directores de un Vector|Cosenos Directores]] de la Clase 02 y la [[Clase 01 - Sistemas Coordenados Cartesianos#Norma o Magnitud|Norma]] de la Clase 01 para construir las ecuaciones de rectas y planos)

## Producto Cruz (Producto Vectorial)

A diferencia del [[Clase 01 - Sistemas Coordenados Cartesianos#Adición y Sustracción de Vectores|producto punto]], que devuelve un escalar, el **producto cruz** de dos vectores $u = (u_1,u_2,u_3)$ y $v = (v_1,v_2,v_3)$ en $\mathbb{R}^3$ devuelve un nuevo **vector**, perpendicular a ambos:

$$
u \times v = \begin{vmatrix} \mathbf{i} & \mathbf{j} & \mathbf{k} \\ u_1 & u_2 & u_3 \\ v_1 & v_2 & v_3 \end{vmatrix} = (u_2v_3-u_3v_2,\ u_3v_1-u_1v_3,\ u_1v_2-u_2v_1)
$$

### Propiedades

- $u \times v$ es perpendicular tanto a $u$ como a $v$.
- $|u \times v| = |u||v|\sin\theta$ — coincide numéricamente con el área del paralelogramo formado por $u$ y $v$.
- $u \times v = -(v \times u)$ (anticonmutativo, a diferencia del producto punto).
- $u \times v = \vec{0}$ si y solo si $u$ y $v$ son paralelos.

## Ecuación de una Recta en el Espacio

Una recta en $\mathbb{R}^3$ queda determinada por un punto $P_0(x_0,y_0,z_0)$ y un vector director $v = (a,b,c)$ paralelo a la recta.

### Ecuación Vectorial

$$
(x,y,z) = (x_0,y_0,z_0) + t(a,b,c), \quad t \in \mathbb{R}
$$

### Ecuaciones Paramétricas

$$
x = x_0 + ta, \qquad y = y_0 + tb, \qquad z = z_0 + tc
$$

### Ecuaciones Simétricas

Si $a,b,c \neq 0$, despejando $t$ de cada ecuación paramétrica e igualando:

$$
\frac{x-x_0}{a} = \frac{y-y_0}{b} = \frac{z-z_0}{c}
$$

## Ecuación de un Plano en el Espacio

Un plano queda determinado por un punto $P_0(x_0,y_0,z_0)$ y un **vector normal** $n = (a,b,c)$ perpendicular al plano.

$$
a(x-x_0) + b(y-y_0) + c(z-z_0) = 0
$$

que se simplifica a la forma general:

$$
ax+by+cz = d
$$

### Obtención del Vector Normal con el Producto Cruz

Si en lugar del vector normal se conocen dos vectores directores $u$ y $v$ contenidos en el plano (por ejemplo, formados a partir de tres puntos no colineales del plano), el vector normal se obtiene con:

$$
n = u \times v
$$

Esto conecta directamente ambos conceptos de la clase: el producto cruz es, en la práctica, la herramienta para encontrar el vector normal que define un plano.

### Ejemplo

Encontrar el plano que pasa por los puntos $A(1,0,0)$, $B(0,1,0)$, $C(0,0,1)$:

$$
u = B - A = (-1,1,0), \qquad v = C - A = (-1,0,1)
$$

$$
n = u \times v = (1,1,1)
$$

Usando el punto $A(1,0,0)$:

$$
1(x-1) + 1(y-0) + 1(z-0) = 0 \implies x+y+z = 1
$$

## Distancia de un Punto a un Plano

$$
d = \frac{|ax_1+by_1+cz_1-d|}{\sqrt{a^2+b^2+c^2}}
$$

donde $(x_1,y_1,z_1)$ es el punto y $ax+by+cz=d$ es el plano.
