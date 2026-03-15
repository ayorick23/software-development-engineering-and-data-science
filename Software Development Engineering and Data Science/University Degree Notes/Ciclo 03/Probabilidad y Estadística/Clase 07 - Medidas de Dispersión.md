---
Fecha de creación: 2026-03-06T18:13:00
Materia:
  - Probabilidad y Estadística
Fecha de clase: 2026-03-06
---
# Medidas de Dispersión

En estadística, además de conocer **el centro de los datos** mediante medidas de tendencia central (media, mediana y moda), también es importante analizar **qué tan dispersos o concentrados están los valores** alrededor de ese centro.

Las **medidas de dispersión** permiten evaluar la **variabilidad de un conjunto de datos**, es decir, cuánto se alejan los valores respecto a la media u otra medida central.

Por ejemplo, dos conjuntos de datos pueden tener la misma media pero comportarse de forma muy diferente si uno está muy concentrado y el otro muy disperso.

## Diferencia entre Medidas de Tendencia Central y Medidas de Dispersión

Las **medidas de tendencia central** describen el punto donde se concentran los datos.

Principales medidas:

- **Media (promedio):** suma de todos los valores dividida entre el número de datos.
- **Mediana:** valor central cuando los datos están ordenados.
- **Moda:** valor que aparece con mayor frecuencia.

Las **medidas de dispersión**, en cambio, indican **qué tan alejados están los datos de ese centro**.

En conjunto, ambas permiten comprender mejor la distribución de los datos.

## Tipos de Datos en Estadística

Antes de calcular medidas estadísticas, es importante distinguir entre dos tipos de organización de datos.

### Datos no agrupados

Son datos individuales sin clasificación en intervalos.

Ejemplo:

```shell
10, 12, 14, 15, 16
```

En estos casos los cálculos se realizan directamente sobre los valores.

### Datos agrupados

Los datos se organizan en **intervalos de clase** y se acompañan de una frecuencia.

Ejemplo:

|Intervalo|Frecuencia|
|---|---|
|10 – 15|4|
|16 – 20|6|
|21 – 25|3|

Este tipo de datos se utiliza cuando hay **muchos valores** y es más práctico resumirlos en tablas.

## Rango

El **rango** es la medida de dispersión más simple.

Representa la diferencia entre el valor máximo y el valor mínimo del conjunto de datos.

**Fórmula:**

$$
\text{Rango} = \text{Valor máximo - Valor mínimo}
$$

## Varianza

La **varianza** mide cuánto se alejan los datos respecto a la media.

Si la varianza es:

- **Pequeña:** los datos están cerca de la media.
- **Grande:** los datos están más dispersos.

**Fórmula:**

$$
s^2 = \frac{\sum(x_i - \bar{x})^2}{n-1}
$$

Donde:

- $x_i$​ = cada dato
- $\bar{x}$ = media
- $n$ = número de datos

## Desviación Estándar

La **desviación estándar** es la raíz cuadrada de la varianza.

Se utiliza más que la varianza porque se expresa en **las mismas unidades que los datos originales**.

**Fórmula:**

$$
s = \sqrt{s^2}
$$

## Coeficiente de Variación

El **coeficiente de variación** permite comparar la dispersión entre diferentes conjuntos de datos.

Se expresa generalmente como porcentaje.

**Fórmula:**

$$
CV = \frac{\text{Desviación estándar}}{\text{Media}}\times100
$$

### Interpretación

- CV bajo → datos más consistentes.
- CV alto → mayor variabilidad.

## Medidas de Dispersión para Datos Agrupados

Cuando los datos están en intervalos, se utiliza el **punto medio de cada clase**.

Ejemplo:

|Intervalo|Punto medio|Frecuencia|
|---|---|---|
|10–15|12.5|4|
|16–20|18|6|
|21–25|23|3|

El punto medio se calcula:

$$
\text{Punto medio} = (\text{Límite inferior}\times\text{Límite superior}/2)
$$

Luego se usan esos valores en las fórmulas de media, varianza y desviación estándar multiplicados por su frecuencia.

## Cálculo en Excel

Excel facilita el cálculo de medidas de dispersión.

### Varianza

Para datos muestrales:

```excel
=VAR.S(A1:A10)
```

### Desviación Estándar

```excel
=DESVEST.M(A1:A10)
```

o bien:

```excel
=RAIZ(varianza)
```

### Coeficiente de Variación

Se calcula manualmente:

```excel
=DESVEST.M(A1:A10) / PROMEDIO(A1:A10)
```

## Interpretación de las Medidas de Dispersión

Las medidas de dispersión permiten responder preguntas como:

- ¿Los datos están concentrados o dispersos?
- ¿Qué tan confiable es el promedio?
- ¿Cuál conjunto de datos es más consistente?

Por ejemplo:

**Conjunto A**
- Media = 50 
- Desviación estándar = 2

**Conjunto B**
- Media = 50
- Desviación estándar = 15

Aunque ambos tienen la misma media, el conjunto **A es mucho más consistente**.
