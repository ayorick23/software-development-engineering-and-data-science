---
Fecha de creación: 2025-08-12T18:01:00
Materia:
  - Física II
Fecha de clase: 2025-08-12
---
[[Clase 04 - Movimiento Ondulatorio|← Clase anterior]] | [[Clase 07 - Mecánica de Fluidos|Clase siguiente →]]

# ¿Qué es el Sonido?
(aplica el [[Clase 04 - Movimiento Ondulatorio|Movimiento Ondulatorio]] de la Clase 04 a las ondas mecánicas longitudinales)

El sonido es una onda mecánica longitudinal que se propaga a través de un medio elástico (sólido, líquido o gas). Se produce por la vibración de un objeto, que causa la compresión y rarefacción de las partículas del medio circundante, creando una onda de presión que viaja desde la fuente. Los humanos podemos percibir estas ondas de presión dentro de un rango de frecuencia específico, generalmente entre 20 Hz y 20,000 Hz.

## Intensidad del Sonido ($I$)

La **intensidad del sonido** es la potencia por unidad de área que transporta una onda sonora. Se mide en vatios por metro cuadrado ($W/m^2$). Se puede pensar en ella como la cantidad de energía sonora que llega a una superficie cada segundo. Para una fuente puntual que emite sonido de manera uniforme en todas las direcciones, la intensidad disminuye con el cuadrado de la distancia ($r$) a la fuente:

$$
I = \frac{P}{A}
$$

$$
I = \frac{P}{4\pi r^2}
$$

- $P$ es la potencia de la fuente sonora.
- $A$ es el área de la superficie esférica, $A=4\pi r^2$.

Se mide en $[\frac{W}{m^2}]$ que es la intensidad neta del sonido.

## Nivel de Intensidad ($\beta$)

Debido al enorme rango de intensidades que el oído humano puede percibir, es más práctico usar una escala logarítmica para medir la intensidad del sonido. Esta escala se conoce como nivel de intensidad y se mide en decibelios (dB).

El nivel de intensidad ($\beta$) es una medida intermedia.

$$
\beta = 10\log_{10}(\frac{I}{I_0})
$$

- $I$ es la intensidad del sonido que se mide.
- $I_0$ es la intensidad de referencia, que es el umbral de audición humano: $I_0=10^{−12}\frac{W}{m^2}$.

Un aumento de 10 dB en el nivel de intensidad corresponde a un aumento de 10 veces en la intensidad del sonido.

> 0 dB no es audible para el ser humano pero si hay sonido.
> 120 dB es umbral del dolor.

### Tabla de Nivel de Intensidad del Sonido

| Nivel de Intensidad ($\beta$) | Intensidad ($I$) | Ejemplo             |
| ----------------------------- | ---------------- | ------------------- |
| 0 dB                          | $10^{−12} W/m^2$ | Umbral de audición  |
| 20 dB                         | $10^{−10}W/m^2$  | Susurro             |
| 60 dB                         | $10^{−6}W/m^2$   | Conversación normal |
| 100 dB                        | $10^{−2}W/m^2$   | Motocicleta         |
| 120 dB                        | $1W/m^2$         | Umbral de dolor     |
| 140 dB                        | $10^{2}W/m^2$    | Avión despegando    |

## Tono

Forma de la onda.

![[Drawing 2025-08-12 19.45.43.excalidraw]]

## Velocidad del Sonido

La **velocidad del sonido** es la rapidez con la que se propaga una onda sonora a través de un medio. Esta velocidad depende de la rigidez (módulo de elasticidad) y de la densidad del medio.

$$
V = (331) \sqrt{\frac{T}{273}}
$$

Velocidad del sonido a $0\degree C$.

Para $20\degree C$:

$$
V = (331) \sqrt{\frac{Tc + 273}{273}}
$$

$$
V = (331) \sqrt{\frac{20 + 273}{273}} = 342.91 m/s
$$

Velocidad estándar:

$$
v = 343 m/s
$$

El sonido viaja más rápido en sólidos que en líquidos, y más rápido en líquidos que en gases. Esto se debe a que las partículas en sólidos y líquidos están más juntas y la rigidez de los medios es mayor, lo que permite que las vibraciones se transmitan más rápidamente.

## Efecto Doppler

El **efecto Doppler** es el cambio en la frecuencia o la longitud de onda de una onda percibida por un observador en movimiento relativo a la fuente de la onda. Es lo que escuchas cuando una ambulancia se acerca (el tono se hace más agudo) y luego se aleja (el tono se hace más grave).

La ecuación general del efecto Doppler para el sonido es:

$$
f_o = f_s(\frac{v + v_o}{v - v_s})
$$

- $f_o$ es la frecuencia percibida por el observador.
- $f_s$ es la frecuencia real de la fuente.
- $v$ es la velocidad del sonido en el medio.
- $v_o$ es la velocidad del receptor.
- $v_s$ es la velocidad de la fuente.

**Convenciones de Signos:**

- **$v_o$ (Observador):** Se usa $+$ si el receptor se mueve hacia la fuente y $-$ si se aleja.
- **$v_s$ (Fuente):** Se usa $-$ si la fuente se mueve hacia el receptor y $+$ si se aleja.
