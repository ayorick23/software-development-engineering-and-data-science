# Glosario Matemática II

## Derivada de una Función

La derivada de una función $f(x)$, denotada como $f'(x)$ o $\frac{dy}{dx}$ , mide la tasa de cambio instantánea de la función con respecto a su variable independiente. Geométricamente, representa la pendiente de la recta tangente a la curva de la función en un punto dado.

$$
f'(x) = \lim_{h \to 0} \frac{f(x+h) - f(x)}{h}
$$

## Regla de la Cadena

Es una regla fundamental para derivar funciones compuestas. Si $y = f(g(x))$, entonces su derivada es $y' = f'(g(x))· g'(x)$. En notación de Leibniz, si $y = f(u)$ y $u = g(x)$, entonces $\frac{dy}{dx} = \frac{dy}{du} · \frac{du}{dx}$ . Es crucial para derivar funciones anidadas.

## Derivadas Sucesivas (de Orden Superior)

Son las derivadas de una función obtenidas al derivar repetidamente la función original.
- Primera derivada: $f'(x)$ o $\frac{dy}{dx}$
- Segunda derivada: $f''(x)$ o $\frac{d^2y}{dx}$. Representa la tasa de cambio de la primera derivada y está relacionada con la concavidad de la función.
- Tercera derivada: $f'''(x)$
- Enésima derivada: $f(n)(x)$ o $\frac{d^ny}{dx^n}$

## Funciones Trascendentes

Son funciones que no pueden expresarse como una combinación finita de operaciones algebraicas (suma, resta, multiplicación, división, potencias enteras y raíces) sobre la variable independiente. Incluyen funciones exponenciales, logarítmicas y trigonométricas.

## Función Exponencial Natural

La función $e^x$ , donde $e$ es el número de Euler ($e ≈ 2,71828$). Su derivada es $\frac{d}{dx} (e^x) = e^x$ . Aplicando la regla de la cadena, $\frac{d}{dx} (e^{u(x)}) = e^{u(x)} · u'(x)$.

## Función Logaritmo Natural

La función $ln(x)$, que es el logaritmo en base $e$. Su derivada es $\frac{d}{dx} (ln(x)) = \frac{1}{x}$ para $x > 0$. Aplicando la regla de la cadena, $\frac{d}{dx} (ln(u(x))) = \frac{u'(x)}{u(x)}$ . Es fundamental para la derivación logarítmica.

## Funciones Trigonométricas

Funciones como $sin(x)$, $cos(x)$, $tan(x)$, $csc(x)$, $sec(x)$, $cot(x)$. Sus derivadas son bien conocidas y se utilizan frecuentemente:

- $\frac{d}{dx}(sin(x)) = cos(x)$
- $\frac{d}{dx}(cos(x)) = -sin(x)$
- $\frac{d}{dx}(tan(x)) = sec^2(x)$
- Y sus recíprocas, aplicando la regla de la cadena para argumentos compuestos.

## Funciones Trigonométricas Inversas ($Arc-funciones$)

Son las inversas de las funciones trigonométricas. Se utilizan para encontrar el ángulo cuando se conoce el valor de la razón trigonométrica. Las más comunes son $arcsin(x)$ (o $sin^−1 (x)$), $arc cos(x)$ (o $cos^−1 (x)$) y $arctan(x)$ (o $tan^−1 (x)$).

## Derivada de $Arcsen(x)$

La derivada de la función $arcsin(x)$ es $\frac{d}{dx} (arcsin(x)) = \frac{1}{\sqrt{1−x^2}}$ . Si se aplica la regla de la cadena para $arcsin(u(x))$, es $\frac{u'(x)}{\sqrt{1−(u(x))^2}}$.

## Derivada de $Arctan(x)$

La derivada de la función $arctan(x)$ es $\frac{d}{dx} (arctan(x)) = \frac{1}{1+x^2}$. Si se aplica la regla de la cadena para $arctan(u(x))$, es $\frac{u'(x)}{1+(u(x))^2}$.

## Derivación Implícita

Técnica utilizada para encontrar la derivada de una función cuando $y$ no está explícitamente definida en términos de $x$. Se deriva cada término de la ecuación con respecto a $x$, tratando a $y$ como una función de $x$ y aplicando la regla de la cadena cuando se deriva un término que contiene $y$.

## Derivación Logarítmica

Método para encontrar la derivada de funciones que son complicadas debido a productos, cocientes o potencias donde la base y el exponente son funciones de $x$. Consiste en tomar el logaritmo natural de ambos lados de la ecuación, simplificar usando propiedades de logaritmos y luego derivar implícitamente. Es particularmente útil para funciones de la forma $f(x)^{g(x)}$.

## Derivada

La derivada de una función mide la tasa a la cual el valor de la función cambia con respecto a un cambio en su variable independiente. Representa la pendiente de la recta tangente a la gráfica de la función en un punto dado.

## Primera Derivada ($f'(x)$ o $\frac{dy}{dx}$)

Indica la tasa de cambio instantánea de una función y su monotonicidad (si es creciente o decreciente). Su signo nos dice si la función sube o baja.

## Segunda Derivada ($f''(x)$ o $\frac{d^2y}{dx^2}$)

Mide la tasa de cambio de la primera derivada. Nos informa sobre la concavidad de la función (si es cóncava hacia arriba o hacia abajo) y es crucial para clasificar máximos y mínimos locales.

## Punto Crítico

Un punto $c$ en el dominio de una función $f(x)$ donde $f'(c) = 0$ o $f'(c)$ no existe. Estos puntos son candidatos a ser máximos locales, mínimos locales o puntos de inflexión.

## Máximo Local

Un punto $c$ donde la función $f(x)$ alcanza un valor máximo en una vecindad de $c$. Se identifica si $f'(c) = 0$ y $f''(c) < 0$.

## Mínimo Local

Un punto $c$ donde la función $f(x)$ alcanza un valor mínimo en una vecindad de $c$. Se identifica si $f'(c) = 0$ y $f''(c) > 0$.

## Punto de Inflexión

Un punto en la gráfica de una función donde la concavidad cambia (de cóncava hacia arriba a cóncava hacia abajo, o viceversa). Típicamente ocurre donde $f''(x) = 0$ o $f''(x)$ no existe.

## Creciente

Una función es creciente en un intervalo si, a medida que el valor de la variable independiente aumenta, el valor de la función también aumenta. Se identifica si $f'(x) > 0$.

## Decreciente

Una función es decreciente en un intervalo si, a medida que el valor de la variable independiente aumenta, el valor de la función disminuye. Se identifica si $f'(x) < 0$.

## Cóncava hacia Arriba

La gráfica de una función es cóncava hacia arriba en un intervalo si se "abre" hacia arriba, similar a una taza. Se identifica si $f''(x) > 0$.

## Cóncava hacia Abajo

La gráfica de una función es cóncava hacia abajo en un intervalo si se "abre" hacia abajo, similar a una colina invertida. Se identifica si $f''(x) < 0$.

## Monotonicidad

Describe el comportamiento de una función en términos de si es creciente, decreciente o constante en un intervalo. Se analiza usando la primera derivada.

## Concavidad

Describe la dirección en la que se "curva" la gráfica de una función (hacia arriba o hacia abajo). Se analiza usando la segunda derivada.

