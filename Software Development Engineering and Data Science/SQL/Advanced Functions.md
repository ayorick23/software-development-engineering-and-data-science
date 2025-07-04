# Advanced Functions

Las funciones en SQL son herramientas poderosas que te permiten manipular y transformar los datos directamente dentro de tus consultas. Más allá de las [[Aggregate Functions|funciones de agregación]], SQL ofrece una rica variedad de funciones para trabajar con texto, números, fechas y lógica condicional. Estas funciones son cruciales para limpiar datos, formatear resultados y realizar análisis más complejos.

## 1. String Functions (_Funciones de Texto_)

Las funciones de texto te permiten manipular cadenas de caracteres, como concatenar, extraer partes, cambiar mayúsculas/minúsculas y reemplazar texto.

| Función                    | Descripción                                                                                                                   | Sintaxis                                                   |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| `CONCAT()`                 | Une dos o más cadenas de texto.                                                                                               | `CONCAT(cadena1, cadena2, ...)`                            |
| `LENGTH()` / `LEN()`       | Devuelve la longitud de una cadena de texto. `LEN()` en SQL Server, `LENGTH()` en MySQL/PostgreSQL.                           | `LENGTH(cadena)` o `LEN(cadena)`                           |
| `SUBSTRING()` / `SUBSTR()` | Extrae una subcadena de una cadena, a partir de una posición inicial y con una longitud opcional. `SUBSTR()` en MySQL/Oracle. | `SUBSTRING(cadena, inicio, longitud)`                      |
| `REPLACE()`                | Reemplaza todas las ocurrencias de una subcadena por otra dentro de una cadena.                                               | `REPLACE(cadena, subcadena_a_reemplazar, nueva_subcadena)` |
| `UPPER()` / `UCASE()`      | Convierte todos los caracteres de una cadena a mayúsculas. `UCASE()` en MySQL.                                                | `UPPER(cadena)`                                            |
| `LOWER()` / `LCASE()`      | Convierte todos los caracteres de una cadena a minúsculas. `LCASE()` en MySQL.                                                | `LOWER(cadena)`                                            |
| `TRIM()`                   | Elimina espacios (u otros caracteres) del inicio, final o ambos lados de una cadena.                                          | `TRIM([LEADING\|TRAILING\|BOTH] [caracteres FROM] cadena)` |
| `LEFT()` / `RIGHT()`       | Extrae un número específico de caracteres desde el inicio (`LEFT`) o el final (`RIGHT`) de una cadena.                        | `LEFT(cadena, numero_caracteres)`                          |

## 2. Numeric Functions (_Funciones Numéricas_)

Las funciones numéricas se utilizan para realizar operaciones matemáticas y manipulaciones con valores numéricos.

| Función                | Descripción                                                                                      | Sintaxis                                          |
| ---------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------- |
| `ABS()`                | Devuelve el valor absoluto de un número.                                                         | `ABS(numero)`                                     |
| `ROUND()`              | Redondea un número a un número específico de decimales.                                          | `ROUND(numero, decimales)`                        |
| `FLOOR()`              | Redondea un número hacia abajo al entero más cercano.                                            | `FLOOR(numero)`                                   |
| `CEILING()` / `CEIL()` | Redondea un número hacia arriba al entero más cercano. `CEIL()` en MySQL/PostgreSQL.             | `CEILING(numero)` o `CEIL(numero)`                |
| `MOD()` / `%`          | Devuelve el resto de una división. `%` en SQL Server/PostgreSQL, `MOD()` en MySQL/Oracle.        | `MOD(dividendo, divisor)` o `dividendo % divisor` |
| `POWER()`              | Devuelve el valor de un número elevado a la potencia de otro.                                    | `POWER(base, exponente)`                          |
| `SQRT()`               | Devuelve la raíz cuadrada de un número.                                                          | `SQRT(numero)`                                    |
| `RAND()` / `RANDOM()`  | Genera un número pseudoaleatorio. `RAND()` en MySQL/SQL Server, `RANDOM()` en PostgreSQL/SQLite. | `RAND()` o `RANDOM()`                             |

## 3. Date and Time Functions (_Funciones de Fecha y Hora_)

Las funciones de fecha y hora te permiten extraer partes de fechas, calcular diferencias, añadir intervalos y formatear valores de fecha/hora. La sintaxis puede variar significativamente entre DBMS.

| Función                                  | Descripción                                                                                             | Sintaxis (puede variar)                                                                                                                          |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `CURRENT_DATE()` / `GETDATE()` / `NOW()` | Devuelve la fecha actual. `GETDATE()` en SQL Server, `CURRENT_DATE()` en PostgreSQL, `NOW()` en MySQL.  | `CURRENT_DATE()` o `GETDATE()` o `NOW()`                                                                                                         |
| `CURRENT_TIME()` / `GETUTCDATE()`        | Devuelve la hora actual o la hora UTC.                                                                  | `CURRENT_TIME()` o `GETUTCDATE()`                                                                                                                |
| `CURRENT_TIMESTAMP()` / `NOW()`          | Devuelve la fecha y hora actuales. `NOW()` en MySQL/PostgreSQL.                                         | `CURRENT_TIMESTAMP()` o `NOW()`                                                                                                                  |
| `DATE()` / `TIME()`                      | Extrae solo la parte de fecha o solo la parte de hora de un valor `DATETIME`/`TIMESTAMP`.               | `DATE(timestamp_expr)` o `TIME(timestamp_expr)`                                                                                                  |
| `YEAR()` / `MONTH()` / `DAY()`           | Extrae el año, mes o día de una fecha.                                                                  | `YEAR(fecha)` / `MONTH(fecha)` / `DAY(fecha)`                                                                                                    |
| `DATE_PART()` / `EXTRACT()`              | Extrae una parte específica de una fecha (ej., año, mes, día, hora, minuto). `EXTRACT()` en PostgreSQL. | `DATE_PART('part', fecha)` (PostgreSQL/Oracle) o `EXTRACT(part FROM fecha)` (SQL Standard)                                                       |
| `DATE_ADD()` / `DATE_SUB()` / `INTERVAL` | Añade o sustrae un intervalo de tiempo de una fecha. Sintaxis varía mucho.                              | `DATE_ADD(fecha, INTERVAL valor unidad)` (MySQL) o `fecha + INTERVAL 'valor unidad'` (PostgreSQL) o `DATEADD(unidad, valor, fecha)` (SQL Server) |
| `DATEDIFF()`                             | Calcula la diferencia entre dos fechas en una unidad específica (días, meses, años).                    | `DATEDIFF(unidad, fecha_inicio, fecha_fin)` (SQL Server) o `DATEDIFF(fecha_fin, fecha_inicio)` (MySQL)                                           |
| `DATE_FORMAT()` / `TO_CHAR()`            | Formatea una fecha/hora en una cadena con un formato específico. Sintaxis varía mucho.                  | `DATE_FORMAT(fecha, '%Y-%m-%d')` (MySQL) o `TO_CHAR(fecha, 'YYYY-MM-DD')` (PostgreSQL/Oracle)                                                    |

## 4. Conditional Functions (_Funciones Condicionales_)

Las funciones condicionales permiten ejecutar lógica `IF-THEN-ELSE` directamente en tus consultas, lo que es útil para transformar datos basados en ciertas condiciones.

| Función          | Descripción                                                                                                                                                                                                                                                                                           | Sintaxis                                                                                                            |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `CASE`           | La función condicional más versátil. Evalúa múltiples condiciones y devuelve un valor basado en la primera condición que sea verdadera. Puede ser simple (`CASE columna WHEN valor THEN resultado ELSE otro_resultado END`) o buscada (`CASE WHEN condicion THEN resultado ELSE otro_resultado END`). | `CASE WHEN condicion1 THEN resultado1 [WHEN condicion2 THEN resultado2 ...] [ELSE resultado_default] END`           |
| `NULLIF()`       | Devuelve `NULL` si las dos expresiones son iguales; de lo contrario, devuelve la primera expresión.                                                                                                                                                                                                   | `NULLIF(expresion1, expresion2)`                                                                                    |
| `COALESCE()`     | Devuelve la primera expresión no nula de una lista de expresiones.                                                                                                                                                                                                                                    | `COALESCE(expresion1, expresion2, ...)`                                                                             |
| `IF()` / `IIF()` | Funciones condicionales más simples para escenarios `IF-THEN-ELSE`. `IF()` en MySQL, `IIF()` en SQL Server.                                                                                                                                                                                           | `IF(condicion, valor_verdadero, valor_falso)` (MySQL) o `IIF(condicion, valor_verdadero, valor_falso)` (SQL Server) |
