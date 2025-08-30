# Glosario Semana 1

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

# Glosario Semana 2

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

# Glosario Semana 3

## Razón de Cambio Instantánea

La interpretación física de la derivada. Mide exactamente qué tan
rápido está cambiando una cantidad en un instante específico de tiempo o en un punto particular.

## Recta Tangente

Una línea recta que "apenas toca" a una curva en un solo punto, compartiendo la misma dirección o pendiente que la curva en ese punto.

## Pendiente (Slope)

El grado de inclinación de una recta. En el contexto del cálculo, la pendiente de la recta tangente a una curva en un punto es el valor de la derivada en ese punto.

## Regla del Producto

Fórmula utilizada para encontrar la derivada de un producto de dos funciones. Si $h(x) = f(x)g(x)$, entonces:

$$
h'(x) = f'(x)g(x) + f(x)g'(x)
$$

## Regla del Cociente

Fórmula para derivar una división de dos funciones. Si $h(x) = \frac{f(x)}{g(x)}$, entonces:

$$
h'(x) = \frac{f'(x)g(x) - f(x)g'(x)}{[g(x)]^2}
$$

## Número $e$

Una constante matemática irracional (aproximadamente 2.71828...) que es la base del logaritmo natural y de la función exponencial $e^x$. La función $f(x)=e^x$ tiene la propiedad única de ser su propia derivada.

## Derivada Interna

En la regla de la cadena aplicada a una función $f(u)$, es la derivada de la función "interna" $u$. Se presentan como $u'$.

## Velocidad Instantánea

La velocidad de un objeto en un momento específico. Si la posición de un objeto está dada por una función del tiempo $p(t)$, su velocidad instantánea es la derivada $p'(t)$.

# Glosario Semana 4

## Función Explícita

Es una función en la que la variable dependiente (gneralmente $y$) está despejada en términos de la variable independiente (generalmente $x$). Tiene la forma $y=f(x)$.

## Función Implícita

Es una relación en la que la variable dependiente no está despejada. La relación entre $x$ e $y$ se define a través de una ecuación, como $F(x,y)=0$.

## Regla de la Cadena (en Derivación Implícita)

Es el principio fundamental que se aplica al derivar términos que cotniene a $y$. Se considera que $y$ es una función de $x$, por lo que la derivada de $f(y)$ es $f'(y.\frac{dy}{dx})$.

## Recta Normal

Es la línea recta perpendicular a la recta tangente en el punto de tangencia. Supendiente es el recíproco negativo de la pendiente de la recta tangente $(m_{normal}=\frac{-1}{m_{tangente}})$.

## Polinomio

Expresión algebraica que consiste en la suma de términos, donde cada término es el producto de un coeficiente y una variable elevada a una potencia entera no negativa.

## Raíz (o Cero)

Un número $c$ (real o complejo) que, al ser sustituido en un polinomio $P(x)$, hace que el resultado sea cero, es decir, $P(c)=0$.

## Multiplicidad de una Raíz

Es el número de veces que una raíz se repite en la factorización del polinomio. Por ejemplo, en $P(x) = (x-2)^3(x+1)$, la raíz $x=2$ tiene multiplicidad 3.

## Teorema Fundamental del Álgebra

Establece que todo polinomio de grado $n >= 1$ tiene exactamente $n$ raíces complejas, contando su multiplicidad.

## Teorema del Factor

Afirma que un número $c$ es una raíz de un polinomio $P(x)$ si y solo si $(x-c)$ es un factor de $P(x)$.

## Teorema del Residuo

Establece que si un polinomio $P(x)$ se divide entre $(x-c)$, el residuo de esa división es igual a $P(c)$.

## División Sintética

Un algoritmo abreviado para dividir un polinomio por un binomio de la forma $(x-c)$. Es una herramienta rápida para probar raíces y reducir el grado del polinomio.

## Teorema de las Raíces Racionales

Proporciona una lista de todas las posibles raíces racionales de un polinomio con coeficiente enteros. Si $\frac{p}{q}$ es una raíz, entonces $p$ es un divisor del término constante y $q$ es un divisor del coeficiente principal.

## Método de Newton

Un algoritmo numérico iterativo utilizado para encontrar aproximaciones cada vez más precisas de las raíces de una función derivable. También se conoce como método de Newton-Raphson.

## Iteración

Cada repetición del proceso en un método numérico. En el método de Newton, cada iteración genera una nueva aproximación ($x_{n+1}$) a partir de la anterior ($x_n$) usando la fórmula $x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}$.

## Convergencia

Se dice que un método iterativo converge si la secuencia de aproximaciones ($x_0, x_1, x_2,...$) se acerca cada vez más al valor real de la raíz a medida que aumenta el número de iteraciones.

## Divergencia

Ocurre cuando la secuencia de aproximaciones de un método iterativo se aleja del valor real de la raíz en lugar de acercarge a él.

## Suposición Inicial ($x_0$)

Es el valor de partida que se elige para comenzar el proceso iterativo del Método de Newton. Una buena suposición inicial, cerca a la raíz real, es clave para asegurar una convergencia rápida y correcta.


# Glosario Semana 5

# Glosario Semana 7

## Partición de un Intervalo

La división de un intervalo cerrado $[a,b]$ en una cantidad finita de subintervalos.

## Integral Definida

El límite de la Suma de Riemann cuando el número de subintervalos tiende a infinito; representa el área neta exacta bajo una curva.

## Aproximación Numérica

El uso de sumas finitas (como las Sumas de Riemann) para estimar el valor de una integral definida, fundamental en la programación de simulaciones y análisis.

## Modelos Matemáticos

Las ecuaciones diferenciales se utilizan para describir fenómenos de la vida real en términos matemáticos.

## Función Continua

Una función que no tiene interrupciones ni saltos en un intervalo determinado.

## Valor Promedio de una Función

La altura media de la gráfica de una función en un intervalo, calculada a través de una integral definida.

## Teorema del Valor Medio para la Integral

El teorema que garantiza que, para una función continua, existe al menos un punto en el intervalo donde el valor de la función es exactamente igual a su valor promedio en ese intervalo.