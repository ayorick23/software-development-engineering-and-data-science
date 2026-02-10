---
Fecha de creación: 2026-01-21T18:01:00
Materia:
  - Matemática III
Fecha de clase: 2026-01-21
---
# Sistemas Coordenados Cartesianos

## Sistema de Coordenadas

El producto cartesiano $A × B$ de dos conjuntos se define como el conjunto de todos los pares ordenados $(x, y)$ donde $x$ es elemento de $A$ y $y$ es elemento de $B$. Este concepto fundamental permite visualizar relaciones matemáticas como una malla de puntos en el plano.

El producto cartesiano más importante es $ℝ × ℝ$, denotado $ℝ^2$, donde $ℝ$ representa el conjunto de números reales. Cada par ordenado en $ℝ^2$ se asocia de forma única con un punto del plano mediante un sistema de coordenadas cartesianas rectangulares.

## Estructura del Sistema

Las rectas perpendiculares divididas en segmentos numerados se llaman **ejes** del sistema de coordenadas. Su punto de intersección, $0$, es el origen. Los ejes dividen al plano en cuatro cuadrantes, numerados I, II, III y IV.

Para asociar un par ordenado $(a, b)$ con un punto S: se traza una recta paralela al eje vertical por el punto $a$ del eje $x$, y una recta paralela al eje horizontal por el punto $b$ del eje $y$. La intersección de estas rectas determina el punto S con coordenadas $(a, b)$.

## Correspondencia Única

Existe una correspondencia biunívoca entre el conjunto de puntos del plano y los miembros de $ℝ^2$. Cada punto del plano corresponde a exactamente un par ordenado, y cada par ordenado corresponde a exactamente un punto.

## Igualdad de Pares

Dos pares ordenados $(a, b)$ y $(c, d)$ de $ℝ^2$ corresponden al mismo punto si y sólo si son iguales: $a = c$ y $b = d$. Esta propiedad fundamental garantiza la unicidad de la representación.

## Fórmula de Distancia

La distancia entre dos puntos $A(x_1, y_1)$ y $B(x_2, y_2)$ está dada por:

$$d(A, B) = √[(x_2 - x_1)^2 + (y_2 - y_1)^2]$$

Esta fórmula es consecuencia directa del teorema de Pitágoras.

## Puntos y Vectores: Representación Geométrica

### Concepto de Vector

Un par ordenado $(x, y)$ puede representar tanto un punto como un **desplazamiento** o **traslación** en el plano. Como vector, describe un movimiento: $x$ unidades paralelas al eje $x$, seguidas de $y$ unidades paralelas al eje $y$.

La representación geométrica de un vector es una flecha o segmento de recta dirigido. Cada vector tiene infinitas representaciones geométricas, una con punto inicial en cada punto del plano.

### Representación Ordinaria

La flecha asociada con $(x, y)$ que tiene su punto inicial en el origen se llama representación ordinaria del vector, y se dice que está en posición ordinaria. Esta representación tiene como punto final el punto $T$ asociado con $(x, y)$.

Si una flecha con punto inicial $S(a, b)$ representa al vector $(x, y)$, entonces su punto final es $T(c, d)$ donde $(c, d) = (a + x, b + y)$. Recíprocamente, si el punto final es $T(c, d)$, entonces $(x, y) = (c - a, d - b)$.

>**Notación:** Se usan las letra en negrita **v, u** y **t** para vectores, y letras cursivas como *u, v* y *t* para escalares (números reales). En manuscrito, se puede usar una flecha encima o subrayado.

## Adición y Sustracción de Vectores

### Adición

Si $v₁ = (x₁, y₁)$ y $v₂ = (x₂, y₂)$, entonces $v₁ + v₂ = (x₁ + x₂, y₁ + y₂)$. Esta operación representa la traslación total resultante de aplicar sucesivamente ambos vectores.

### Regla del Paralelogramo

Al completar un paralelogramo cuyos lados adyacentes son las representaciones geométricas de dos vectores, la diagonal representa su suma vectorial.

### Regla del Triángulo

Si una flecha que representa v se dibuja con punto inicial en el punto final de u, entonces la flecha del punto inicial de u al punto final de v representa $u + v$.

### Vector Cero e Inverso Aditivo

El vector $(0, 0)$, denotado $0$, es el vector cero y actúa como elemento idéntico: $v + 0 = v$ para cualquier vector $v$. Su representación geométrica es simplemente un punto. El inverso aditivo o negativo de $v = (x, y)$ es $-v = (-x, -y)$. Se cumple que $v + (- v) = 0$. La representación geométrica de $-v$ es colineal con $v$ pero de sentido opuesto.

### Diferencia de Vectores

La diferencia se define como: $v₁ - v₂ = v₁ + (-v₂) = (x₁ - x₂, y₁ - y₂)$. Geométricamente, $v - u$ es "el vector que va de $u$ a $v$". Si $u$, $v$ y $u$ $-v$ forman un triángulo, sus flechas están dirigidas de manera que $v + (u - v) = u$.

## Magnitud y Dirección de un Vector

### Norma o Magnitud

