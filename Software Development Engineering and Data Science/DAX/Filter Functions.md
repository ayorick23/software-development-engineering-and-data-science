# Funciones de Filtro

En **DAX**, las **funciones de filtro** son esenciales para manipular y reducir los datos que se utilizan en los cálculos. Entender el **Contexto de Filtro** es la clave para dominarlas. El contexto de filtro es el conjunto de filtros que ya están aplicados a los datos antes de que se evalúe una expresión, ya sea por una segmentación de datos, una fila en una tabla dinámica o por otra medida.

Las funciones de filtro más importantes son `CALCULATE`, `FILTER` y las que modifican el contexto de filtro, como `ALL`.

## CALCULATE: El motor de DAX

`CALCULATE` es la función más importante en DAX. Su función principal es evaluar una expresión (casi siempre una [[DAX Statements#Medidas Cálculos Dinámicos|medida]] o una [[DAX/Aggregate Functions#Agregaciones Clásicas (Funciones Simples)|función de agregación]]) en un contexto de filtro modificado.

```dax
CALCULATE(<expresión>, [<filtro1>], [<filtro2>], ...)
```

### ¿Cómo funciona?

- Evalúa la expresión inicial.
- Elimina los filtros de la expresión.
- Aplica los nuevos filtros que se pasan como argumentos.
- Vuelve a aplicar el contexto de filtro original.
- Evalúa la expresión final con el nuevo contexto.

### Tipos de filtros en `CALCULATE`

- **Filtros de expresión booleana:** Simples condiciones como `Ventas[Región] = "Norte"`. Son muy eficientes.
- **Filtros de tabla:** Se crean con funciones como `FILTER`, que devuelven una tabla filtrada. Son menos eficientes, pero más flexibles.

## Funciones para Modificar el Contexto

Estas funciones no son filtros por sí mismas, sino que se utilizan como argumentos dentro de `CALCULATE` para eliminar filtros del contexto.

### ALL()

Devuelve todas las filas de una tabla o todos los valores de una o más columnas, ignorando cualquier filtro externo. Se usa dentro de `CALCULATE` para "limpiar" los filtros de una tabla o columna específica.

```dax
ALL([<tabla> | <columna>])
```

### ALLNOBLANKROW()

Devuelve todas las filas de una tabla o todos los valores distintos de una columna, excluyendo la fila en blanco, y ignora los filtros de contexto que puedan existir.

```dax
ALLNOBLANKROW(<tabla> | <columna1>[, <columna2>[, <columna3>[,…]]])
```

### ALLEXCEPT()

Elimina todos los filtros de una tabla, excepto los que se han aplicado a las columnas especificadas como argumentos.

```dax
ALLEXCEPT(<tabla>, <columna1>, <columna2>, ...)
```

### REMOVEFILTERS()

Es una función equivalente a `ALL()` pero con una sintaxis más moderna y fácil de leer. Borra todos los filtros de las tablas o columnas especificadas.

```dax
REMOVEFILTERS([<table> | <columna>])
```

## Otras Funciones de Filtro

### FILTER()

Devuelve una tabla que contiene solo las filas que cumplen la condición de la expresión de filtro. Se utiliza a menudo dentro de `CALCULATE` para aplicar filtros complejos que no se pueden expresar con una simple expresión booleana.

```dax
FILTER(<tabla>, <expresión_de_filtro>)
```

### HASONEVALUE()

Devuelve `TRUE` si el contexto de filtro de la columna especificada se ha reducido a un único valor. Es muy útil para mostrar resultados de una [[DAX Statements#Medidas Cálculos Dinámicos|medida]] solo cuando el usuario ha seleccionado un único elemento en un filtro o en una segmentación de datos.

```dax
HASONEVALUE(<columna>)
```

## Consideraciones Importantes

- **Contexto de filtro:** Las funciones de filtro operan en el contexto de filtro, que es el conjunto de filtros aplicados a los datos antes de la evaluación de la expresión DAX.
- **Eficiencia:** En muchos casos, es más eficiente usar expresiones booleanas en lugar de la función `FILTER` dentro de `CALCULATE` para aplicar filtros.
- **Restricciones:** Las expresiones booleanas utilizadas como filtros en `CALCULATE` tienen algunas restricciones, como no poder hacer referencia a medidas o usar funciones anidadas `CALCULATE`.
