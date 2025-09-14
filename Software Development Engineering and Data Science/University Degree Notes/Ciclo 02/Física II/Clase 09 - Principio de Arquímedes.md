---
Fecha de creación: 2025-09-09T18:05:00
Materia:
  - Física II
Fecha de clase: 2025-09-09
---
# Principio de Arquímedes

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

## Principio de Arquímedes

El **principio de Arquímedes** es fundamental para entender por qué los objetos flotan o se hunden. Afirma que un cuerpo sumergido, total o parcialmente en un fluido, experimenta un **empuje** vertical y hacia arriba, igual al peso del fluido que desaloja.

### Empuje Hidrostático ($E$)

La fuerza de empuje es la fuerza hacia arriba que contrarresta el peso de un objeto en un fluido.

$$
E=ρ_{fluido}​ ⋅ g ⋅ V_{sumergido}
​$$

- $ρ_{fluido}$​ es la densidad del fluido.
- $g$ es la aceleración de la gravedad.
- $V_{sumergido}$​ es el **volumen del cuerpo que está sumergido** en el fluido.

### Casos de Flotación

- **Flotación:** Si el **peso del objeto** ($W$) es **menor** que el empuje ($E$), el objeto flota. El objeto se sumerge hasta que el peso del agua desalojada (el empuje) es igual a su propio peso.
- **Equilibrio:** Si el peso del objeto ($W$) es **igual** al empuje ($E$), el objeto queda suspendido en el fluido.
- **Hundimiento:** Si el peso del objeto ($W$) es **mayor** que el empuje ($E$), el objeto se hunde hasta el fondo.

## Dinámica de Fluidos

La **dinámica de fluidos** estudia los fluidos en movimiento. Consideraremos el **flujo ideal**, que es un modelo simplificado sin fricción ni turbulencia.

### Ecuación de Continuidad

Esta ecuación se basa en la **conservación de la masa**. Para un fluido incompresible que fluye a través de un tubo, el caudal (volumen de fluido que pasa por unidad de tiempo) es constante. Si el área de la sección transversal del tubo disminuye, la velocidad del fluido debe aumentar para mantener el caudal.

$$
A_1 ⋅ v_1 ​= A_2 ​⋅ v_2​
$$

- $A$ es el área de la sección transversal.
- $v$ es la velocidad del fluido.

### Ecuación de Bernoulli

Esta ecuación es una expresión de la **conservación de la energía** para un fluido ideal en movimiento. Relaciona la presión, la velocidad y la altura del fluido.

$$
P_1 ​+ \frac{1}{2} ​ρv_1^2 ​+ ρgh_1 ​= P_2 ​+ \frac{1}{c}​ρv_2^2​ + ρgh_2​
$$

- $P$ es la presión del fluido.
- $\frac{1}{2}​ρv^2$ es la **presión dinámica** (relacionada con la energía cinética).
- $ρgh$ es la **presión hidrostática** (relacionada con la energía potencial).
