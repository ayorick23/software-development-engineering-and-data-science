---
Fecha de creación: 2025-10-22T18:09:00
Materia:
  - Matemática II
Fecha de clase: 2025-10-22
---
[[Clase 14 - Sustitución Trigonométrica|← Clase anterior]]

# Integración por Partes
(ver [[Derivadas e Integrales Fundamentales#Integración por Partes|Integración por Partes]])

Es el método análogo a la regla del producto en derivación, y se usa cuando el integrando es un **producto de dos funciones de tipo distinto** (por ejemplo, un polinomio multiplicado por una exponencial, o un logaritmo).

## Fórmula General

$$
\int u \, dv = uv - \int v \, du
$$

(Regla mnemotécnica: "Un Día Vi Una Vaca Sin Cola Vestida De Uniforme")

## Regla ILATE / LIATE para Elegir $u$

La dificultad principal del método es decidir qué parte del integrando es $u$ (lo que se deriva) y cuál es $dv$ (lo que se integra). La regla ILATE da un orden de prioridad para elegir $u$: se elige como $u$ la función que aparezca primero en esta lista.

| Prioridad | Tipo de función        | Ejemplo                    |
| --------- | ----------------------- | --------------------------- |
| **I**     | Inversa Trigonométrica | $\arctan x, \arcsin x$     |
| **L**     | Logarítmica            | $\ln x, \log_a x$          |
| **A**     | Algebraica             | $x^2, 3x, (x+1)$           |
| **T**     | Trigonométrica         | $\sin x, \cos x, \tan x$   |
| **E**     | Exponencial            | $e^x, a^x$                 |

La razón: se busca que $u$ sea algo que se **simplifique** al derivar (como un logaritmo o un polinomio) y que $dv$ sea algo fácil de **integrar** (como una exponencial o una función trigonométrica).

## Ejemplo

Calcular $\int x e^x \, dx$:

Por ILATE, entre Algebraica ($x$) y Exponencial ($e^x$), la Algebraica tiene prioridad, así que:

$$
u = x \implies du = dx
$$

$$
dv = e^x dx \implies v = e^x
$$

Aplicando la fórmula:

$$
\int x e^x \, dx = x e^x - \int e^x \, dx = xe^x - e^x + C
$$

## Integración por Partes Repetida (Tabular)

Cuando la función algebraica es un polinomio de grado mayor a 1 (por ejemplo $x^2 e^x$), es necesario aplicar la fórmula varias veces, derivando el polinomio hasta llegar a cero. El **método tabular** organiza este proceso en una tabla de derivadas sucesivas de $u$ e integrales sucesivas de $dv$, alternando signos, para evitar repetir la fórmula manualmente en cada paso.
