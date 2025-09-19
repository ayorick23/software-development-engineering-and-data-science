---
Fecha de creación: 2025-07-08T18:05:00
Materia:
  - Física II
Fecha de clase: 2025-07-08
---
# Unidad 1: Ondas y Acústica

## Movimiento Periódico y Movimiento Oscilatorio

Un **movimiento periódico** es aquel que se repite a intervalos de tiempo regulares. Un ejemplo sencillo es el movimiento de las manecillas de un reloj. El **movimiento oscilatorio** es un tipo de movimiento periódico donde un objeto se mueve de un lado a otro (o "vaivén") alrededor de una posición de equilibrio. Un péndulo que se balancea es un ejemplo clásico. Todos los movimientos armónicos simples son oscilatorios y periódicos, pero no todos los movimientos oscilatorios son armónicos simples.

## Movimiento Armónico Simple (MAS)

El **Movimiento Armónico Simple** es un movimiento oscilatorio y periódico que ocurre cuando la fuerza restauradora que actúa sobre un objeto es directamente proporcional a su desplazamiento desde la posición de equilibrio y siempre apunta hacia esa posición. Esta relación se describe por la Ley de Hooke:

$$
F = -kx
$$

- $F$ es la fuerza restauradora.
- $k$ es la constante de elasticidad del resorte o la constante de fuerza, que mide la rigidez del resorte.
- $x$ es el desplazamiento del objeto desde la posición de equilibrio.

El signo negativo indica que la fuerza siempre actúa en dirección opuesta al desplazamiento, empujando o tirando del objeto de vuelta a la posición de equilibrio.

El **MAS** tiene dos condiciones:

1. El movimiento es periódico y es oscilatorio.
2. La aceleración del movimiento tiene que ser directamente proporcional a la posición del movimiento.

**Oscilatorio:** Cuando empieza en una posición y termina en esa misma posición.

---

### Sistema Masa-Resorte

El sistema masa-resorte es el ejemplo más puro y fundamental para entender el **MAS**. Imagina una masa m unida a un resorte horizontal sin fricción. Cuando la masa se desplaza de su posición de equilibrio ($x=0$) y se suelta, oscila debido a la fuerza restauradora del resorte. La aceleración de la masa en cualquier instante se puede encontrar usando la segunda ley de Newton ($F=ma$):

Se basa en la ley de Hook. Mientras mas sea la longitud de estiramiento, mayor es la fuerza con la que regresa.

La longitud natural ($Ln$) es la posición 0, cuando se estira llega a la posición $A$, cuando se regresa se pasa de la $Ln$ y llega a la posición $-A$ y así sucesivamente.

$$
ma = -kx
$$

$$
a = -\frac{k}{m}x
$$

$$
\frac{d^2x}{dt^2} = -\frac{k}{m}x
$$

Esta ecuación nos muestra que la aceleración es proporcional y opuesta al desplazamiento, que es la condición necesaria para el MAS.

---

Estos tres conceptos son fundamentales para describir las oscilaciones.

### Frecuencia Angular ($\omega$)

Describe la rapidez con la que el objeto oscila en términos de radianes por segundo. Se relaciona directamente con las propiedades del sistema (la masa y la constante del resorte).

$$
\omega = \sqrt{\frac{k}{m}}
$$

### Período ($T$)

Es el tiempo que tarda el objeto en completar una oscilación completa (ida y vuelta). Se mide en segundos. Es el inverso de la frecuencia.

$$
T = \frac{2\pi}{\omega} = 2\pi \sqrt{\frac{m}{k}}
$$

### Frecuncia ($f$)

Es el número de oscilaciones completas que ocurren por unidad de tiempo. Se mide en Hertz ($Hz$), que es igual a $1/s$. Es el inverso del período.

$$
f = \frac{1}{T} = \frac{\omega}{2\pi} = \frac{1}{2\pi}\sqrt{\frac{k}{m}}
$$

---

## Posición, Velocidad y Aceleración en MAS

La posición, velocidad y aceleración de un objeto en MAS cambian constantemente a lo largo del tiempo. Estas se describen mediante funciones sinusoidales. Si consideramos que la oscilación comienza en la posición de máxima elongación ($x_0 = A$), las ecuaciones son:

**Posición $(x(t))$:**

$$
x(t) = A\cos(\omega t+\phi)
$$

- $A$: La amplitud es la distancia máxima que la masa se puede alejar de la Longitud Natural ($Ln$).
- $\omega[\frac{rad}{s}]$: Frecuencia angular, que tan rápido me muevo en una circunferencia.
- $t$: Tiempo.
- $\phi$: Desfase, que tantos radianes se ha alejado de donde debería iniciar.

Otros términos:

- $T[s]$: Periodo, el tiempo en el que un sistema repite una oscilación.
- $f[Hz]$: Frecuencia, cuantas oscilaciones se pueden repetir en una unidad de tiempo.

**Velocidad $v(t))$:** Se obtiene derivando la posición con respecto al tiempo.

$$
v(t) = -A\omega\sin(\omega t+\phi)
$$

La velocidad máxima es $v_{máx}=\omega A$ y ocurre cuando el objeto pasa por la posición de equilibrio $(x=0)$.

**Aceleración $a(t))$:** Se obtiene derivando la velocidad con respecto al tiempo.

$$
a(t) = -A\omega ^2 \cos(\omega t + \phi)
$$

La aceleración máxima es $a_{máx}=\omega^2A$ y ocurre en los puntos de máxima elongación $(x=±A)$.

**NOTA:** Si el movimiento inicia en la amplitud positiva, $\phi$ automáticamente es 0 porque se sabe que el sistema no esta desplazado.

## La Energía en el MAS

La energía total de un sistema en MAS se conserva, asumiendo que no hay fricción. La energía se transforma continuamente entre energía potencial elástica ($U$) y energía cinética ($K$).

### Energía Potencial Elástica ($U$)

Es la energía almacenada en el resorte debido a su compresión o estiramiento. Es máxima en los puntos de máxima elongación ($x=±A$) y cero en la posición de equilibrio ($x=0$).

$$
U = \frac{1}{2}kx^2
$$

### Energía Cinética ($K$)

Es la energía del movimiento de la masa. Es máxima en la posición de equilibrio ($x=0$) y cero en los puntos de máxima elongación ($x=±A$).

$$
K = \frac{1}{2}mv^2
$$

### Energía Mecánica Total ($E$)

La suma de la energía potencial y cinética. Esta es constante a lo largo del movimiento.

$$
E = K + U = \frac{1}{2}mv^2 + \frac{1}{2}kx^2
$$

En los puntos de máxima elongación, toda la energía es potencial ($E = \frac{1}{2}kA^2$). En la posición de equilibrio, toda la energía es cinética ($E = \frac{1}{2}mv^2_{máx}$). Por lo tanto:

$$
E = \frac{1}{2}kA^2 = \frac{1}{2}mv^2_{máx}
$$

---

Relaciones:

$$
T = \frac{1}{f}
$$

$$
f = \frac{2\pi}{T}
$$
