---
Fecha de creación: 2025-09-23T19:13:00
Materia:
  - Física II
Fecha de clase: 2025-09-23
---
# Viscosidad

La **viscosidad** es una propiedad física de los fluidos que representa la resistencia interna al flujo. Es la "fricción" entre las capas de un fluido cuando se mueven unas sobre otras.

## Flujo Laminar y Flujo Turbulento

| Régimen de Flujo     | Concepto                                                                                                                                             | Características                                                                                                               |
| :------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------- |
| **Flujo Laminar**    | Ocurre a bajas velocidades. El fluido se mueve en capas o láminas paralelas y ordenadas, sin mezcla transversal.                                     | Movimiento suave y predecible. La velocidad es máxima en el centro del conducto y nula en las paredes (debido a la adhesión). |
| **Flujo Turbulento** | Ocurre a altas velocidades. El movimiento es caótico e irregular, con remolinos y torbellinos (vórtices) que provocan una mezcla intensa del fluido. | Movimiento desordenado y difícil de predecir. Aumenta drásticamente la disipación de energía (pérdida de presión).            |

## Viscosidad Dinámica

La **viscosidad dinámica** ($\eta$) o **viscosidad absoluta** es la medida de la resistencia a la deformación por esfuerzo cortante.

Considera un fluido entre dos placas paralelas, donde la placa superior se mueve a una velocidad constante v y la inferior está fija. El esfuerzo cortante (τ, "tau") es la fuerza aplicada por unidad de área ($\frac{F}{A}$). Este esfuerzo es proporcional al gradiente de velocidad (dv/dy, cambio de velocidad con la altura).

La fórmula de la viscosidad es:

La unidad de medida en el Sistema Internacional (SI) para la viscosidad es el **Pascal-segundo ($Pa⋅s$)**, también llamado **Poiseuille** (aunque menos común). La unidad $dina⋅s/cm^2$ se conoce como **Poise** (P), y un $Pa⋅s$ es igual a 10 Poise.

La sumatoria de las fricciones entre las partículas

Viscosidad Cinemática
$$
F = \eta 
$$

### Cuadro de Viscosidades Comunes

| **Fluido**              | **Temperatura ($\degree C$)** | **Viscosidad ($\eta\times 10^3$)** |
| ----------------------- | ----------------------------- | ---------------------------------- |
| Aire                    | 0                             | 0,0171                             |
|                         | 20                            | 0,0181                             |
|                         | 40                            | 0,0190                             |
|                         | 100                           | 0,0218                             |
| Amoniaco                | 20                            | 0,00974                            |
| Dióxido de Carbono      | 20                            | 0,0147                             |
| Helio                   | 20                            | 0,0196                             |
| Hidrógeno               | 0                             | 0,0090                             |
| Mercurio                | 20                            | 0,0450                             |
| Oxígeno                 | 20                            | 0,0203                             |
| Vapor                   | 100                           | 0,0130                             |
| Agua Líquida            | 0                             | 1,792                              |
|                         | 20                            | 1,002                              |
|                         | 37                            | 0,6947                             |
|                         | 40                            | 0,653                              |
|                         | 100                           | 0,282                              |
| Sangre Completa         | 20                            | 3,015                              |
|                         | 37                            | 2,084                              |
| Plasma Sanguíneo        | 20                            | 1,810                              |
|                         | 37                            | 1,257                              |
| Alcohol Etílico         | 20                            | 1,20                               |
| Metanol                 | 20                            | 0,584                              |
| Aceite (Máquina Pesada) | 20                            | 660                                |
| Aceite (Motor, SAE 10)  | 30                            | 200                                |
| Aceite de Oliva         | 20                            | 138                                |
| Glicerina               | 20                            | 1.500                              |

## Ley de Poiseuille

La **Ley de Poiseuille** (o a veces Ley de Hagen-Poiseuille) describe el flujo laminar de un fluido incompresible y viscoso a través de una tubería cilíndrica de radio constante. Esta ley es fundamental para entender el caudal en sistemas de tuberías y vasos sanguíneos.

La ecuación relaciona el **caudal volumétrico** ($Q$) con la diferencia de presión ($\Delta P$) a lo largo de la tubería, la viscosidad ($\eta$), la longitud de la tubería ($L$) y el radio ($R$).

$$
Q = \frac{P_2-P_1}{R}
$$

$$
Q = \frac{\pi R^4\Delta P}{8\eta L}
$$

- $Q$: Caudal volumétrico ($m^3/s$).
- $\Delta P$: Caída de presión a lo largo de la tubería ($P1​−P2$​).
- $R$: Radio interior de la tubería.
- $\eta$: Viscosidad dinámica del fluido.
- $L$: Longitud de la tubería.

**Observaciones clave de Poiseuille:**

1. El caudal es **directamente proporcional** a la diferencia de presión ($\Delta P$).
2. El caudal es **inversamente proporcional** a la viscosidad ($\eta$) y a la longitud de la tubería ($L$).
3. Lo más importante: El caudal es **extremadamente sensible al radio** ($R^4$). Si duplicas el radio de la tubería, el caudal aumenta en un factor de $2^4=16$.

## Resistencia

$$
R = \frac{8\eta L}{\pi r^4}
$$

Suponiendo que es flujo laminar y el flujo es incompresible.

- **Entidad compresible:** si cambias la dimensión, cambia la densidad.
- **Entidad incompresible:** si cambias la dimensión, no cambia la densidad.

## Turbulencia

El fenómeno de la **turbulencia** marca la transición de un flujo ordenado (_laminar_) a uno caótico (_turbulento_).

## Número de Reynolds ($Re$)

El **Número de Reynolds** es un número adimensional clave en dinámica de fluidos que se utiliza para predecir si un flujo será laminar o turbulento. Representa la razón entre las fuerzas inerciales (las que mantienen el flujo en movimiento) y las fuerzas viscosas (las que se oponen al movimiento).

$$
N_R = \frac{2\rho vr}{\eta}
$$

$$
R_e = \frac{\text{Fuerzas Inerciales}}{\text{Fuerzas Viscosas}} = \frac{\rho vD}{\eta}
$$

- $\rho$: Densidad del fluido ($kg/m^3$).
- $v$: Velocidad promedio del flujo ($m/s$).
- $D$: Dimensión característica (diámetro para un tubo).
- $\eta$: Viscosidad dinámica del fluido ($Pa⋅s$).

### Criterios de Transición para Flujo en Tuberías

Los valores del número de Reynolds determinan el tipo de flujo:

| $N_R<2$       | $2\le N_R\le3$  | $N_R>3$          |
| ------------- | --------------- | ---------------- |
| Flujo Laminar | Flujo Inestable | Flujo Turbulento |
