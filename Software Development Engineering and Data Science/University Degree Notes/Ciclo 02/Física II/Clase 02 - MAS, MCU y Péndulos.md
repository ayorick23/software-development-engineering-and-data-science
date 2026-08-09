---
Fecha de creación: 2025-07-15T18:16:00
Materia:
  - Física II
Fecha de clase: 2025-07-15
---
[[Clase 01 - Movimiento Armónico Simple (MAS)|← Clase anterior]] | [[Clase 03 - Oscilaciones Amortiguadas y Forzadas|Clase siguiente →]]

# Comparación entre MAS y MCU

El Movimiento Circular Uniforme (MCU) es el movimiento de un objeto a lo largo de una circunferencia con rapidez constante. Aunque parezca muy diferente al MAS, hay una profunda conexión matemática entre ambos. El MAS es, de hecho, la proyección de un MCU sobre un eje (ya sea el eje x o el eje y).

Imagina un punto $P$ que se mueve en un círculo de radio $A$ a una velocidad angular constante $\omega$. La proyección de este punto sobre el eje $x$ (su coordenada $x$) realiza un Movimiento Armónico Simple.

| Característica                | Movimiento Armónico Simple (MAS)                      | Movimiento Circular Uniforme (MCU)                            |
| ----------------------------- | ----------------------------------------------------- | ------------------------------------------------------------- |
| Trayectoria                   | Rectilínea, de vaivén.                                | Circular, a velocidad constante.                              |
| Velocidad                     | Variable, máxima en el centro y cero en los extremos. | Constante en magnitud, variable en dirección.                 |
| Aceleración                   | Variable, máxima en los extremos y cero en el centro. | Constante en magnitud (centrípeta), dirigida hacia el centro. |
| Analogía                      | La proyección de la posición de un objeto en MCU.     | Movimiento del objeto en un círculo.                          |
| Frecuencia angular ($\omega$) | $\omega = \sqrt{\frac{k}{m}}$                         | $\omega$ = velocidad angular de rotación                      |

La relación entre ellos es clave: la frecuencia angular del MAS es numéricamente igual a la velocidad angular del MCU que lo genera.

## Energía en el MAS

La energía total en un MAS se conserva y es la suma de la energía cinética ($K$) y la energía potencial ($U$).

$$
E_{total} = K + U = \frac{1}{2}mv^2 + \frac{1}{2}kx^2 = constante
$$

Esta energía se puede expresar en términos de la amplitud ($A$) o la velocidad máxima ($v_{max}$):

$$
E_{total} = \frac{1}{2}kA^2 = \frac{1}{2}mv^2_{max}
$$

> **NOTA:** Colocar la notación científica en números grandes

Trabajo se refiere al movimiento.

[[Clase 01 - Movimiento Armónico Simple (MAS)#Movimiento Armónico Simple (MAS)|Amplitud]] positiva ($A$) y negativa ($-A$) van a tener **energía potencial elástica** máxima y su velocidad será 0.
El punto $x = 0$ la velocidad es máxima, por ende, la **energía cinética** será máxima.

---

## Péndulos

Un péndulo es un sistema que oscila bajo la acción de la gravedad. Existen varios tipos de péndulos, y aunque el péndulo simple es una idealización, los otros nos ayudan a entender sistemas más complejos.

### Péndulo Simple

Un **péndulo simple** consiste en una masa puntual ($m$) suspendida de un hilo de longitud ($L$) sin masa. Como viste en la clase anterior, para oscilaciones pequeñas (ángulos menores a $10\degree$), el movimiento es aproximadamente un MAS. La fuerza restauradora es la componente de la gravedad que actúa tangencialmente.

El período ($T$) de oscilación de un péndulo simple está dado por:

$$
T = 2\pi\sqrt{\frac{L}{g}}
$$

- $L$ es la longitud del hilo.
- $g$ es la aceleración de la gravedad.

Es crucial notar que el período no depende de la masa ni de la amplitud (para ángulos pequeños).

Características que debe cumplir:

- Compuesto por cuerda y masa
- La cuerda debe ser ligera (_la masa de la cuerda es 0_)
- La masa debe ser puntual (_un punto fino_)

### Péndulo Físico

Un **péndulo físico** (o péndulo compuesto) es cualquier cuerpo rígido que puede pivotar libremente alrededor de un punto que no es su centro de masa. A diferencia del péndulo simple, la masa no está concentrada en un solo punto.

El período de un péndulo físico está dado por:

$$
T = 2\pi\sqrt{\frac{I}{mgd}}
$$

- $I$ es el momento de inercia del cuerpo con respecto al punto de giro.
- $m$ es la masa total del cuerpo.
- $d$ es la distancia desde el punto de giro hasta el centro de masa del cuerpo.

Se puede notar que si el péndulo físico es una masa puntual ($m$), su momento de inercia sería $I=mL^2$ (donde $L=d$), y la fórmula se simplifica a la del péndulo simple.

### Péndulo de Torsión

Un **péndulo de torsión** consiste en un objeto suspendido de un alambre o varilla. Cuando el objeto se gira un ángulo $\theta$ desde su posición de equilibrio, el alambre ejerce un par de torsión restaurador ($\tau$) que es proporcional al desplazamiento angular.

$$
\tau = -\kappa\theta
$$

- $\kappa$ es la constante de torsión del alambre.
- $\theta$ es el ángulo.

El movimiento de este péndulo también es un MAS angular, y su período está dado por:

$$
T = 2\pi\sqrt{\frac{I}{\kappa}}
$$

- $I$ es el momento de inercia del objeto con respecto al eje de rotación.
- $\kappa$ es la constante de torsión.

Este tipo de péndulo es fundamental en el funcionamiento de relojes antiguos y otros instrumentos de precisión.

Debe estar compuesto por:

- Una fibra delgada.
- Tiene que ser capaz de transmitir torsión a través de la fibra.
- Tiene que estar colocado a un objeto en concreto en el centro.

> No requieren de ángulos pequeños como los péndulos simples y físicos.
