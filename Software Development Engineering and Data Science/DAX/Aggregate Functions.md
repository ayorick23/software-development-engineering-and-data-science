# Funciones de Agregación

En **DAX** (_Data Analysis Expressions_), las funciones de agregación y matemáticas te permiten resumir y analizar grandes volúmenes de datos. Se dividen en dos categorías principales:

- **Funciones de agregación simples:** Operan directamente sobre una columna completa.
- **Funciones de iteración (`X`):** Evalúan una expresión fila por fila antes de realizar la agregación. Estas son las más potentes y flexibles.

## Agregaciones Clásicas (Funciones Simples)
(misma lógica de [[SQL/Aggregate Functions|Aggregate Functions]])

Estas funciones son las que usas para obtener un valor único de una columna. Son rápidas y eficientes para cálculos directos.

### SUM()

Suma todos los números en una columna. Es ideal para calcular totales directos como ventas o ingresos.

```dax
SUM(<columna>)
```

### AVERAGE()

Calcula el promedio (media aritmética) de todos los números en una columna.

```dax
AVERAGE(<columna>)
```

### MEDIAN()

Calcula la mediana de una columna de números.

```dax
MEDIAN(<columna>)
```

### GEOMEAN()

Calcula la media geométrica de una columna numérica. Es útil para promediar tasas de cambio, porcentajes o ratios, donde la multiplicación es más apropiada que la suma para calcular el promedio.

```dax
GEOMEAN(<columna>)
```

### MIN() & MAX()

Devuelven el valor mínimo y máximo de una columna. Son útiles para encontrar el valor más bajo, el más alto o la fecha más antigua/reciente.

```dax
MIN(<columna>), MAX(<columna>)
```

## Funciones de Iteración (Las que terminan en `X`)

Estas son las funciones más importantes en DAX. Te permiten realizar cálculos personalizados en cada fila de una tabla y luego agregar los resultados. Su sintaxis es siempre `FUNCIONX(tabla, expresión)`.

### SUMX()

Evalúa una expresión por cada fila de una tabla y luego suma los resultados. Es esencial para calcular medidas complejas.

- **¿Por qué es clave?** Imagina que quieres calcular las ventas totales. Si simplemente sumas la columna `[Importe]`, podrías perder información. Con `SUMX`, puedes multiplicar el precio por la cantidad para obtener el total por cada línea de pedido y luego sumarlos todos.

```dax
SUMX(<table>, <expression>)
```

### AVERAGEX()

Calcula el promedio de una expresión evaluada sobre cada fila de una tabla.

```dax
AVERAGEX(<table>, <expression>)
```

### MEDIANX()

Calcula la mediana de una expresión.

```dax
MEDIANX(<table>, <expression>)
```

### GEOMEANX()

Calcula la media geométrica de una expresión.

```dax
GEOMEANX(<table>, <expression>)
```

## Funciones de Conteo

Estas funciones te ayudan a contar filas o valores en tus tablas.

### COUNT()

Cuenta el número de celdas que contienen valores (no blancos) en una columna.

```dax
COUNT(<columna>)
```

### COUNTROWS()

Cuenta todas las filas en una tabla. Se usa comúnmente para contar el número de registros o transacciones.

```dax
COUNTROWS([<table>])
```

### COUNTX()

Cuenta el número de filas en una tabla donde una expresión se evalúa como un valor no blanco.

```dax
COUNTX(<table>, <expression>)
```

### DISTINCTCOUNT()

Cuenta el número de valores únicos o distintos en una columna. Es perfecta para saber cuántos clientes únicos o productos únicos tienes.

```dax
DISTINCTCOUNT(<columna>)
```

## Funciones de Lógica y Clasificación

### DIVIDE()

Realiza una división segura, previniendo errores de división por cero. Si el denominador es cero, devuelve el resultado alternativo que definas (o `BLANK()` si no especificas uno).

```dax
DIVIDE(<numerador>, <denominador> [, <resultado_alternativo>])
```

### RANKX()

Devuelve la clasificación de un número dentro de una lista de números. Es una función de iteración que te permite clasificar elementos (productos, clientes, etc.) basándote en una [[DAX Statements#Medidas Cálculos Dinámicos|medida]] o expresión.

```dax
RANKX(<table>, <expression>[, <value>[, <order>[, <ties>]]])
```

**Argumentos clave:**

- `<table>`: La tabla sobre la que iterar.
- `<expression>`: La medida o cálculo que determina la clasificación.
- `<order>`: `ASC` (ascendente) o `DESC` (descendente).
