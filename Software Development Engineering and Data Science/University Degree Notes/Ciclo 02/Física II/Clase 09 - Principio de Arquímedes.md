---
Fecha de creación: 2025-09-09T18:05:00
Materia:
  - Física II
Fecha de clase: 2025-09-09
---
[[Clase 08 - Presión de Fluidos|← Clase anterior]] | [[Clase 10 - Ecuación de Bernoulli|Clase siguiente →]]

# Principio de Arquímedes
(repasa la [[Clase 07 - Mecánica de Fluidos#Presión Hidrostática ($P_{hidro​}$)|Presión Hidrostática]] y el [[Clase 08 - Presión de Fluidos#Principio de Pascal|Principio de Pascal]] de las clases 07 y 08 antes de introducir la dinámica de fluidos)

El **principio de Arquímedes** es fundamental para entender por qué los objetos flotan o se hunden. Afirma que un cuerpo sumergido, total o parcialmente en un fluido, experimenta un **empuje** vertical y hacia arriba, igual al peso del fluido que desaloja.

## Estática de Fluidos

La **estática de fluidos** estudia los fluidos en reposo. Un concepto clave es la presión, que es una fuerza que actúa de manera uniforme en todas direcciones.

### Presión

La presión ($P$) se define como la fuerza ($F$) aplicada perpendicularmente a una superficie dividida por el área ($A$) de esa superficie.

$$
P = \frac{F}{A}
$$

### Presión Hidrostática

Es la presión que un fluido ejerce en un punto debido a su peso. A mayor profundidad, mayor es la presión.

$$
P_{hidro} = \rho ⋅ g ⋅ h
$$

- $\rho$ es la densidad del fluido (en $kg/m^3$).
- $g$ es la aceleración de la gravedad.
- $h$ es la profundidad (en metros).

### Principio de Pascal

Este principio establece que un cambio de presión aplicado a un fluido incompresible y en reposo se transmite sin disminuir a cada punto del fluido y a las paredes del recipiente que lo contiene.

**Aplicación:** Las prensas hidráulicas, donde una pequeña fuerza en un área pequeña puede levantar un gran peso en un área grande.

$$
\frac{F_1}{A_1} = \frac{F_2}{A_2}
$$

- $F1$​ y $A1$​ son la fuerza y el área del pistón pequeño.
- $F2$​ y $A2$​ son la fuerza y el área del pistón grande.

### Empuje Hidrostático ($E$)

La fuerza de empuje es la fuerza hacia arriba que contrarresta el peso de un objeto en un fluido.

$$
E=\rho_{fluido}​ ⋅ g ⋅ V_{sumergido}
​
$$

- $\rho_{fluido}$​ es la densidad del fluido.
- $g$ es la aceleración de la gravedad.
- $V_{sumergido}$​ es el **volumen del cuerpo que está sumergido** en el fluido.

### Flotabilidad ($f$)

La **flotabilidad** es una fuerza ascendente que ejerce un fluido (líquido o gas) sobre un objeto, oponiéndose a la fuerza de gravedad que tiende a hundirlo. Esta fuerza, explicada por el Principio de Arquímedes, es igual al peso del fluido que el objeto desaloja.

$$
\frac{v_f}{v_o} = \frac{\rho_0}{\rho_f} = f
$$

### Casos de Flotación

- **Flotación:** Si el **peso del objeto** ($W$) es **menor** que el empuje ($E$), el objeto flota. El objeto se sumerge hasta que el peso del agua desalojada (el empuje) es igual a su propio peso.
- **Equilibrio:** Si el peso del objeto ($W$) es **igual** al empuje ($E$), el objeto queda suspendido en el fluido.
- **Hundimiento:** Si el peso del objeto ($W$) es **mayor** que el empuje ($E$), el objeto se hunde hasta el fondo.

## Dinámica de Fluidos

La **dinámica de fluidos** estudia los fluidos en movimiento. Consideraremos el **flujo ideal**, que es un modelo simplificado sin fricción ni turbulencia.

### Flujo másico ($ṁ$)

El **flujo másico** (o caudal másico) es la cantidad de masa de una sustancia que atraviesa una sección transversal de referencia en un período de tiempo específico.

$$
ṁ = \frac{m}{t}
$$

### Principio de Continuidad

$$
ṁ_1=ṁ_2
$$

#### Ecuación de Continuidad

Esta ecuación se basa en la **conservación de la masa**. Para un fluido incompresible que fluye a través de un tubo, el caudal (volumen de fluido que pasa por unidad de tiempo) es constante. Si el área de la sección transversal del tubo disminuye, la velocidad del fluido debe aumentar para mantener el caudal.

$$
Q_1 = Q_2
$$

$$
\frac{v_1}{t} = \frac{v_2}{t}
$$

$$
A_1 ⋅ v_1 ​= A_2 ​⋅ v_2​
$$

- $A$ es el área de la sección transversal.
- $v$ es la velocidad del fluido.
