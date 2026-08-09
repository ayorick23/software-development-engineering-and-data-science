---
Fecha de creación: 2026-03-04T18:04:00
Materia:
  - Matemática III
Fecha de clase: 2026-03-04
---
[[Clase 05 - Repaso para el Parcial|← Clase anterior]] | [[Clase 08 - Regla de la Cadena, Derivación Implícita y Gradiente|Clase siguiente →]]

# Unidad II: Funciones de Varias Variables

Una **función de dos variables** es una regla que asigna un valor real a cada par ordenado:

$$
f:D⊆\mathbb{R}^2\to\mathbb{R}
$$
$$
z = f(x,y)
$$

- **Dominio**: conjunto de pares $(x,y)$ válidos.
- **Rango**: valores posibles de $z$.

Geométricamente representa una superficie en $\mathbb{R}^3$

**Ejemplo:**

$$
f(x,y) = x^2 + 2y^2
$$

Dominio:

$$
Dom(f) = \mathbb{R}^2
$$

Superficie generada:

- Paraboloide elíptico
- Mínimo en $(0,0)$

## Restricciones del Dominio

Para que una función produzca valores reales se deben cumplir ciertas reglas.

### Denominadores

Si

$$
f(x,y)=\frac{Q(x,y)}{P(x,y)}​
$$

entonces

$$
Q(x,y)\neq0
$$

### Radicales pares

Si

$$
f(x,y)=\sqrt{P(x,y)}
$$

entonces

$$
P(x,y) \ge 0
$$

### Logaritmos

Si

$$
f(x,y)=\ln(P(x,y))
$$

entonces

$$
P(x,y) > 0
$$

## Límites de Funciones de Varias Variables

En funciones de varias variables se puede aproximar un punto **desde infinitas direcciones**.

Se define:

$$
\lim_{(x,y)\to(a,b)}​f(x,y)=L
$$

si

$$
\forall \epsilon > 0,\ \exists\, \delta > 0
$$

tal que

$$
|f(x,y) - L| < \epsilon
$$

cuando

$$
0 < \sqrt{(x-a)^2+(y-b)^2} < \delta
$$

### Interpretación

- $\epsilon$ → tolerancia en el valor de la función
- $\delta$ → radio del entorno alrededor del punto

A diferencia de una función de una variable (donde solo hay dos formas de acercarse a un punto: por la izquierda o por la derecha), aquí el punto $(x,y)$ puede aproximarse a $(a,b)$ desde **infinitas direcciones y trayectorias** dentro de ese círculo de radio $\delta$. Para que el límite exista, $f(x,y)$ debe acercarse a $L$ sin importar por cuál de esas infinitas trayectorias nos aproximemos — si dos trayectorias distintas dan resultados diferentes, el límite no existe.