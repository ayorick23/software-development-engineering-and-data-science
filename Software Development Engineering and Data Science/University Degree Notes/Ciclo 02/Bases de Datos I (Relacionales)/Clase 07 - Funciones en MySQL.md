---
Fecha de creación: 2025-08-29T18:19:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-08-29
---
# Funciones en MySQL

## Funciones de Caracteres
(ver [[Advanced Functions#1. String Functions (_Funciones de Texto_)|String Functions]])

- **``CHAR_LENGTH``:** retorna la longitud de una cadena de caracteres.

```sql
SELECT CHAR_LENGTH("SQL Tutorial") AS longitud;
```

- **``CONCAT``:** Une distintos caracteres en una sola cadena.

```sql
SELECT CONCAT("SQL ", "Tutorial ", "es ", "divertido!") AS una_sola_cadena;
```

- **``INSTR``:** retorna la posición de la primera aparición de un carácter en una cadena de caracteres.

```sql
SELECT INSTR("UEES", "S") AS posición;
```

- **``LCASE``:** convierte una cadena de caracteres a minúsculas.

```sql
SELECT LCASE("SQL Tutorial es Divertido!");
```

- **``LEFT``:** extrae de una cadena de caracteres un número determinado de caracteres comenzando por la izquierda.

```sql
SELECT LEFT("SQL Tutorial", 3) AS extracción;
```

- **``LTRIM``:** elimina los espacios en blanco desde la izquierda de una cadena de caracteres.

```sql
SELECT LTRIM("     SQL Tutorial") AS caracteres_recortados;
```

- **``REPLACE``:** reemplaza todas las ocurrencias de una sub-cadena de caracteres dentro de una cadena de caracteres.

```sql
SELECT REPLACE("SQL Tutorial", "SQL", "HTML");
```

- **``RIGHT``:** extrae de una cadena de caracteres un número determinado de caracteres comenzando por la derecha.

```sql
SELECT RIGHT("SQL Tutorial es súper", 5) AS cadena_extraida;
```

- ``**SUBSTR``:** extrae de una cadena de caracteres una sub-cadena de caracteres desde una posición específica con una longitud específica.

```sql
SELECT SUBSTR("SQL Tutorial", 5, 3) AS cadena_extraída;
```

- **``UPPER``:** convierte toda una cadena de caracteres a mayúsculas.

```sql
SELECT UPPER("SQL Tutorial es divertido!");
```

## Funciones de Números
(ver [[SQL/Aggregate Functions|Aggregate Functions]])

- **``ABS``:** retorna el valor absoluto de un número.

```sql
SELECT ABS(-243.5);
```

- **``AVG``:** retorna el valor promedio de un conjunto de valores de una columna.

```sql
SELECT AVG(Precio) AS PrecioPromedio FROM Productos;
```

- **``COUNT``:** retorna la cantidad de registros, si bien es cierto está dentro de las funciones numéricas se utiliza con columnas tanto numéricas como que no lo sean.

```sql
SELECT COUNT(ProductoID) AS NúmeroDeProductos FROM Productos;
```

- **``DIV``:** retorna el resultado entero de una división.

```sql
SELECT 10 DIV 5;
```

- **``MAX``:** retorna el valor máximo de una columna de un conjunto de valores.

```sql
SELECT MAX(Precio) AS MáximoPrecio FROM Productos;
```

- **``MIN``:** retorna el valor mínimo de una columna de un conjunto de valores.

```sql
SELECT MIN(Precio) AS MínimoPrecio FROM Productos;
```

- **``SUM``:** retorna la suma de una columna de un conjunto de valores.

```sql
SELECT SUM(Cantidad) AS TotalDeProductosOrdenados FROM OrdenDetalle;
```

- **``ROUND``:** redondea un valor numérico a la cantidad de decimales definidos.

```sql
SELECT ROUND(135.375, 2);
```

## Funciones de Fechas
(ver [[Advanced Functions#3. Date and Time Functions (_Funciones de Fecha y Hora_)|Date and Time Functions]])

- **``CURRENT_DATE``:** retorna la fecha actual.

```sql
SELECT CURRENT_DATE();
```

- **``ADDDATE``:** esta función añade un intervalo de tiempo a una columna de tipo fecha.

```sql
SELECT ADDDATE("2017-06-15", INTERVAL 10 DAY);
```

- **``DATEDIFF``:** retorna el número de días entre dos columnas de tipo fecha.

```sql
SELECT DATEDIFF("2017-06-25", "2017-06-15");
```

- **``DATE_FORMAT``:** esta función da formato a una columna de fecha de acuerdo con lo especificado.

```sql
SELECT DATE_FORMAT("2017-06-15", "%M %d %Y");
```

- **``DAYNAME``:** retorna el nombre del día de la semana de una columna de fecha.

```sql
SELECT DAYNAME("2017-06-15");
```

- **``MONTHNAME``:** retorna el nombre del mes de una columna de fecha.

```sql
SELECT MONTHNAME("2017-06-15");
```

- **``QUARTER``:** retorna el nombre del trimestre de una columna de fecha.

```sql
SELECT QUARTER("2017-06-15");
```

- **``WEEK``:** retorna el número de la semana de una columna de fecha.

```sql
SELECT WEEK("2017-06-15");
```

- **``YEAR``:** retorna el año de una columna de fecha.

```sql
SELECT YEAR("2017-06-15");
```

- **``TIMEDIFF``:** retorna la diferen
- 
- cia en tiempo entre dos columnas de fecha.

```sql
SELECT TIMEDIFF("13:10:11", "13:10:10");
```
