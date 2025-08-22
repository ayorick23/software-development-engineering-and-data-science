# Funciones de Fecha y Hora

En **DAX**, las **funciones de fecha y hora** te permiten manipular, extraer y analizar datos basados en el tiempo. Son esenciales para tareas de inteligencia de negocio, como el análisis de ventas por año, la comparación de meses o el cálculo de la antigüedad de un producto.

## Creación de Tablas de Fecha

Una de las mejores prácticas en DAX es usar una tabla de fechas dedicada. La función `CALENDAR` es tu herramienta principal para esto.

```dax
CALENDAR(<fecha_inicio>, <fecha_fin>)
```

Devuelve una tabla de una sola columna llamada `Date` que contiene una fecha por cada día entre `start_date` y `end_date`. A menudo se usa para crear una tabla de calendario completa para tu modelo de datos, la cual luego puedes enriquecer con columnas adicionales como Año, Mes, Trimestre, etc.

## Extracción de Componentes de Fecha
(misma lógica de [[Advanced Functions#3. Date and Time Functions (_Funciones de Fecha y Hora_)|Funciones de Fecha y Hora en SQL]])

Estas funciones te permiten descomponer una fecha en sus partes constitutivas para análisis más detallados.

### YEAR()

Devuelve el año de una fecha como un número entero.

```dax
YEAR(<fecha>)
```

### MONTH()

Devuelve el mes de una fecha como un número (del 1 al 12).

```dax
MONTH(<fecha>)
```

### DAY()

Devuelve el día del mes como un número (del 1 al 31).

```dax
DAY(<fecha>)
```

### QUARTER()

Devuelve el trimestre del año como un número (del 1 al 4).

```dax
QUARTER(<fecha>)
```

### WEEKNUM()

Devuelve el número de semana de una fecha. El argumento `tipo_retorno` especifica qué día se considera el inicio de la semana. Por ejemplo, 1 (el predeterminado) indica que la semana comienza el domingo.

```dax
WEEKNUM(<fecha>[, <tipo_retorno>])
```

## Conversión y Cálculo entre Fechas

Estas funciones son cruciales para manejar y medir períodos de tiempo.

### DATE()

Crea un valor de fecha a partir de sus componentes de año, mes y día.

```dax
DATE(<año>, <mes>, <día>)
```

### DATEDIFF()

Calcula la diferencia entre dos fechas en un intervalo de tiempo específico (segundos, minutos, horas, días, meses, años).

```dax
DATEDIFF(<fecha1>, <fecha2>, <intervalo>)
```

Ideal para calcular la antigüedad de un cliente, el tiempo de entrega de un pedido, o el tiempo transcurrido desde un evento.

### DATEVALUE()

Convierte una cadena de texto que representa una fecha en un valor de fecha.

```dax
DATEVALUE(<texto_de_fecha>)
```

Esencial para limpiar datos de fechas que se importaron como texto.
