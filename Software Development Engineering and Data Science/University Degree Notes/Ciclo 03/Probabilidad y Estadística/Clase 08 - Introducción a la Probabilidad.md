---
Fecha de creación: 2026-03-13T18:18:00
Materia:
  - Probabilidad y Estadística
Fecha de clase: 2026-03-13
---
# Introducción a la Probabilidad

La **probabilidad** es una rama de la estadística que estudia la posibilidad de que ocurra un determinado evento en situaciones donde el resultado no puede predecirse con certeza.

Se utiliza para analizar fenómenos **aleatorios**, es decir, procesos cuyo resultado depende del azar. La probabilidad permite **cuantificar qué tan probable es que ocurra un evento**, asignándole un valor entre 0 y 1.

Ejemplos donde se utiliza la probabilidad:

- Lanzamiento de una moneda
- Lanzamiento de un dado
- Predicción del clima
- Evaluación de riesgos financieros
- Modelos de predicción en ciencia de datos

## Experimento Aleatorio

Un **experimento aleatorio** es un proceso o acción que puede repetirse bajo las mismas condiciones y cuyo resultado no se puede predecir con certeza antes de realizarlo.

Aunque no se sabe cuál resultado ocurrirá, sí se conocen **todos los posibles resultados**.

## Ejemplos

- Lanzar una moneda.
- Lanzar un dado.
- Extraer una carta de una baraja.
- Seleccionar una persona al azar de un grupo.

En todos estos casos el resultado específico es incierto, pero el conjunto de posibles resultados es conocido.

## Espacio Muestral

El **espacio muestral** es el conjunto de todos los posibles resultados de un experimento aleatorio.

Se representa generalmente con la letra:

$$
S
$$

**Ejemplo:** Lanzamiento de un dado

Si se lanza un dado de seis caras, el espacio muestral es:

$$
S=\{1,2,3,4,5,6\}
$$

Cada número representa un resultado posible.

## Tipos de Eventos

Un **evento** es cualquier subconjunto del espacio muestral.

Es decir, representa uno o varios resultados del experimento.

### Evento simple

Un **evento simple** ocurre cuando el evento contiene **un solo resultado posible**.

**Ejemplo:**

Lanzar un dado y obtener:

$$
A = {3}
$$

Solo hay un resultado.

### Evento compuesto

Un **evento compuesto** contiene **dos o más resultados posibles**.

**Ejemplo:**

**Evento A:** obtener un número par al lanzar un dado.

$$
A=\{2,4,6\}
$$

Aquí el evento puede ocurrir de tres formas diferentes.

## Regla de Laplace

La **Regla de Laplace** se utiliza para calcular probabilidades cuando todos los resultados del experimento tienen **la misma posibilidad de ocurrir**.

La fórmula es:

$$
P(A)=\frac{\text{Casos favorables}}{\text{Casos posibles}}
$$

**Ejemplo:**

Se lanza un dado y queremos calcular la probabilidad de obtener un número par.

Espacio muestral:

$$
S=\{1,2,3,4,5,6\}
$$

Eventos favorables:

$$
A=\{2,4,6\}
$$

Entonces:

$$
P(A) = \frac{3}{6} = \frac{1}{2}
$$

Esto significa que la probabilidad de obtener un número par es **0.5 o 50%**.

## Eventos Mutuamente Excluyentes

Dos eventos son **mutuamente excluyentes** cuando **no pueden ocurrir al mismo tiempo**, es decir, no comparten resultados en común.

Si ocurre uno de ellos, el otro no puede ocurrir.

**Ejemplo con un dado:**

Espacio muestral:

$$
S=\{1,2,3,4,5,6\}
$$

**Evento A:** obtener número **par**

$$
A=\{2,4,6\}
$$

**Evento B:** obtener número **impar**

$$
B=\{1,3,5\}
$$

En este caso:

- No existe ningún número que sea par e impar al mismo tiempo.    
- Por lo tanto, **A y B son mutuamente excluyentes**.

### Ejemplo de eventos que NO son mutuamente excluyentes

**Evento A:** número par

$$
A=\{2,4,6\}
$$

**Evento C:** número mayor que 3

$$
C=\{4,5,6\}
$$

Aquí existe intersección:

$$
A \cap C = \{4,6\}
$$

Por lo tanto, **no son mutuamente excluyentes** porque comparten resultados.

## Operaciones entre Eventos

En probabilidad se utilizan operaciones entre conjuntos.

### Unión de eventos

La unión representa que ocurra **A o B**.

Se escribe:

$$
A \cup B
$$

Incluye todos los resultados que pertenecen a A, a B o a ambos.

### Intersección de eventos

La intersección representa que ocurran **A y B al mismo tiempo**.

Se escribe:

$$
A \cap B
$$

Incluye solo los elementos que pertenecen a ambos eventos.

## Representación con Diagramas de Venn

Los **diagramas de Venn** se utilizan para representar visualmente los eventos dentro del espacio muestral.

- El rectángulo representa el **espacio muestral (S)**.
- Los círculos representan los **eventos**.
- Las áreas donde se superponen muestran la **intersección de eventos**.

Estos diagramas ayudan a entender las relaciones entre eventos.

## Axiomas de Probabilidad

Los axiomas son las reglas fundamentales que rigen la teoría de la probabilidad.

Sea $A$ un evento dentro del espacio muestral $S$.

### Axioma 1 — No negatividad

La probabilidad de cualquier evento nunca puede ser negativa.

$$
P(A) \ge 0
$$

### Axioma 2 — Probabilidad del espacio muestral

La probabilidad de que ocurra **algún resultado del espacio muestral** es igual a 1.

$$
P(S) = 1
$$

Esto significa que siempre ocurrirá algún resultado posible.

### Axioma 3 — Adición para eventos mutuamente excluyentes

Si dos eventos no pueden ocurrir al mismo tiempo, entonces:

$$
P(A \cup B) = P(A) + P(B)
$$

**Ejemplo:**

Si:

$$
P(A)=0.5
$$
$$
P(B)=0.5
$$
Entonces:

$$
P(A \cup B) = 1
$$

>[!IMPORTANT] **IMPORTANTE:** En probabilidades nunca obtenemos valores negativos ni mayores que 1.
