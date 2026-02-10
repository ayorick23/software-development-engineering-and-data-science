---
Fecha de creación: 2026-02-06T18:25:00
Materia:
  - Probabilidad y Estadística
Fecha de clase: 2026-02-06
---
# Tipos de Gráficos Estadísticos

Los gráficos estadísticos permiten **visualizar datos de forma clara**, detectar patrones, comparar grupos y comunicar resultados de manera efectiva. Elegir el gráfico correcto depende de:

- Tipo de variable (cualitativa o cuantitativa)
- Nivel de medición
- Objetivo del análisis (comparar, explorar, resumir, detectar anomalías)

## Histograma

Representa la distribución de una **variable cuantitativa continua** agrupada en intervalos.

### Características

- Barras **pegadas** (sin espacios)
- Cada barra representa un **intervalo de clase**
- El eje horizontal muestra intervalos
- El eje vertical muestra frecuencias
- Permite identificar:
    - Forma de la distribución
    - Simetría
    - Sesgo
    - Concentración de datos
    - Valores atípicos

### Cuándo usarlo

- Para analizar comportamiento de tiempos, pesos, edades, temperaturas, mediciones físicas.

**Ejemplo:**

> Tiempo (min) que tarda una máquina en ensamblar una pieza.

|Intervalo|fi|
|---|---|
|4–6|5|
|7–9|7|
|10–12|5|
|13–15|3|

### Errores comunes

- Usar histograma para datos categóricos
- Separar las barras
- Usar intervalos desiguales sin ajustar área

## Polígono de Frecuencias

Es una representación lineal que une los puntos medios de cada intervalo de clase.

### Características

- Se basa en la **marca de clase**
- Permite comparar **dos o más distribuciones**
- Muestra tendencias generales
- Puede superponerse fácilmente

### Marca de clase

$$
\text{Marca de clase} = \frac{\text{Límite inferior} + \text{Límite superior}}{2}
$$

**Ejemplo:**

|Intervalo|Marca|fi|
|---|---|---|
|4–6|5|5|
|7–9|8|7|
|10–12|11|5|
|13–15|14|3|

Se grafican los puntos ($5,5$), ($8,7$), ($11,5$), ($14,3$) y se unen.

### Cuándo usarlo

- Comparar producción de dos turnos
- Analizar evolución de resultados entre dos pruebas

## Diagrama de Barras

Representa datos **cualitativos** o **cuantitativos discretos**.

### Características:

- Barras **separadas**
- Cada barra representa una categoría
- Altura proporcional a la frecuencia o porcentaje
- El orden puede ser arbitrario o lógico

**Ejemplos:**

- Tipos de defectos
- Nivel educativo
- Número de productos vendidos por categoría

### Errores comunes:

- Usar barras pegadas (eso sería histograma)
- Usar para datos continuos

## Diagrama Circular (Gráfico de Pastel)

Representa **proporciones relativas** de un total.

### Características:

- El total siempre es 100%
- Útil cuando hay **pocas categorías**
- Visualiza participación o composición

### Cuándo usarlo

- Distribución del presupuesto
- Participación de mercado
- Porcentaje de defectos por tipo

### Errores comunes

- Usarlo con muchas categorías
- Comparar muchos grupos diferentes

## Ojiva de Galton (Curva de Frecuencia Acumulada)

Representa la **frecuencia acumulada**, mostrando cuántos datos están por debajo de cierto valor.

### Características:

- Se construye con los **límites superiores de clase**
- Es una curva creciente
- Finaliza en el total de datos
- Permite identificar:
    - Mediana
    - Percentiles
    - Cuartiles

**Ejemplo:**

|Intervalo|Límite superior|Fi|
|---|---|---|
|4–6|6|5|
|7–9|9|12|
|10–12|12|17|
|13–15|15|20|

Se grafican los puntos $(6,5)$, $(9,12)$, $(12,17)$, $(15,20)$.

### Usos:

- Control de calidad
- Evaluación académica
- Análisis de tiempos de espera
