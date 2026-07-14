---
Fecha de creación: 2026-03-25T18:11:00
Materia:
  - Matemática III
Fecha de clase: 2026-03-25
---
# Multiplicadores de Lagrange

El método de **Multiplicadores de Lagrange** es un procedimiento estratégico de cálculo multivariable utilizado para encontrar los máximos y mínimos locales de una función $f(x, y, . . .)$ sometida a la condición impuesta por una restricción de igualdad $g(x, y, . . .) = k$. Introduce una nueva variable escalar $\lambda$ para transformar un problema con restricciones en un sistema de ecuaciones resoluble.

## Condición de Lagrange

Suponga que $f(x, y)$ y $g(x, y)$ tienen derivadas parciales continuas, y que $\nabla g \neq \vec{0}$ sobre la curva de restricción $g(x, y) = k$. Si $f(x, y)$ tiene un extremo (máximo o mínimo) en un punto ($x0, y0$) sobre la restricción, entonces en ese punto el gradiente de $f$ es paralelo al gradiente de $g$. Es decir, existe un número real $\lambda$ tal que:

$$
\nabla f(x_0,y_0) = \lambda\nabla g(x_0,y_0)
$$

## Hessiano Orlado

El **Hessiano Orlado (Bordered Hessian)**, denotado comúnmente como $H$ , es una expansión de la Matriz Hessiana tradicional. Se construye «orlando» o rodeando las segundas derivadas de la función Lagrangiana $L(x, y, \lambda) = f(x, y) − \lambda(g(x, y) − k)$ con las primeras derivadas de la función de restricción $g(x, y)$. Su determinante nos permite clasificar el punto crítico restringido.

### Criterio del Determinante del Hessiano Orlado

Sea la función Lagrangiana $L(x, y, λ) = f(x, y) − λ(g(x, y) − k)$. Evaluada en el punto crítico $(x_0, y_0, \lambda_0)$, la Matriz Hessiana Orlada para dos variables es:

$$
H = \begin{bmatrix}0&g_x&g_y\\g_x&L_{xx}&L_{xy}\\g_y&L_{yx}&L_{yy}\end{bmatrix}
$$

El determinante de esta matriz, evaluado en el punto crítico, dictamina la naturaleza del punto:

- Si $|H| > 0$ (Positivo), la función tiene un **Máximo Relativo** restringido.
- Si $|H| < 0$ (Negativo), la función tiene un **Mínimo Relativo** restringido.
- Si $|H| = 0$, el criterio no es concluyente (falla).

## Selección de Características mediante Regularización

La **Selección de Características (_Feature Selection_)** es el proceso de reducir la dimensionalidad del modelo predictivo. Matemáticamente, se modela como un problema de optimización donde minimizamos el **Error Cuadrático Medio (MSE)**, sujeto a una restricción Lagrangiana en la magnitud de los pesos (Ej. $\sum|w_i| \leq k$). Esto obliga al algoritmo a "apagar" (hacer cero) los pesos de las características menos importantes.

## Glosario de Saberes Previos

### Función Objetivo

La función objetivo, denotada como $f(x, y)$, es la ecuación matemática que deseamos maximizar (ej. rendimiento, FPS de un juego, ganancias) o minimizar (ej. uso de CPU, tasa de error, pérdida).

### Ecuación de Restricción

Una restricción es una condición inflexible que las variables del sistema deben cumplir. Se modela igualando a una constante: $g(x, y) = k$. Geométricamente, representa un cerco, una frontera o un carril del cual el algoritmo no puede escapar.

### Curvas de Nivel