---
Fecha de creación: 2026-02-11T18:05:00
Materia:
  - Matemática III
Fecha de clase: 2026-02-11
---
[[Clase 03 - Producto Cruz, Rectas y Planos en el Espacio|← Clase anterior]] | [[Clase 05 - Repaso para el Parcial|Clase siguiente →]]

# Superficies Cuádricas
(las superficies aquí descritas serán, en la [[Clase 07 - Funciones de Varias Variables|Clase 07]], reinterpretadas como gráficas de funciones $z=f(x,y)$)

Una **superficie cuádrica** es la gráfica en $\mathbb{R}^3$ de una ecuación de segundo grado en $x$, $y$, $z$. Son la generalización tridimensional de las cónicas (elipses, parábolas, hipérbolas) que ya conoces en el plano. Reconocer su forma general permite visualizar de inmediato cómo luce una superficie sin necesidad de graficarla punto por punto.

## Método de las Trazas

La técnica estándar para identificar y dibujar una superficie cuádrica es examinar sus **trazas**: las curvas de intersección de la superficie con cada uno de los planos coordenados ($xy$, $xz$, $yz$) o con planos paralelos a ellos (fijando $x=k$, $y=k$ o $z=k$).

## Elipsoide

$$
\frac{x^2}{a^2}+\frac{y^2}{b^2}+\frac{z^2}{c^2} = 1
$$

Todas sus trazas son elipses. Si $a=b=c$, se reduce a una esfera de radio $a$.

## Paraboloide Elíptico

$$
z = \frac{x^2}{a^2}+\frac{y^2}{b^2}
$$

Sus trazas horizontales ($z=k$, $k>0$) son elipses; sus trazas verticales son parábolas. Tiene forma de "copa". Ya apareció como ejemplo en la [[Clase 07 - Funciones de Varias Variables|Clase 07]] con $f(x,y)=x^2+2y^2$.

## Paraboloide Hiperbólico ("Silla de Montar")

$$
z = \frac{x^2}{a^2}-\frac{y^2}{b^2}
$$

Sus trazas horizontales son hipérbolas; sus trazas verticales son parábolas (algunas hacia arriba, otras hacia abajo). Es la superficie clásica de "silla de montar", con un punto de silla en el origen — relevante para clasificar puntos críticos que no son ni máximos ni mínimos.

## Hiperboloide de Una Hoja

$$
\frac{x^2}{a^2}+\frac{y^2}{b^2}-\frac{z^2}{c^2} = 1
$$

Superficie conexa, en forma de "tubo" que se ensancha hacia los extremos. Sus trazas horizontales son elipses; sus trazas verticales son hipérbolas.

## Hiperboloide de Dos Hojas

$$
-\frac{x^2}{a^2}-\frac{y^2}{b^2}+\frac{z^2}{c^2} = 1
$$

A diferencia del anterior, consta de **dos superficies separadas** (sin puntos entre $-c<z<c$). El signo negativo en dos de los tres términos es lo que distingue "una hoja" de "dos hojas".

## Cono Elíptico

$$
\frac{x^2}{a^2}+\frac{y^2}{b^2} = \frac{z^2}{c^2}
$$

Sus trazas horizontales son elipses que crecen linealmente con $|z|$; sus trazas verticales que pasan por el origen son pares de rectas que se cruzan, formando el vértice del cono en el origen.

## Cilindro (Caso Degenerado)

Cuando la ecuación de una cónica en el plano ($x^2+y^2=r^2$, por ejemplo) carece de una de las tres variables, la superficie resultante es un **cilindro**: la curva se extiende indefinidamente en la dirección de la variable ausente.

$$
x^2+y^2 = r^2 \quad \text{(cilindro circular, se extiende a lo largo del eje } z\text{)}
$$

## Tabla Resumen

| Superficie | Ecuación | Traza Horizontal | Traza Vertical |
| --- | --- | --- | --- |
| Elipsoide | $\frac{x^2}{a^2}+\frac{y^2}{b^2}+\frac{z^2}{c^2}=1$ | Elipse | Elipse |
| Paraboloide elíptico | $z=\frac{x^2}{a^2}+\frac{y^2}{b^2}$ | Elipse | Parábola |
| Paraboloide hiperbólico | $z=\frac{x^2}{a^2}-\frac{y^2}{b^2}$ | Hipérbola | Parábola |
| Hiperboloide de 1 hoja | $\frac{x^2}{a^2}+\frac{y^2}{b^2}-\frac{z^2}{c^2}=1$ | Elipse | Hipérbola |
| Hiperboloide de 2 hojas | $-\frac{x^2}{a^2}-\frac{y^2}{b^2}+\frac{z^2}{c^2}=1$ | Elipse (si existe) | Hipérbola |
| Cono elíptico | $\frac{x^2}{a^2}+\frac{y^2}{b^2}=\frac{z^2}{c^2}$ | Elipse | Rectas que se cruzan |
