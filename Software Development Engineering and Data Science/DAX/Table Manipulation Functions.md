# Funciones de Manipulación de Tablas

Las **funciones de manipulación de tablas** en DAX te permiten crear, modificar y combinar tablas en la memoria. Son esenciales para realizar análisis complejos, ya que te dan la flexibilidad de transformar tus datos para una pregunta específica, sin alterar tu modelo.

## Agregación y Resumen

Estas funciones te permiten resumir una tabla de datos en una nueva tabla más pequeña, agrupando los datos por una o más columnas y calculando agregaciones para cada grupo.

### SUMMARIZE()

Crea una tabla de resumen. Es ideal para crear tablas de agrupamiento sencillas. Aunque es la función más conocida, a menudo se considera una función heredada debido a su comportamiento en el contexto de la transición de fila.

```dax
SUMMARIZE(<tabla>, <columna_agrupar_por>... , <nombre>, <expresión>...)
```

Rápido y efectivo para agrupaciones simples.

### GROUPBY()
(misma lógica de [[DQL (Data Query Language)#4. `GROUP BY` (_Agrupación de Filas_)|GROUP BY en SQL]])

Similar a `SUMMARIZE`, pero con un comportamiento más predecible y la capacidad de anidar [[DAX/Aggregate Functions#Agregaciones Clásicas (Funciones Simples)|agregaciones]]. Es la función moderna preferida por muchos expertos en DAX para agrupar y resumir datos.

```dax
GROUPBY(<table>, <columna_agrupar_por>..., <nombre>, <expresión>...)
```

Recomendado para crear tablas de resumen.

## Modificación de Tablas

Estas funciones no resumen los datos, sino que crean una nueva versión de una tabla existente con cambios específicos.

### ADDCOLUMNS()

Agrega nuevas [[DAX Statements#Columnas Calculadas Nuevos Atributos en el Modelo|columnas calculadas]] a una tabla existente. Es ideal para enriquecer una tabla con información adicional en tiempo real.

```dax
ADDCOLUMNS(<table>, <nombre>, <expresión>...)
```

Para crear una tabla con métricas adicionales para un análisis temporal.

### SELECTCOLUMNS()

Crea una nueva tabla seleccionando solo las columnas que necesitas. Es una forma efectiva de reducir la cantidad de datos en memoria y simplificar una tabla para un cálculo específico.

```dax
SELECTCOLUMNS(<table>, <nombre>, <expresión>...)
```

Cuando solo necesitas un subconjunto de columnas.

## Combinación de Tablas
(misma lógica de los [[Joins|Joins en SQL]])

Estas funciones te permiten combinar o comparar tablas, lo cual es útil para escenarios donde necesitas consolidar datos de múltiples fuentes o encontrar valores comunes.

### UNION()

Combina dos o más tablas en una sola. Las tablas deben tener el mismo número de columnas y los tipos de datos en las columnas correspondientes deben ser compatibles. Los nombres de las columnas se toman de la primera tabla.

```dax
UNION(<table>, <table>...)
```

Para consolidar datos de ventas de diferentes años o regiones.

### INTERSECT()

Devuelve una tabla con las filas que son comunes en ambas tablas.

```dax
INTERSECT(<tabla1>, <tabla2>)
```

Para identificar los clientes que compraron tanto el Producto A como el Producto B.

### NATURALINNERJOIN() y NATURALLEFTOUTERJOIN()

Unen dos tablas basándose en columnas con los mismos nombres. `NATURALINNERJOIN` solo mantiene las filas que tienen coincidencias en ambas tablas, mientras que `NATURALLEFTOUTERJOIN` mantiene todas las filas de la tabla izquierda, agregando `BLANK` si no hay coincidencias en la derecha.

```dax
NATURALINNERJOIN(<tabla1>, <tabla2>)
NATURALLEFTOUTERJOIN(<tabla1>, <tabla2>)
```

Útiles para uniones simples en tiempo real.

## Otras Funciones

### DISTINCT()

Devuelve una tabla con los valores únicos de una columna o las filas únicas de una tabla. Es útil para obtener una lista de valores sin duplicados.

```dax
DISTINCT(<columna> | <table>)
```

Para obtener una lista de clientes únicos, productos únicos, etc.
