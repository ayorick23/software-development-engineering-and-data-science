# Funciones de Inteligencia del Tiempo

Las funciones de inteligencia de tiempo en DAX permiten a los usuarios realizar análisis y comparaciones basadas en el tiempo, como cálculos de año tras año, mes a mes, etc. Estas funciones son cruciales para comprender tendencias, crecimiento y rendimiento a lo largo del tiempo en informes y análisis de datos. También se puede asociar a algunas [[Advanced Functions#3. Date and Time Functions (_Funciones de Fecha y Hora_)|funciones de fecha y hora de SQL]].

## Funciones Principales

- `DATEADD(<dates>, <number_of_intervals>, <interval>)`: Mueve una fecha por un intervalo específico.
- `DATESBETWEEN(<dates>, <date_1>, <date_2>)`: Devuelve las fechas entre fechas especificadas.
- `TOTALYTD(<expression>, <dates>[, <filter>][, <year_end_date>])`: Evalúa el valor del año hasta la fecha de la expresión en el contexto actual.
- `SAMEPERIODLASTYEAR(<dates>)`: Devuelve una tabla que contiene una columna de fechas desplazadas un año atrás en el tiempo.
- `STARTOFMONTH(<dates>) // ENDOFMONTH(<dates>)`: Devuelve el inicio // final del mes.
- `STARTOFQUARTER(<dates>) // ENDOFQUARTER(<dates>)`: Devuelve el inicio // final del trimestre.
- `STARTOFYEAR(<dates>) // ENDOFYEAR(<dates>)`: Devuelve el inicio // final del año.
