---
Fecha de creación: 2025-09-16T17:54:00
Materia:
  - Física II
Fecha de clase: 2025-09-16
---
# Ecuación de Bernoulli

La **Ecuación de Bernoulli** es un principio fundamental de la dinámica de fluidos que describe la relación entre la presión, la velocidad y la altura de un fluido en movimiento. Es una expresión de la ley de la conservación de la energía, aplicada a un fluido ideal (incompresible y no viscoso) en un flujo estacionario y laminar.

$$
P_1 ​+ \frac{1}{2} \rho v_1^2 ​+ \rho gh_1 ​= P_2 ​+ \frac{1}{2}​\rho v_2^2​ + \rho gh_2​
$$

- $P$ es la presión del fluido.
- $\frac{1}{2}​ρv^2$ es la **presión dinámica** (relacionada con la energía cinética).
- $ρgh$ es la **presión hidrostática** (relacionada con la energía potencial).

La ecuación establece que la suma de la presión estática, la presión dinámica y la presión hidrostática es constante a lo largo de cualquier línea de corriente en el flujo de un fluido ideal. Esto significa que si la velocidad de un fluido aumenta, su presión estática disminuye, y viceversa.

> Las presiones que aparece en la ecuación de Bernoulli son absolutas.

## Aplicaciones de la Ecuación de Bernoulli

La Ecuación de Bernoulli tiene numerosas aplicaciones prácticas en la ingeniería y la física. Aquí se detallan algunas de las más importantes.

### Principio de Venturi

El **Principio de Venturi** describe el comportamiento de un fluido que se mueve a través de una tubería con una sección transversal que se estrecha.

$$
P_1 ​+ \frac{1}{2}​\rho v_1^2 ​= P_2 ​+ \frac{1}{2}\rho v_2^2​
$$

Cuando el fluido pasa por la parte estrecha (la "garganta"), su velocidad aumenta para mantener el mismo caudal. Según la Ecuación de Bernoulli, este aumento de velocidad provoca una disminución en la presión del fluido.

Esta relación es clave para el funcionamiento del tubo de Venturi, un dispositivo utilizado para medir la velocidad de un fluido o el caudal. La diferencia de presión entre la parte ancha y la parte estrecha de la tubería se puede medir, y a partir de esta diferencia, se puede calcular la velocidad del fluido.

---

### Tubo de Pitot

El **Tubo de Pitot** es un instrumento diseñado para medir la velocidad de un fluido, como la velocidad del aire de un avión o el flujo de agua en un río.

El Tubo de Pitot mide la diferencia entre la presión total (o de estancamiento) y la presión estática del fluido. La presión total se mide en un punto donde el fluido se detiene completamente (punto de estancamiento), y la presión estática se mide en un punto donde el fluido se mueve libremente.

Aplicando la Ecuación de Bernoulli entre un punto de flujo libre (punto 1) y el punto de estancamiento (punto 2), obtenemos:

$$
P_1 ​+ \frac{1}{2}​\rho v_1^2 ​= P_2
$$

Donde $P_2$ es la presión total (presión estática + presión dinámica).

De esta ecuación, podemos despejar la velocidad del fluido ($v_1$):

$$
v = \sqrt{\frac{2\rho fgh}{\rho}}
$$

La diferencia de presión ($P_2 − P_1$) es medida por el tubo de Pitot, lo que permite calcular la velocidad del fluido.

---

### Principio de Torriceli

El **Principio de Torricelli** es una aplicación de la Ecuación de Bernoulli para determinar la velocidad de salida de un líquido a través de un orificio en un recipiente.

Considera un tanque con un orificio pequeño a una profundidad h por debajo de la superficie del líquido. Aplicando la Ecuación de Bernoulli entre la superficie superior (punto 1) y el orificio (punto 2).

Si el tanque es muy grande, la velocidad de descenso del nivel del líquido ($v_1$) es despreciable. Asumiendo que la presión en la superficie y en el orificio es la presión atmosférica ($P_1=P_2$), y que la altura en el orificio es $h_2=0$, la ecuación se simplifica a:

$$
\frac{1}{2}\rho v^2 = \rho gh
$$

Despejando la velocidad de salida ($v$), obtenemos la fórmula de Torricelli:

$$
v = \sqrt{2gh}
$$

Esta ecuación indica que la velocidad de salida del fluido es la misma que la de un objeto que cae libremente desde una altura $h$.
