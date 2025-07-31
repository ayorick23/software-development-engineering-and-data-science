---
Fecha de creación: 2025-07-23T18:08:00
Materia:
  - Matemática II
Fecha de clase: 2025-07-23
---


# Derivadas de Funciones Trascendentes

## Derivación de Funciones Trigonométricas

Las funciones trigonométricas describen ciclos y ondas, y sus derivadas nos muestran cómo cambian en cada instante. Para derivarlas, usamos un conjunto de reglas básicas que siempre debes tener a mano.

### Reglas Fundamentales

Aquí están las 6 derivadas esenciales. Si el ángulo es una función $u(x)$ en lugar de solo $x$, aplicamos la **Regla de la Cadena**, multiplicando por la derivada interna $u'$.

$$
\frac{d}{dx}(sin(u)) = cos(u)·u'
$$
$$
\frac{d}{dx}(tan(u)) = sec^2(u)·u'
$$
$$
\frac{d}{dx}(sec(u)) = sec(u)tan(u)·u'
$$
$$
\frac{d}{dx}(cos(u)) = -sin(u)·u'
$$
$$
\frac{d}{dx}(cot(u)) = -csc^2(u)·u'
$$
$$
\frac{d}{dx}(csc(u)) = -csc(u)cot(u)·u'
$$

## Derivación de Funciones Trigonométricas Inversas

Estas funciones nos devuelven un ángulo a partir de un valor. Sus derivadas son expresiones algebraicas, lo cual es un resultado fascinante del cálculo.

### Reglas Fundamentales

Nuevamente, la **Regla de la Cadena** es clave. Si el argumento es una función $u(x)$, sustituimos $x$ por $u$ y multiplicamos por $u'$.

$$
\frac{d}{dx}(arcsin(x)) = \frac{1}{\sqrt{1-u^2}}·u'
$$
$$
\frac{d}{dx}(arctan(x)) = \frac{1}{1+u^2}·u'
$$
$$
\frac{d}{dx}(arcsec(x)) = \frac{1}{|u|\sqrt{u^2-1}}·u'
$$
$$
\frac{d}{dx}(arctan(x)) = \frac{1}{1+u^2}·u'
$$
$$
\frac{d}{dx}(arcos(x)) = -\frac{1}{\sqrt{1-u^2}}·u'
$$
$$
\frac{d}{dx}(arccot(x)) = -\frac{1}{1+u^2}·u'
$$
$$
\frac{d}{dx}(arccsc(x)) = -\frac{1}{|u|\sqrt{u^2-1}}·u'
$$

## Derivación de la Función Exponencial

La función exponencial describe crecimientos (o decrecimientos) muy rápidos. Su derivada tiene una propiedad única y elegante, especialmente la base $e$.

### Reglas Fundamentales

- **Base natural $e$:** La función $e^u$ es su propia derivada, multiplicada por la derivada del exponente (regla de la cadena).
$$
\frac{d}{dx}(e^u) = e^u·u'
$$
- **Otra base $a$:** Para cualquier base positiva $a$, la derivada es la misma función, pero multiplicada por el logaritmo natural de la base y la derivada del exponente.
$$
\frac{d}{dx}(a^u) = a^u·ln(a)·u'
$$

## Interpretación de la Derivada

Más allá de las fórmulas, ¿qué es la derivada? La derivada tiene dos interpretaciones principales que son la base de casi todas sus aplicaciones.

### Interpretación Geométrica

- La derivada de una función $f(x)$ en un punto $x = c$, denotada como $f'(c)$, es la **pendiente de la línea recta tangente** a la gráfica de $f(x)$ en el punto $(c, f(c))$.
- Si $f'(c) > 0$, la función es creciente en ese punto.
- Si $f'(c) < 0$, la función es decreciente en ese punto.
- Si $f'(c) = 0$, la recta tangente es horizontal. Estos son puntos críticos (posibles máximos o mínimos locales).

### Interpretación como Razón de Cambio

- La derivada representa la razón de cambio instantánea de una variable con respecto a otra.
- Es la respuesta a la pregunta: ¿Qué tan rápido está cambiando $y$ en el preciso instante en que cambia $x$?
- Ejemplo Clásico: Si $p(t)$ es la posición de un objeto en el tiempo $t$, entonces su derivada $p'(t)$ es la velocidad instantánea del objeto en el tiempo $t$.