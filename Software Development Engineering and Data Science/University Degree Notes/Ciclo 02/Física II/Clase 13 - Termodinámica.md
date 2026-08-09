---
Fecha de creación: 2025-10-07T18:01:00
Materia:
  - Física II
Fecha de clase: 2025-10-07
---
[[Clase 11 - Viscosidad|← Clase anterior]] | [[Clase 14 - Calor|Clase siguiente →]]

# Unidad 3. Introducción a la Termodinámica

## Termodinámica

La **Termodinámica** es la rama de la física que estudia las relaciones entre el **calor ($Q$)** y otras formas de **energía ($E$)**, así como el trabajo ($W$). Se enfoca en los estados de equilibrio de la materia y cómo la energía se transforma dentro de un sistema.

Es el estudio de tres variables en concreto y las interacciones entre ellas:

- Presión ($Pa$)
- Volumen ($V$)
- Temperatura ($\degree C$, $\degree F$)

## Temperatura

La **temperatura** es una medida de la energía cinética promedio de las partículas que componen un objeto (átomos o moléculas). **No** es una medida de la energía térmica total (calor) del objeto.

### Escalas de Temperatura

- **Celsius ($°C$):** Basada en las propiedades del agua al nivel del mar. Define 0 $°C$ como el punto de congelación y 100 $°C$ como el punto de ebullición.
- **Fahrenheit ($°F$):** Usada comúnmente en Estados Unidos. El punto de congelación del agua es 32 $°F$ y el punto de ebullición es 212 $°F$.
- **Kelvin ($K$):** Es la unidad teórica fundamental de temperatura en el Sistema Internacional (SI) y la escala usada en la física. Es una escala **absoluta**.

En las escalas Celsius y Fahrenheit, el valor de 0 no representa la ausencia de temperatura. $0°C$ es simplemente el punto de congelación del agua.

El **Cero Absoluto** es $0K$, que equivale a $−273.15°C$ y $−459.67°F$. En esta temperatura, el movimiento térmico de las partículas de un sistema se detiene (mínimo movimiento cuántico), lo que representa la **ausencia total de energía térmica**. Por esta razón, la escala Kelvin solo tiene valores positivos.

## Conversión entre Unidades de Temperatura

### Celsius a Fahrenheit

![[Drawing 2025-10-07 18.49.22.excalidraw]]

$$
y = mx + b
$$

$$
m = \frac{\Delta y}{\Delta x} = \frac{212-32}{100-0} = 1.8
$$

$$
T_{\degree F} = 1.8 T_{\degree C} + 32
$$

### Fahrenheit a Celsius

$$
T_{\degree C} = 1.8(T_{\degree F}-32)
$$

### Celsius a Kelvin

![[Drawing 2025-10-07 19.07.53.excalidraw]]



$$
T_K = T_{\degree C} + 273.15
$$

## Diferencia de Temperaturas

Es fundamental distinguir entre la conversión de un valor de temperatura y la conversión de un **cambio (diferencia)** de temperatura.

- **Diferencia de temperatura en $°C$ y $K$:** Dado que las unidades Celsius y Kelvin tienen el mismo tamaño de intervalo (un grado $C$ equivale a un Kelvin), la diferencia es la misma:

$$
\Delta T_{\degree C}=\Delta T_{K}
$$

- **Diferencia de temperatura en $°F$ y $°C$:** La diferencia entre $0°C$ y $1°C$ es $1°C$. La diferencia entre $32°F$ y $33.8°F$ (el equivalente de $1°C$) es $1.8°F$. Por lo tanto:

$$
\Delta T_{\degree F} = \frac{9}{5}\Delta T_{\degree C} = 1.8\Delta T_{\degree C}
$$

$$
\Delta T_{\degree C} = \frac{5}{9}\Delta T_{\degree F}
$$

>No se debe usar la conversión directa con el $+32$ cuando se trabaja con **cambios** de temperatura ($\Delta T$).
### Dilatación

La **dilatación térmica** es el cambio en las dimensiones (longitud, área o volumen) de un cuerpo debido a un cambio de temperatura. Al aumentar la temperatura, las partículas vibran con mayor amplitud, aumentando la distancia promedio entre ellas y, por lo tanto, el tamaño del objeto.

El volumen y la presión atmosférica es directamente proporcional a la temperatura.

- **Dilatación Lineal ($\Delta L$):** Describe el cambio en la **longitud** de un objeto, como una varilla o un alambre.
$$
\Delta L = L_0 \alpha\Delta T
$$

	- $\Delta L$ es el cambio en la longitud.
	- $L_0$​ es la longitud inicial.
	- $\Delta T$ es el cambio de temperatura.
	- $\alpha$ es el **Coeficiente de Dilatación Lineal** (unidad: $°C−1$ o $K−1$).
	
- **Dilatación Superficial ($\Delta A$):** Describe el cambio en el **área** de un objeto bidimensional, como una lámina o placa. El coeficiente de dilatación superficial ($\gamma$) es, aproximadamente, el doble del coeficiente lineal.

$$
\Delta A = A_0 \gamma\Delta T
$$

$$
\gamma\approx 2\alpha
$$

	- $\Delta A$ es el cambio en el área.
	- $A_0$​ es el área inicial.
	- $\gamma$ es el **Coeficiente de Dilatación Superficial**.

- **Dilatación Volumétrica ($\Delta V$):** Describe el cambio en el **volumen** de un objeto tridimensional (sólido, líquido o gas). El coeficiente de dilatación volumétrica ($\beta$) es, aproximadamente, el triple del coeficiente lineal.

$$
\Delta V = V_0 \beta\Delta T
$$

$$
\beta\approx 3\alpha
$$

	- $\Delta V$ es el cambio en el volumen.
	- $V_0$​ es el volumen inicial.
	- $\beta$ es el **Coeficiente de Dilatación Volumétrica**.

Esto no significa que el cuerpo solo se va a expandir en una sola dimensión, sino que en una dimensión es mas relevante que las otras porque sus cambios no son significativos.

### Coeficientes de Dilatación

Los coeficientes de dilatación ($α$, $β$, $γ$) son propiedades **intrínsecas** del material y dependen de su naturaleza.

| Material                            | Coeficiente de Dilatación Lineal $\alpha(1/\degree C)$ | Coeficiente de Expansión (Dilatación) Volumétrica $\beta(1/\degree C)$ |
| ----------------------------------- | ------------------------------------------------------ | ---------------------------------------------------------------------- |
| _Sólidos_                           |                                                        |                                                                        |
| Aluminio                            | $25\times 10^{-6}$                                     | $75\times 10^{-6}$                                                     |
| Latón                               | $19\times 10^{-6}$                                     | $56\times 10^{-6}$                                                     |
| Cobre                               | $17\times 10^{-6}$                                     | $51\times 10^{-6}$                                                     |
| Oro                                 | $14\times 10^{-6}$                                     | $42\times 10^{-6}$                                                     |
| Hierro o Acero                      | $12\times 10^{-6}$                                     | $35\times 10^{-6}$                                                     |
| Invar (aleación de níquel y hierro) | $0,9\times 10^{-6}$                                    | $2,7\times 10^{-6}$                                                    |
