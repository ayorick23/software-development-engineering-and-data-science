---
Fecha de creación: 2025-07-30T18:01:00
Materia:
  - Matemática II
Fecha de clase: 2025-07-30
---
# Derivación Implícita

La derivación implícita es una técnica que se utiliza para encontrar la derivada de una variable con respecto a otra cuando la función no está en su forma explícita, es decir, no está despejada como $y = f(x)$. En su lugar, tenemos una ecuación que define una relación entre $x$ e $y$, como $x^2 + y ^2 = 25$.

El método consiste en derivar ambos lados de la ecuación con respecto a $x$ y luego resolver algebraicamente para $\frac{dy}{dx}$ . La clave es recordar que $y$ es una función de $x (y = y(x))$, por lo que al derivar términos que contienen a $y$, debemos aplicar la Regla de la Cadena.

## Pasos para Derivar Implícitamente

1. **Derivar ambos lados:** Aplica el operador de derivada $\frac{d}{dx}$ a ambos lados de la ecuación.
2. **Aplicar la Regla de la Cadena:** Cuando derives un término con la variable $y$, multiplica el resultado por $\frac{dy}{dx}$ . Por ejemplo, $\frac{d}{dx}(y^3) = 3y^2 · \frac{dy}{dx}$ .
3. **Agrupar términos:** Mueve todos los términos que contengan $\frac{dy}{dx}$ a un lado de la ecuación y todos los demás términos al otro lado.
4. **Factorizar:** Saca $\frac{dy}{dx}$ como factor común.
5. **Despejar:** Resuelve para $\frac{dy}{dx}$ dividiendo por el factor que lo acompaña.

# Exploración de Raíces de un Polinomio

Una raíz (o un cero) de un polinomio $P(x)$ es un número $c$ (real o complejo) tal que al evaluarlo en el polinomio, el resultado es cero, es decir, $P(c) = 0$. Explorar las raíces de un polinomio es el proceso de encontrar todos estos valores. El **Teorema Fundamental del Algebra** establece que todo polinomio de grado $n ≥ 1$ tiene exactamente $n$ raíces en el conjunto de los números complejos, contando su multiplicidad.

## Teoremas y Métodos de Exploración

**Teorema del Factor:** Un número $c$ es una raíz de $P(x)$ si y solo si $(x − c)$ es un factor de $P(x)$. Esto nos permite reducir el grado del polinomio una vez que encontramos una raíz.

**División Sintética:** Es un método rápido para dividir un polinomio entre un binomio de la forma $(x − c)$. Si el residuo de la división es cero, entonces $c$ es una raíz. El cociente es el polinomio reducido”.

**Teorema de las Raíces Racionales:** Es la herramienta principal para empezar la búsqueda. Si un polinomio con coeficientes enteros tiene una raíz racional $\frac{p}{q}$ (en su forma más simple), entonces $p$ debe ser un divisor del término constante y q debe ser un divisor del coeficiente principal.

# Método de Newton

El Método de Newton (_también conocido como el método de Newton-Raphson_) es un poderoso algoritmo iterativo para encontrar aproximaciones de las raíces de una función real derivable $f(x)$.

La idea geométrica es la siguiente: se comienza con una suposición inicial $x_0$. La siguiente aproximación, $x_1$, se obtiene encontrando el punto donde la recta tangente a la curva $y = f(x)$ en el punto $(x_0, f(x_0))$ cruza el eje $x$. Este proceso se repite, generando una secuencia de aproximaciones que, bajo ciertas condiciones, converge muy rápidamente a la raíz real.

## Fórmula y Pasos del Método

La fórmula iterativa que define el método es:

$$
x_n+1=x_n - \frac{f(x_n)}{f'(xn)}
$$

donde $x_n$ es la aproximación actual y $x_{n+1}$ es la siguiente.

1. **Definir la función:** Escribe la ecuación en la forma $f(x) = 0$.
2. **Calcular la derivada:** Encuentra $f'(x)$.
3. **Elegir una suposición inicial ($x_0$):** Escoge un valor cercano a la raíz que se busca. Un gráfico de la función puede ayudar a elegir un buen punto de partida.
4. **Iterar:** Aplica la fórmula repetidamente para generar $x_1, x_2, x_3, ...$ hasta que la diferencia entre dos aproximaciones sucesivas ($|x_{n+1} − x_n|$) sea suficientemente pequeña para la precisión deseada.

>***Tarea: Hacer un programa en Python que haga el método de Newton.***
