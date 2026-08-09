---
Fecha de creación: 2025-07-29T18:09:00
Materia:
  - Física II
Fecha de clase: 2025-07-29
---
[[Clase 03 - Oscilaciones Amortiguadas y Forzadas|← Clase anterior]] | [[Clase 05 - Sonido|Clase siguiente →]]

# Movimiento Ondulatorio

## ¿Qué es una Onda?

Una **onda** es una perturbación que se propaga a través de un medio o en el vacío, transportando energía y momento pero no materia. Piensa en una ola en el agua: la ola se mueve hacia la orilla, pero las moléculas de agua simplemente suben y bajan en su lugar. La onda es el movimiento de la perturbación, no de las partículas del medio.

### Tipos de Ondas

Las ondas se pueden clasificar de varias maneras, dependiendo de sus propiedades.

- **Según el medio de propagación:**
  - **Ondas mecánicas.** Requieren de un medio físico para propagarse. No pueden viajar en el vacío. Ejemplos son las ondas de sonido, las ondas en el agua y las ondas sísmicas.
  - **Ondas electromagnéticas.** No necesitan un medio para propagarse y pueden viajar en el vacío. Se componen de campos eléctricos y magnéticos que oscilan y son perpendiculares entre sí. Ejemplos son la luz visible, las ondas de radio, los rayos X y las microondas.
- **Según la dirección de la propagación:**
  - **Ondas longitudinales.** Donde la propagación y la perturbación van en la misma dirección (_en paralelo_). Un ejemplo es el sonido, donde las moléculas de aire se comprimen y se expanden en la misma dirección en la que viaja el sonido.
  - **Ondas transversales.** Donde la propagación es transversal a la perturbación que provoca. Si mueves el extremo de una cuerda hacia arriba y hacia abajo, la perturbación se mueve horizontalmente, mientras que cada punto de la cuerda se mueve verticalmente. Las ondas electromagnéticas son ondas transversales.

> En Física II solo trabajaremos con ondas mecánicas, transversales, unidireccionales y específicamente onda en una cuerda.

Todas las ondas van a moverse con MRU (Movimiento Rectilíneo Uniforme). La velocidad de la propagación no varía.

## Pulsos

Es una perturbación individual que viaja a través de un medio. Piensa en un solo movimiento rápido de una cuerda que genera una sola "montaña" de onda. Un pulso transporta energía.

## Onda viajera

Es una serie continua de pulsos periódicos. Si mueves el extremo de una cuerda repetidamente con Movimiento Armónico Simple, generas una onda viajera continua, generalmente de forma senoidal. La mayoría de los fenómenos ondulatorios que estudiamos, como el sonido y la luz, son ondas viajeras.

El valor $\frac{2\pi}{\lambda}$ se define como el número de onda. El símbolo del número de onda es $\kappa$ y tiene unidades de metros inversos, $m^{−1}$:

$$
\kappa = \frac{2\pi}{\lambda}
$$

- $\kappa$: Número de onda.

Cuando la posición de la masa se modeló como $x(t)=Acos(\omega t+\phi)$. El ángulo $\phi$ es un deslizamiento de fase, añadido para tener en cuenta que la masa puede tener condiciones iniciales distintas de $x=+A$ y $v=0$. Por motivos similares, la fase inicial se añade a la función de onda. La función de onda que modela una onda sinusoidal y permite un deslizamiento de fase inicial $\phi$, es:

$$
y(x,t)=Asen(kx±\omega t+\phi)
$$

- $±$:
  - $+$: Izquierda
  - $-$: Derecha

## Velocidad de Propagación

La **velocidad de propagación ($v$)** de una onda es la rapidez con la que la perturbación se mueve a través del medio. Esta velocidad depende de las propiedades del medio, no de la fuente que genera la onda.

La velocidad de una onda periódica se puede calcular con la siguiente ecuación, que relaciona la longitud de onda ($\lambda$) y la frecuencia ($f$):

$$
v = \lambda f
$$

- $\lambda$ (lambda) es la longitud de onda, que es la distancia entre dos puntos idénticos consecutivos de la onda (por ejemplo, de cresta a cresta).
- $f$ (frecuencia) es el número de oscilaciones completas por segundo, medido en Hertz (Hz).

La frecuencia es una propiedad de la fuente de la onda, mientras que la velocidad y la longitud de onda dependen del medio.

$$
v = \sqrt{\frac{P_{elastica}}{P_{lineal}}}
$$

$$
v = \sqrt{\frac{F_r}{\mu}}
$$

## Velocidad y Aceleración del Medio

Velocidad:

$$
v(x,t) = A\omega cos(kx±\omega t + \phi)
$$

Aceleración (derivada, ver [[Derivadas e Integrales Fundamentales#Derivadas Fundamentales|Derivadas Fundamentales]]):

$$
a(x,t) = -A\omega^2 sen(kx±\omega t + \phi)
$$

## Energía en una Onda Periódica

Una onda transporta energía. La energía de una onda es proporcional al cuadrado de su amplitud ($A$) y al cuadrado de su frecuencia ($\omega$ o $f$).

La potencia promedio ($P_{prom}$) transferida por una onda en una cuerda se puede expresar como:

$$
E = K + U
$$

Donde $K = \frac{1}{4}\mu A^2\omega^2\lambda$ y $U = \frac{1}{4}\mu A^2\omega^2\lambda$.

$$
E = \frac{1}{2}\mu A^2\omega^2v
$$

- $\mu$ (mu) es la densidad de masa lineal de la cuerda (masa por unidad de longitud).
- $A$ es la amplitud de la onda.
- $\omega$ es la frecuencia angular de la onda ($\omega=2\pi f$).
- $v$ es la velocidad de propagación.

Esta ecuación demuestra que si duplicas la amplitud de una onda, la potencia que transporta se cuadruplica. Esto es muy importante, por ejemplo, para entender la intensidad del sonido. Un sonido más fuerte (mayor amplitud) transporta mucha más energía.
