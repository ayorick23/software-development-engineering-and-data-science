---
Fecha de creación: 2025-09-17T18:00:00
Materia:
  - Matemática II
Fecha de clase: 2025-09-17
---
[[Clase 09 - Área de Región entre Curvas|← Clase anterior]] | [[Clase 11 - Repaso para Parcial|Clase siguiente →]]

# Sólidos de Revolución
(continúa el [[Clase 09 - Área de Región entre Curvas#Cálculo de Volúmenes|Cálculo de Volúmenes]] introducido al final de la Clase 09, usando los mismos elementos representativos: discos, arandelas y capas cilíndricas)

Un **sólido de revolución** es el cuerpo tridimensional que se obtiene al girar una región plana alrededor de una línea recta (llamada **eje de revolución**), que generalmente es uno de los ejes coordenados. El volumen de este sólido se puede calcular integrando el área de sus secciones transversales, siguiendo la misma lógica de acumulación vista en la Clase 09: el volumen total es la suma acumulada de infinitos elementos representativos muy delgados.

## Método de Discos

Se usa cuando la región gira alrededor de un eje y **toca** ese eje (no queda un hueco en el sólido resultante). Cada sección transversal perpendicular al eje de revolución es un disco (círculo) sólido.

Si $f(x) \geq 0$ es continua en $[a, b]$ y la región bajo $f(x)$ gira alrededor del eje $x$, el volumen es:

$$
V = \pi \int_a^b [f(x)]^2 \, dx
$$

Esto se deduce porque cada disco tiene radio $r = f(x)$ y grosor $dx$, por lo que su volumen es $\pi r^2 \, dx = \pi [f(x)]^2 \, dx$, y se acumulan (integran) todos los discos del intervalo $[a,b]$.

## Método de Arandelas (_Washers_)

Se usa cuando la región que gira **no toca** el eje de revolución, dejando un agujero en el centro del sólido. Cada sección transversal es un anillo (arandela): un disco grande con un disco pequeño removido del centro.

Si la región está delimitada por una curva exterior $f(x)$ (radio externo) y una curva interior $g(x)$ (radio interno), con $f(x) \geq g(x) \geq 0$ en $[a,b]$, y gira alrededor del eje $x$:

$$
V = \pi \int_a^b \left( [f(x)]^2 - [g(x)]^2 \right) dx
$$

Es, en esencia, el volumen del sólido generado por $f(x)$ menos el volumen del "hueco" generado por $g(x)$.

## Método de Capas Cilíndricas (_Shells_)

Se usa cuando resulta más conveniente integrar en la dirección **paralela** al eje de revolución en lugar de perpendicular a él — típicamente cuando despejar la función en términos de la otra variable sería complicado. Cada elemento representativo es una capa cilíndrica delgada (como un tubo) en lugar de un disco.

Si la región bajo $f(x) \geq 0$ en $[a, b]$ (con $0 \leq a$) gira alrededor del eje $y$, el volumen de cada capa es $2\pi (\text{radio})(\text{altura})(\text{grosor})$, y el volumen total es:

$$
V = 2\pi \int_a^b x \cdot f(x) \, dx
$$

- $x$: distancia del elemento al eje de revolución (radio de la capa).
- $f(x)$: altura de la capa.

## ¿Cuándo Usar Cada Método?

- **Discos/Arandelas:** cuando es sencillo expresar el radio (o los radios) en función de la variable de integración, y las secciones transversales perpendiculares al eje son fáciles de describir.
- **Capas Cilíndricas:** cuando despejar la variable "contraria" sería difícil o generaría una función a trozos, o cuando el eje de revolución no coincide con ninguno de los bordes de la región.
