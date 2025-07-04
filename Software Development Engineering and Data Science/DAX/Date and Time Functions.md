# Funciones de Fecha y Hora

En DAX, las funciones de fecha y hora permiten manipular y calcular datos basados en fechas y horas. Estas funciones son esenciales para tareas como extraer partes de una fecha (año, mes, día), calcular diferencias entre fechas, y trabajar con datos de tiempo en modelos de datos, muy similar a las [[Advanced Functions#3. Date and Time Functions (_Funciones de Fecha y Hora_)|funciones de fecha y hora de SQL]].

## Funciones Principales

- `CALENDAR(<start_date>, <end_date>)`: Devuelve una tabla con una sola columna denominada "Fecha" que contiene un conjunto contiguo de fechas.
- `DATE(<year>, <month>, <day>)`: Devuelve la fecha especificada en formato de fecha y hora.
- `DATEDIFF(<date_1>, <date_2>, <interval>)`: Devuelve el número de unidades entre dos fechas según lo definido en `<interval>`.
- `DATEVALUE(<date_text>)`: Convierte una fecha en texto a una fecha en formato de fecha y hora.
- `DAY(<date>)`: Devuelve un número del 1 al 31 que representa el día del mes.
- `WEEKNUM(<date>, <return_type>)`: Devuelve el número de semana del año.
- `MONTH(<date>)`: Devuelve un número del 1 al 12 que representa un mes.
- `QUARTER(<date>)`: Devuelve un número del 1 al 4 que representa un cuarto.
