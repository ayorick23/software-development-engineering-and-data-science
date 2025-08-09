---
Fecha de creación: 2025-07-22T18:35:00
Materia:
  - Física II
Fecha de clase: 2025-07-22
---

# Oscilaciones Amortiguadas

Una **oscilación amortiguada** es un movimiento oscilatorio cuya amplitud disminuye gradualmente con el tiempo debido a la disipación de energía del sistema. Esta disipación puede ser causada por fuerzas como la fricción o la resistencia aerodinámica. A diferencia de las oscilaciones no amortiguadas, donde la amplitud se mantiene constante, en las oscilaciones amortiguadas la energía se va perdiendo, lo que provoca que la amplitud de la oscilación disminuya hasta eventualmente detenerse.

Considere las fuerzas que actúan sobre la masa. Note que la única contribución del peso es cambiar la posición de equilibrio. Por lo tanto, la fuerza neta es igual a la fuerza del resorte y la fuerza de amortiguación ($F_D$). Si la magnitud de la velocidad es pequeña, es decir, la masa oscila lentamente, la fuerza de amortiguación es proporcional a la velocidad y actúa contra la dirección del movimiento ($F_D=–bv$). Por lo tanto, la fuerza neta sobre la masa es:

$$
ma = -bv-kx
$$

Al escribir esto como una ecuación diferencial en x, obtenemos:

$$
m\frac{d^2x}{d^2t}+b\frac{dx}{dt}+kx=0
$$

La curva se asemeja a una curva coseno que oscila en una envoltura de una función exponencial $A0e^{–αt}$ donde $\alpha=\frac{b}{2m}$. La solución es:

$$
x(t) = A_0e^{-\frac{b}{2m}t}cos(\omega t+\phi)
$$

Amplitud Original:

$$
A(t) = A_0e^{-\frac{b}{2m}t}
$$

Original:

$$
\omega _0 = \sqrt{\frac{k}{m}}
$$

Amortiguado:

$$
\omega ' = \sqrt{{\omega _0}^2 - (\frac{b}{2m})^2}
$$

## Tipo de Amortiguamiento

| Subamortiguamiento | Amortiguamiento Crítico | Sobreamortiguamiento |
| ------------------ | ----------------------- | -------------------- |
| $\omega '> 0$      | $\omega ' = 0$          | $\omega ' = ?$       |
| $b < 2\sqrt{km}$   | $b = 2\sqrt{km}$        | $b > 2\sqrt{km}$     |

# Oscilaciones Forzadas

Las **oscilaciones forzadas** ocurren cuando un sistema oscilante es impulsado por una fuerza externa periódica, lo que hace que el sistema oscile a la frecuencia de la fuerza externa en lugar de su frecuencia natural. En otras palabras, es cuando se aplica una fuerza externa que varía con el tiempo a un objeto que ya tiene la capacidad de oscilar, como un péndulo o un resorte, y esta fuerza hace que el objeto oscile a una frecuencia diferente a la que lo haría naturalmente.

Al usar la segunda ley de Newton ($F_{neta}=ma$), podemos analizar el movimiento de la masa. La ecuación resultante es similar a la ecuación de fuerza para el oscilador armónico amortiguado, con la adición de la fuerza impulsora:

$$
-kx-b\frac{dx}{dt}+F_0sen(\omega t) = m\frac{d^2x}{dt^2}
$$

$$
F(t) = F_0sen(\omega_D t)
$$

Donde:

- $F(t)$: Fuerza conductora.
- $F_0$: $F_{max}$
- $\omega_D$: Conductora.

Tomando la primera y la segunda derivada temporal de $x(t)$ y sustituyéndolas en la ecuación de fuerza se observa que $x(t)=Asen(\omega t+\phi)$ es una solución siempre que la amplitud sea igual a:

$$
A = \frac{F_0}{\sqrt{m^2(\omega_D^2-\omega_0^2)^2+b^2\omega_D^2}}
$$

## Resonancia

Prolongación del sonido, que se va disminuyendo por grados.

$$
A = \frac{F_0}{\omega^2-\omega_0^2}
$$
