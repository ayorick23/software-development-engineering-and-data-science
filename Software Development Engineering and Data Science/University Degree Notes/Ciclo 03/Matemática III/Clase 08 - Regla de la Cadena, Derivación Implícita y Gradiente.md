---
Fecha de creación: 2026-03-11T18:00:00
Materia:
  - Matemática III
Fecha de clase: 2026-03-11
---
[[Clase 07 - Funciones de Varias Variables|← Clase anterior]] | [[Clase 09 - Derivadas Direccionales, Gradientes y Fórmula de Taylor|Clase siguiente →]]

# Regla de la Cadena, Derivación Implícita y Gradiente
(usa las [[Clase 07 - Funciones de Varias Variables|Funciones de Varias Variables]] introducidas en la Clase 07)

Un pequeño ejemplo aplicado a la ciencia de datos

## Curva Paramétrica

Una **curva paramétrica** es un trayecto en el plano donde ambas coordenadas cartesianas dependen de un tercer parámetro unificador, generalmente el tiempo $t$. Se denota como $x = g(t)$ e $y = h(t)$. En programación de videojuegos, esto dicta el recorrido de un objeto en el tiempo independientemente de la función geométrica.

## Derivada Parcial ($\frac{∂f}{∂x}$)

La **derivada parcial** $\frac{∂f}{∂x}$ representa la tasa de cambio de una función de varias variables con respecto a una de ellas, manteniendo estrictamente constantes las demás. Informáticamente, es medir cómo el resultado de un algoritmo cambia si modifico solo una línea de código y mantengo el resto intacto.

## Notación Diferencial de Leibniz

La **notación de Leibniz** utiliza fracciones de diferenciales (ej. $\frac{dy}{dx}$ ). Es vital para la Regla de la Cadena porque resalta visualmente las variables involucradas, a diferencia de la notación prima ($f' (x)$) que oculta respecto a qué variable se está derivando, lo cual es fatal en programación multivariable.

## Regla de la Cadena: Caso 1

Sea $z = f(x, y)$ una función diferenciable de las variables $x$ e $y$ (denominadas variables intermedias), donde a su vez $x = g(t)$ e $y = h(t)$ son funciones diferenciables de una única variable independiente $t$. La función compuesta $z = f(g(t), h(t))$ es diferenciable respecto a $t$, y su derivada total se define como la suma de los productos de las derivadas parciales por las derivadas ordinarias.

Si z = f(x, y) es diferenciable, y x = x(t), y = y(t) son diferenciables en t, entonces:

$$
\frac{dz}{dt} = \frac{∂f}{∂x}\frac{dx}{dt}+\frac{∂f}{∂y}\frac{dy}{dt}
$$

## Regla de la Cadena: Caso 2

Sea $z = f(x, y)$ una función diferenciable de $x$ e $y$, donde $x = g(u, v)$ e $y = h(u, v)$ son a su vez funciones diferenciables de las variables independientes $u$ y $v$. Entonces, $z$ es una función diferenciable de $u$ y $v$, y sus derivadas parciales están dadas por la suma de los productos de las rutas correspondientes.

Bajo las condiciones de diferenciabilidad previas, las derivadas parciales de z respecto a u y v son:

$$
\frac{∂z}{∂u} = \frac{∂z}{∂x}\frac{∂x}{∂u}+\frac{∂z}{∂y}\frac{∂y}{∂u}
$$
$$
\frac{∂z}{∂v} = \frac{∂z}{∂x}\frac{∂x}{∂v}+\frac{∂z}{∂y}\frac{∂y}{∂v}
$$

## Derivación Implícita Multivariable

Sea una superficie definida implícitamente por una ecuación continua de la forma $F(x, y, z) = 0$. Si $F$ es diferenciable y asumimos que $z$ está definida implícitamente como una función $z = f(x, y)$, entonces las derivadas parciales directas son:

$$
\frac{∂z}{∂x} = -\frac{F_x}{F_z}
$$

y

$$
\frac{∂z}{∂y} = -\frac{F_y}{F_z}
$$

Con la condición estricta de que $F_z\neq0$.

## Gradiente
(se profundiza en [[Clase 09 - Derivadas Direccionales, Gradientes y Fórmula de Taylor|Clase 09]])

El **gradiente** de una función $f(x,y)$, denotado $\nabla f$, es el vector formado por todas sus derivadas parciales de primer orden:

$$
\nabla f(x,y) = \left\langle \frac{\partial f}{\partial x},\ \frac{\partial f}{\partial y} \right\rangle
$$

Geométricamente, en cada punto $(x_0,y_0)$ el vector $\nabla f(x_0,y_0)$ apunta en la dirección en la que $f$ crece más rápidamente, y su magnitud $|\nabla f|$ es esa tasa máxima de crecimiento. Es la generalización multivariable de la derivada ordinaria $f'(x)$, y es la pieza central tanto de la derivada direccional como del método del descenso de gradiente usado para entrenar modelos de machine learning (ver [[Clase 10 - Multiplicadores de Lagrange#Selección de Características mediante Regularización|Selección de Características]]).
